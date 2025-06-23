# Observabilidade e Resiliência - Sistema GNRE

## 🎯 Objetivo

Implementar observabilidade completa e estratégias de resiliência para garantir alta disponibilidade, performance e facilidade de troubleshooting do sistema GNRE.

## 📊 Observabilidade Unificada

### 1. Distributed Tracing com OpenTelemetry

#### 1.1. Configuração Base

**Frontend (React/Next.js):**
```javascript
// tracing.js
import { WebTracerProvider } from '@opentelemetry/sdk-trace-web';
import { getWebAutoInstrumentations } from '@opentelemetry/auto-instrumentations-web';
import { registerInstrumentations } from '@opentelemetry/instrumentation';

const provider = new WebTracerProvider({
  resource: new Resource({
    'service.name': 'gnre-frontend',
    'service.version': '1.0.0',
  }),
});

registerInstrumentations({
  instrumentations: [
    getWebAutoInstrumentations({
      '@opentelemetry/instrumentation-document-load': {
        enabled: true,
      },
      '@opentelemetry/instrumentation-user-interaction': {
        enabled: true,
      },
      '@opentelemetry/instrumentation-fetch': {
        enabled: true,
      },
    }),
  ],
});
```

**Backend (FastAPI/Python):**
```python
# tracing.py
from opentelemetry import trace
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.instrumentation.sqlalchemy import SQLAlchemyInstrumentor
from opentelemetry.instrumentation.redis import RedisInstrumentor

def setup_tracing():
    trace.set_tracer_provider(TracerProvider(
        resource=Resource.create({
            "service.name": "gnre-backend",
            "service.version": "1.0.0"
        })
    ))
    
    jaeger_exporter = JaegerExporter(
        agent_host_name="jaeger",
        agent_port=6831,
    )
    
    span_processor = BatchSpanProcessor(jaeger_exporter)
    trace.get_tracer_provider().add_span_processor(span_processor)
    
    # Auto-instrumentação
    FastAPIInstrumentor.instrument_app(app)
    SQLAlchemyInstrumentor().instrument()
    RedisInstrumentor().instrument()
```

**Aplicação Local (Go):**
```go
// tracing.go
package main

import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/exporters/jaeger"
    "go.opentelemetry.io/otel/sdk/trace"
)

func initTracing() {
    exp, err := jaeger.New(jaeger.WithCollectorEndpoint(
        jaeger.WithEndpoint("http://jaeger:14268/api/traces"),
    ))
    if err != nil {
        log.Fatal(err)
    }
    
    tp := trace.NewTracerProvider(
        trace.WithBatcher(exp),
        trace.WithResource(resource.NewWithAttributes(
            semconv.SchemaURL,
            semconv.ServiceNameKey.String("gnre-local-agent"),
            semconv.ServiceVersionKey.String("1.0.0"),
        )),
    )
    
    otel.SetTracerProvider(tp)
}
```

### 2. Métricas de Negócio e Técnicas

#### 2.1. Métricas Críticas

**Métricas de Negócio:**
```python
# metrics.py
from prometheus_client import Counter, Histogram, Gauge

# Métricas de negócio
gnres_generated_total = Counter(
    'gnres_generated_total',
    'Total number of GNREs generated',
    ['company_id', 'uf', 'status']
)

gnre_processing_duration = Histogram(
    'gnre_processing_duration_seconds',
    'Time spent processing GNRE',
    ['processing_stage']
)

active_subscriptions = Gauge(
    'active_subscriptions_total',
    'Number of active subscriptions',
    ['plan_type']
)

# Métricas técnicas
api_requests_total = Counter(
    'api_requests_total',
    'Total API requests',
    ['method', 'endpoint', 'status_code']
)

database_connections = Gauge(
    'database_connections_active',
    'Active database connections'
)
```

#### 2.2. Dashboard Grafana

```json
{
  "dashboard": {
    "title": "GNRE System Overview",
    "panels": [
      {
        "title": "GNREs Generated per Hour",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(gnres_generated_total[1h])",
            "legendFormat": "{{uf}} - {{status}}"
          }
        ]
      },
      {
        "title": "API Response Time P95",
        "type": "singlestat",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(api_request_duration_seconds_bucket[5m]))"
          }
        ]
      },
      {
        "title": "System Health",
        "type": "table",
        "targets": [
          {
            "expr": "up{job=\"gnre-backend\"}"
          }
        ]
      }
    ]
  }
}
```

## 🛡️ Estratégias de Resiliência

### 1. Circuit Breaker Pattern

#### 1.1. Implementação para SEFAZ

```python
# circuit_breaker.py
from pybreaker import CircuitBreaker
import logging

logger = logging.getLogger(__name__)

# Configuração do Circuit Breaker para SEFAZ
sefaz_breaker = CircuitBreaker(
    fail_max=5,           # Falha após 5 tentativas
    reset_timeout=60,     # Tenta reabrir após 60s
    exclude=[ValueError], # Não conta erros de validação
    name="SEFAZ_WebService"
)

@sefaz_breaker
def call_sefaz_webservice(xml_data, certificate):
    """
    Chama webservice da SEFAZ com circuit breaker
    """
    try:
        response = sefaz_client.send_gnre(xml_data, certificate)
        logger.info(f"SEFAZ call successful: {response.protocol}")
        return response
    except ConnectionError as e:
        logger.error(f"SEFAZ connection failed: {e}")
        raise
    except TimeoutError as e:
        logger.error(f"SEFAZ timeout: {e}")
        raise

# Fallback quando circuit breaker está aberto
def sefaz_fallback(xml_data):
    """
    Estratégia de fallback quando SEFAZ está indisponível
    """
    # Salvar na fila para retry posterior
    redis_client.lpush("sefaz_retry_queue", json.dumps({
        "xml_data": xml_data,
        "timestamp": datetime.utcnow().isoformat(),
        "retry_count": 0
    }))
    
    return {
        "status": "queued_for_retry",
        "message": "SEFAZ temporariamente indisponível. Processamento agendado."
    }
```

### 2. Retry com Backoff Exponencial

#### 2.1. Aplicação Local

```go
// retry.go
package main

import (
    "time"
    "math"
    "context"
)

type RetryConfig struct {
    MaxRetries  int
    BaseDelay   time.Duration
    MaxDelay    time.Duration
    Multiplier  float64
}

func RetryWithBackoff(ctx context.Context, config RetryConfig, fn func() error) error {
    var lastErr error
    
    for attempt := 0; attempt <= config.MaxRetries; attempt++ {
        if attempt > 0 {
            delay := time.Duration(float64(config.BaseDelay) * math.Pow(config.Multiplier, float64(attempt-1)))
            if delay > config.MaxDelay {
                delay = config.MaxDelay
            }
            
            select {
            case <-time.After(delay):
            case <-ctx.Done():
                return ctx.Err()
            }
        }
        
        if err := fn(); err != nil {
            lastErr = err
            log.Printf("Attempt %d failed: %v", attempt+1, err)
            continue
        }
        
        return nil
    }
    
    return fmt.Errorf("all %d attempts failed, last error: %w", config.MaxRetries+1, lastErr)
}

// Uso
func sendToAPI(data []byte) error {
    config := RetryConfig{
        MaxRetries: 5,
        BaseDelay:  1 * time.Second,
        MaxDelay:   30 * time.Second,
        Multiplier: 2.0,
    }
    
    return RetryWithBackoff(context.Background(), config, func() error {
        return httpClient.Post("/api/v1/gnre/from-local-agent", data)
    })
}
```

### 3. Health Checks e Readiness Probes

#### 3.1. Backend Health Checks

```python
# health.py
from fastapi import APIRouter, HTTPException
from sqlalchemy import text
import redis

router = APIRouter()

@router.get("/health")
async def health_check():
    """Health check básico"""
    return {"status": "healthy", "timestamp": datetime.utcnow()}

@router.get("/health/ready")
async def readiness_check():
    """Verifica se todos os serviços dependentes estão disponíveis"""
    checks = {}
    overall_status = "healthy"
    
    # Check database
    try:
        db.execute(text("SELECT 1"))
        checks["database"] = "healthy"
    except Exception as e:
        checks["database"] = f"unhealthy: {str(e)}"
        overall_status = "unhealthy"
    
    # Check Redis
    try:
        redis_client.ping()
        checks["redis"] = "healthy"
    except Exception as e:
        checks["redis"] = f"unhealthy: {str(e)}"
        overall_status = "unhealthy"
    
    # Check MinIO
    try:
        minio_client.bucket_exists("gnre-files")
        checks["minio"] = "healthy"
    except Exception as e:
        checks["minio"] = f"unhealthy: {str(e)}"
        overall_status = "unhealthy"
    
    if overall_status == "unhealthy":
        raise HTTPException(status_code=503, detail=checks)
    
    return {"status": overall_status, "checks": checks}
```

#### 3.2. Docker Health Checks

```dockerfile
# Dockerfile
FROM python:3.11-slim

# ... outras configurações ...

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1
```

```yaml
# docker-compose.yml
services:
  backend:
    build: .
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health/ready"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

## 📋 Implementação por Agente

### Agente 5 - Backend Developer
- [ ] Implementar OpenTelemetry tracing
- [ ] Configurar métricas Prometheus
- [ ] Implementar Circuit Breaker para SEFAZ
- [ ] Criar health checks completos
- [ ] Configurar retry policies

### Agente 4 - Frontend Developer  
- [ ] Instrumentar frontend com OpenTelemetry
- [ ] Implementar error boundaries
- [ ] Configurar client-side monitoring
- [ ] Implementar offline fallbacks

### Agente 6 - DevOps Engineer
- [ ] Configurar Jaeger para tracing
- [ ] Setup Prometheus + Grafana
- [ ] Implementar alerting rules
- [ ] Configurar log aggregation

### Agente 10 - Observability Specialist
- [ ] Criar dashboards Grafana
- [ ] Definir SLIs/SLOs
- [ ] Configurar alertas inteligentes
- [ ] Implementar runbooks automatizados

---

*Este documento complementa o planejamento principal com foco em observabilidade e resiliência.*

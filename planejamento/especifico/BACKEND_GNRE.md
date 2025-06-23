# Planejamento Detalhado do Backend da Aplicação de Geração de GNRE

## 3. Backend (API)

O Backend será o núcleo do sistema, projetado para ser `stateless`, seguro, eficiente e escalável.

### 3.1. Stack Tecnológica Consolidada

*   **Framework:** `FastAPI (Python 3.11)` é a escolha recomendada, conforme `BACK.md` e `OpenAI`. Embora `Claude` e `GEMINI` sugiram `Node.js/NestJS`, o `FastAPI` oferece alta performance, tipagem forte com `Pydantic` e uma excelente experiência de desenvolvimento em Python, que é uma linguagem robusta para processamento de dados e integração com sistemas legados (SEFAZ).
*   **Servidor ASGI:** `Uvicorn` para desenvolvimento e `Gunicorn` para produção.
*   **ORM:** `Prisma` (se Node.js fosse escolhido) ou ORM nativo do Python (como `SQLAlchemy` com `Alembic` para migrações) para interação com o PostgreSQL. A documentação `BACK.md` não especifica um ORM, mas o `Prisma` é uma boa prática para Node.js. Para Python, `SQLAlchemy` é a escolha padrão.
*   **Autenticação:** `Python-Jose` para `JWT` e `Passlib/Argon2/Bcrypt` para hashing seguro de senhas. `Passport.js` seria para Node.js.
*   **Validação de Dados:** `Pydantic` para validação de DTOs e `Joi` (se Node.js) para schemas rigorosos.
*   **Filas de Processamento:** `Bull/BullMQ` (se Node.js) ou uma solução baseada em `Redis` (`Celery` para Python) para processamento assíncrono de XMLs e geração de GNREs.
*   **Logging:** `Winston` (se Node.js) ou `Serilog` (se .NET) ou sistema de logging nativo do Python com `ELK Stack` ou `Logtail` para logging centralizado e imutável.
*   **Integração SEFAZ:** Biblioteca confiável para comunicação `SOAP` com webservices da SEFAZ.

### 3.2. Endpoints da API (RESTful)

A API será versionada (ex: `/api/v1/...`).

*   **Autenticação:**
    *   `POST /auth/login`: Autentica usuário, retorna JWT.
    *   `POST /auth/register`: Cadastra nova empresa e usuário.
    *   `POST /auth/refresh-token`: Obtém novo token de acesso.
    *   `POST /auth/forgot-password`, `POST /auth/reset-password`.
*   **Empresas:**
    *   `GET /companies/me`: Retorna dados da empresa logada.
    *   `PUT /companies/me`: Atualiza dados da empresa.
    *   `POST /companies/me/certificate`: Upload seguro de certificado digital.
    *   `POST /companies/me/api-token`: Gera novo token de API para aplicação local.
*   **GNRE:**
    *   `POST /gnre/process-xml`: Recebe XMLs do frontend, cria jobs em fila.
    *   `POST /gnre/from-local-agent`: Recebe dados de XMLs processados do agente local.
    *   `POST /gnre/send-batch`: Envia lote de GNREs para SEFAZ.
    *   `GET /gnre/config/:uf`: Consulta configurações específicas por UF.
    *   `GET /gnre/batch/:batchId`: Consulta status de lote.
    *   `GET /gnre/:id`: Consulta detalhes de GNRE específica.
    *   `GET /gnre`: Lista GNREs com filtros e paginação.
    *   `GET /gnre/:id/pdf`: Gera/retorna PDF da GNRE (via URL pré-assinada do MinIO).
*   **Upload/Storage:**
    *   `POST /upload/xml`: Upload de XMLs.
    *   `POST /upload/certificate`: Upload de certificados.
    *   `GET /files/:id`: Download de arquivos.

### 3.3. Foco em Segurança (Backend)

*   **Autenticação e Autorização Robusta:**
    *   `JWT` com algoritmo `RS256` (assimétrico) para assinatura.
    *   `Claims` do token com `sub`, `company_id`, `exp`.
    *   `Refresh tokens` com armazenamento de `hash SHA-256` no banco.
    *   `RBAC` (Role-Based Access Control) e `ABAC` (Attribute-Based Access Control) para permissões granulares.
*   **Validação de Entrada e Sanitização:**
    *   Uso de `DTOs` com `Pydantic` para validação rigorosa de tipo, tamanho, formato.
    *   Sanitização de strings para prevenir `SQL Injection`, `NoSQL Injection`, `XSS`.
    *   Uso de `queries parametrizadas` com o ORM.
*   **Rate Limiting e Proteção contra Brute Force:**
    *   `nestjs-throttler` (se Node.js) ou implementação similar com `Redis` para limitar requisições por IP/usuário em endpoints sensíveis.
*   **Segurança de Cabeçalhos HTTP:**
    *   Uso de `Helmet.js` (se Node.js) ou configuração manual de cabeçalhos como `Strict-Transport-Security` (HSTS), `X-Content-Type-Options`, `X-Frame-Options`.
*   **Gerenciamento de Segredos:**
    *   Utilização de serviços como `HashiCorp Vault`, `AWS Secrets Manager` ou `Doppler` para gerenciar chaves de API, senhas de banco, etc., evitando hardcoding.
*   **Manuseio Ultra-Seguro do Certificado Digital:**
    *   Upload via HTTPS, senha nunca logada.
    *   Criptografia do certificado e senha com `DEK` (Data Encryption Key) gerada por `KMS` (Key Management Service) como `AWS KMS` ou `HashiCorp Vault`.
    *   Armazenamento do certificado criptografado no MinIO e da `DEK` criptografada e senha criptografada no PostgreSQL.
*   **Auditoria e Logging Completo:**
    *   Registro de eventos sensíveis (login, upload, download, alterações) com timestamp, IP, ID do usuário.
    *   Alertas para atividades suspeitas (múltiplos logins falhos, acessos incomuns).
    *   Logs separados para segurança e auditoria.

## 5. Agente 5 – Desenvolvedor(a) Back-end

**Objetivo:** Criar as APIs (FastAPI 0.115.12), configurar PostgreSQL (Supabase), Redis, filas de mensageria, validações com Pydantic e integrar serviços externos (SEFAZ, Gateways de Pagamento).

**Task List / To-Do:**

*   **5.1. Analisar Documentos (Discovery, Arquitetura, UX/UI, Front-end Requirements):**
    *   [ ] Revisar o `Planejamento_Detalhado_GNRE.md` para entender endpoints, parâmetros, payloads e constraints.
    *   [ ] Compreender as regras de negócio para processamento de XML e geração de GNRE.

*   **5.2. Configuração do Ambiente Back-end:**
    *   [ ] Inicializar projeto FastAPI com Python 3.11.
    *   [ ] Configurar `Uvicorn` (0.30.1) para desenvolvimento e `Gunicorn` (22.0.0) para produção.
    *   [ ] Configurar conexão com `Supabase/PostgreSQL` (16.x.x).
    *   [ ] Configurar `Redis` (7.2.x) para cache e filas.
    *   [ ] Implementar migrações de banco de dados (ex: com `Alembic` (1.13.2) para SQLAlchemy).
    *   [ ] Configurar variáveis de ambiente (`.env`) e gerenciamento de segredos (Vault/KMS).

*   **5.3. Implementação de Funcionalidades (APIs):**
    *   [ ] **Módulo de Autenticação:**
        *   [ ] Implementar `POST /auth/login`, `POST /auth/register`, `POST /auth/refresh-token`.
        *   [ ] Implementar `POST /auth/forgot-password`, `POST /auth/reset-password`.
        *   [ ] Gerar e validar `JWT` com `Python-Jose` (3.3.0) e `RS256`.
        *   [ ] Hashing de senhas com `Argon2` (23.1.0)/`Bcrypt` (4.1.3).
        *   [ ] Implementar `Rate Limiting` para endpoints de autenticação.
    *   [ ] **Módulo de Empresas:**
        *   [ ] Implementar `GET /companies/me`, `PUT /companies/me`.
        *   [ ] Implementar `POST /companies/me/certificate` (upload seguro, criptografia).
        *   [ ] Implementar `POST /companies/me/api-token` (geração de token com hash).
    *   [ ] **Módulo de GNRE:**
        *   [ ] Implementar `POST /gnre/process-xml` (recebe XML, envia para fila).
        *   [ ] Implementar `POST /gnre/from-local-agent` (recebe dados processados do agente local).
        *   [ ] Implementar `POST /gnre/send-batch`.
        *   [ ] Implementar `GET /gnre/config/:uf`, `GET /gnre/batch/:batchId`, `GET /gnre/:id`.
        *   [ ] Implementar `GET /gnre` (listagem com filtros e paginação).
        *   [ ] Implementar `GET /gnre/:id/pdf` (busca/gera PDF, retorna URL pré-assinada).
    *   [ ] **Módulo de Upload/Storage:**
        *   [ ] Implementar `POST /upload/xml`, `POST /upload/certificate`.
        *   [ ] Implementar `GET /files/:id`.
        *   [ ] Integrar com `MinIO Client` (7.x.x) para armazenamento de objetos.

*   **5.4. Lógica de Negócio e Integrações:**
    *   [ ] **Processamento de XML:**
        *   [ ] Desenvolver serviço para parse e validação de XML (prevenção de XXE).
        *   [ ] Extrair dados fiscais e aplicar regras de negócio para cálculo de GNRE.
    *   [ ] **Integração SEFAZ:**
        *   [ ] Desenvolver wrapper para comunicação `SOAP` com webservices da SEFAZ.
        *   [ ] Gerenciar ambientes de homologação e produção.
    *   [ ] **Filas de Processamento (Celery (5.4.0)/Redis):**
        *   [ ] Configurar workers para processar filas de XML, GNRE, PDF e e-mail.
    *   [ ] **Webhooks:** Implementar webhooks para integrações com terceiros (ex: gateway de pagamento).
    *   [ ] **Persistência de Logs:** Implementar `audit trail` para ações críticas.

*   **5.5. Testes e Documentação (Básico nesta etapa):**
    *   [ ] Escrever testes unitários para serviços e controladores.
    *   [ ] Escrever testes de integração para fluxos críticos.
    *   [ ] Gerar documentação da API com `Swagger/OpenAPI` (integrado ao FastAPI).

*   **5.6. Foco em Segurança (Backend):**
    *   [ ] Implementar `RBAC/ABAC` para controle de acesso granular.
    *   [ ] Utilizar `Pydantic` para validação rigorosa de DTOs.
    *   [ ] Sanitizar todas as entradas para prevenir injeções.
    *   [ ] Usar `queries parametrizadas` com o ORM.
    *   [ ] Implementar `Rate Limiting` por IP/usuário.
    *   [ ] Configurar `Cabeçalhos HTTP de Segurança` (`Strict-Transport-Security`, `X-Content-Type-Options`, `X-Frame-Options`).
    *   [ ] Manuseio ultra-seguro de certificados digitais (criptografia com KMS).
    *   [ ] Auditoria e logging completo de eventos de segurança.
    *   [ ] Detecção de atividades suspeitas (tentativas de login, acessos incomuns).

**Artefato de Saída:** Código fonte do Backend.

## 7. Velocidade e Escalabilidade (Backend)

*   `FastAPI` para alta performance e baixo consumo de recursos.
*   Arquitetura `stateless` para fácil escalabilidade horizontal.
*   Filas de processamento (`Redis/Celery`) para operações assíncronas e desacoplamento.
*   `Redis` para cache de dados frequentemente acessados.
*   Otimização de queries de banco de dados com índices.

## 9. Justificativas para Divergências (Backend)

*   **Backend (Python/FastAPI vs Node.js/NestJS/Express):**
    *   **Sua escolha (`BACK.md`):** Python 3.11 com FastAPI.
    *   **Outras sugestões:** Node.js com Express.js/Fastify (`Claude`), NestJS (`GEMINI`).
    *   **Justificativa:** A escolha do Python com FastAPI é **validada e recomendada**. FastAPI é um framework moderno, de alta performance, com tipagem forte (Pydantic) e excelente para construir APIs. Para um sistema que envolve processamento de dados fiscais e integração com sistemas legados (SEFAZ, que muitas vezes usam SOAP), Python é uma linguagem muito capaz e com um ecossistema maduro para essas tarefas. A performance do FastAPI é comparável ou superior a muitos frameworks Node.js em cenários de I/O bound. Manter uma stack mais homogênea (Python para backend e Go para aplicação local) pode simplificar a curva de aprendizado da equipe em comparação com Node.js/TypeScript para backend e Go para aplicação local.
## 🔧 Tratamento de Erros Padronizado (RFC 7807)

### 6.1. Problem Details Implementation
```python
# src/core/exceptions.py
from typing import Optional, Dict, Any
from fastapi import HTTPException
from pydantic import BaseModel

class ProblemDetail(BaseModel):
    type: str
    title: str
    status: int
    detail: Optional[str] = None
    instance: Optional[str] = None
    extensions: Optional[Dict[str, Any]] = None

class BusinessRuleException(HTTPException):
    def __init__(
        self,
        status_code: int,
        title: str,
        detail: Optional[str] = None,
        type_uri: Optional[str] = None,
        instance: Optional[str] = None,
        **extensions
    ):
        self.problem_detail = ProblemDetail(
            type=type_uri or f"https://api.gnre.com/problems/{title.lower().replace(' ', '-')}",
            title=title,
            status=status_code,
            detail=detail,
            instance=instance,
            extensions=extensions if extensions else None
        )
        super().__init__(status_code=status_code, detail=self.problem_detail.dict())

# Exceções específicas do domínio
class InvalidXMLException(BusinessRuleException):
    def __init__(self, detail: str, xml_errors: list = None):
        super().__init__(
            status_code=422,
            title="Invalid XML Format",
            detail=detail,
            xml_errors=xml_errors
        )

class CertificateExpiredException(BusinessRuleException):
    def __init__(self, expiry_date: str):
        super().__init__(
            status_code=400,
            title="Certificate Expired",
            detail=f"Digital certificate expired on {expiry_date}",
            expiry_date=expiry_date
        )

# src/core/error_handlers.py
from fastapi import Request, HTTPException
from fastapi.responses import JSONResponse
import logging

logger = logging.getLogger(__name__)

async def business_rule_exception_handler(request: Request, exc: BusinessRuleException):
    logger.warning(
        "Business rule violation",
        extra={
            "path": request.url.path,
            "method": request.method,
            "problem_type": exc.problem_detail.type,
            "status_code": exc.problem_detail.status
        }
    )

    return JSONResponse(
        status_code=exc.problem_detail.status,
        content=exc.problem_detail.dict(exclude_none=True),
        headers={"Content-Type": "application/problem+json"}
    )
```

### 6.2. Idempotência de Endpoints
```python
# src/middleware/idempotency.py
from fastapi import Request, HTTPException
from functools import wraps
import hashlib
import json
from typing import Optional
import redis

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def idempotent(ttl_seconds: int = 3600):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            request: Request = kwargs.get('request') or args[0]

            # Gerar chave de idempotência
            idempotency_key = request.headers.get('Idempotency-Key')
            if not idempotency_key:
                # Auto-gerar baseado no conteúdo
                body = await request.body()
                content_hash = hashlib.sha256(
                    f"{request.method}:{request.url.path}:{body.decode()}".encode()
                ).hexdigest()
                idempotency_key = f"auto:{content_hash}"

            cache_key = f"idempotency:{idempotency_key}"

            # Verificar se já foi processado
            cached_response = redis_client.get(cache_key)
            if cached_response:
                return json.loads(cached_response)

            # Processar requisição
            result = await func(*args, **kwargs)

            # Cachear resultado
            redis_client.setex(
                cache_key,
                ttl_seconds,
                json.dumps(result, default=str)
            )

            return result
        return wrapper
    return decorator

# Uso nos endpoints
@app.post("/gnre/from-local-agent")
@idempotent(ttl_seconds=1800)  # 30 minutos
async def process_xml_from_agent(
    xml_data: XMLProcessRequest,
    request: Request,
    current_user: User = Depends(get_current_user)
):
    return await gnre_service.process_xml_data(xml_data, current_user.company_id)
```

## 🏗️ Arquitetura de Domínio e Regras de Negócio

### 7.1. Domain Services
```python
# src/domain/gnre_calculator.py
from decimal import Decimal
from typing import Dict, Any
from dataclasses import dataclass
from enum import Enum

class UFTaxRules(Enum):
    SP = "sao_paulo"
    RJ = "rio_janeiro"
    MG = "minas_gerais"
    # ... outros estados

@dataclass
class GNRECalculationInput:
    uf_origem: str
    uf_destino: str
    valor_mercadoria: Decimal
    cfop: str
    ncm: str
    peso: Optional[Decimal] = None

@dataclass
class GNRECalculationResult:
    valor_gnre: Decimal
    aliquota: Decimal
    base_calculo: Decimal
    codigo_receita: str
    detalhes: Dict[str, Any]

class GNRECalculatorService:
    def __init__(self, tax_rules_repository: TaxRulesRepository):
        self.tax_rules = tax_rules_repository

    async def calculate_gnre(self, input_data: GNRECalculationInput) -> GNRECalculationResult:
        # Buscar regras específicas do estado
        uf_rules = await self.tax_rules.get_rules_for_uf(input_data.uf_destino)

        # Aplicar regras de negócio
        if input_data.cfop.startswith('6'):  # Venda interestadual
            aliquota = await self._get_interstate_rate(input_data.uf_origem, input_data.uf_destino)
        else:
            aliquota = uf_rules.aliquota_interna

        # Calcular base de cálculo
        base_calculo = self._calculate_base(input_data, uf_rules)

        # Calcular valor final
        valor_gnre = base_calculo * aliquota

        return GNRECalculationResult(
            valor_gnre=valor_gnre,
            aliquota=aliquota,
            base_calculo=base_calculo,
            codigo_receita=uf_rules.codigo_receita,
            detalhes={
                "cfop": input_data.cfop,
                "ncm": input_data.ncm,
                "regra_aplicada": uf_rules.nome
            }
        )

    def _calculate_base(self, input_data: GNRECalculationInput, rules: UFTaxRules) -> Decimal:
        # Lógica complexa de cálculo da base
        base = input_data.valor_mercadoria

        # Aplicar reduções/acréscimos conforme regras do estado
        if rules.reducao_base:
            base = base * (1 - rules.reducao_base)

        return base
```

### 7.2. Gerenciamento de Transações
```python
# src/core/database.py
from contextlib import asynccontextmanager
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.exc import SQLAlchemyError
import logging

logger = logging.getLogger(__name__)

@asynccontextmanager
async def transaction_scope(session: AsyncSession):
    """Context manager para transações com rollback automático"""
    try:
        yield session
        await session.commit()
        logger.info("Transaction committed successfully")
    except SQLAlchemyError as e:
        await session.rollback()
        logger.error(f"Transaction rolled back due to error: {e}")
        raise
    except Exception as e:
        await session.rollback()
        logger.error(f"Transaction rolled back due to unexpected error: {e}")
        raise

# src/services/gnre_service.py
class GNREService:
    def __init__(self, db: AsyncSession, minio_client, sefaz_client):
        self.db = db
        self.minio = minio_client
        self.sefaz = sefaz_client

    async def process_complete_gnre_workflow(
        self,
        xml_data: bytes,
        company_id: str
    ) -> GNREProcessResult:
        """Processa GNRE com transação distribuída"""

        async with transaction_scope(self.db) as session:
            # 1. Salvar XML no MinIO
            xml_path = await self._upload_xml_to_storage(xml_data, company_id)

            try:
                # 2. Processar XML e extrair dados
                gnre_data = await self._extract_gnre_data(xml_data)

                # 3. Calcular valores
                calculation = await self._calculate_gnre_values(gnre_data)

                # 4. Salvar no banco
                gnre_record = await self._save_gnre_record(
                    session, gnre_data, calculation, xml_path, company_id
                )

                # 5. Enviar para SEFAZ (operação externa)
                try:
                    sefaz_response = await self._send_to_sefaz(gnre_record)
                    gnre_record.protocol_number = sefaz_response.protocol
                    gnre_record.status = GNREStatus.GERADO
                except SefazException as e:
                    # SEFAZ falhou, mas mantemos o registro para retry
                    gnre_record.status = GNREStatus.ERRO
                    gnre_record.error_message = str(e)

                    # Agendar retry
                    await self._schedule_sefaz_retry(gnre_record.id)

                await session.flush()
                return GNREProcessResult(
                    gnre_id=gnre_record.id,
                    status=gnre_record.status,
                    protocol=gnre_record.protocol_number
                )

            except Exception as e:
                # Limpar XML do storage em caso de erro
                await self._cleanup_xml_storage(xml_path)
                raise
```

## 📊 Monitoramento de Filas com Flower

### 8.1. Configuração do Flower
```python
# src/worker/celery_app.py
from celery import Celery
from celery.signals import task_prerun, task_postrun, task_failure
import logging
import time

app = Celery('gnre_worker')
app.config_from_object('src.core.celery_config')

logger = logging.getLogger(__name__)

@task_prerun.connect
def task_prerun_handler(sender=None, task_id=None, task=None, args=None, kwargs=None, **kwds):
    logger.info(
        "Task started",
        extra={
            "task_id": task_id,
            "task_name": task.name,
            "args": args,
            "kwargs": kwargs,
            "timestamp": time.time()
        }
    )

@task_postrun.connect
def task_postrun_handler(sender=None, task_id=None, task=None, args=None, kwargs=None, retval=None, state=None, **kwds):
    logger.info(
        "Task completed",
        extra={
            "task_id": task_id,
            "task_name": task.name,
            "state": state,
            "result": str(retval)[:200],  # Limitar tamanho do log
            "timestamp": time.time()
        }
    )

@task_failure.connect
def task_failure_handler(sender=None, task_id=None, exception=None, traceback=None, einfo=None, **kwds):
    logger.error(
        "Task failed",
        extra={
            "task_id": task_id,
            "task_name": sender.name,
            "exception": str(exception),
            "traceback": traceback,
            "timestamp": time.time()
        }
    )

# Tasks específicas
@app.task(bind=True, max_retries=3)
def process_xml_task(self, xml_data: str, company_id: str):
    try:
        # Lógica de processamento
        result = process_xml_logic(xml_data, company_id)
        return result
    except Exception as exc:
        logger.error(f"XML processing failed: {exc}")
        raise self.retry(exc=exc, countdown=60 * (2 ** self.request.retries))

@app.task(bind=True, max_retries=5)
def send_to_sefaz_task(self, gnre_id: str):
    try:
        # Lógica de envio para SEFAZ
        result = send_to_sefaz_logic(gnre_id)
        return result
    except SefazTemporaryException as exc:
        # Retry para erros temporários
        raise self.retry(exc=exc, countdown=300)  # 5 minutos
    except SefazPermanentException as exc:
        # Não retry para erros permanentes
        logger.error(f"Permanent SEFAZ error for GNRE {gnre_id}: {exc}")
        raise
```

### 8.2. Dashboard de Monitoramento
```python
# src/api/monitoring.py
from fastapi import APIRouter, Depends
from celery.result import AsyncResult
from src.worker.celery_app import app as celery_app

router = APIRouter(prefix="/monitoring", tags=["monitoring"])

@router.get("/queue-stats")
async def get_queue_stats():
    """Estatísticas das filas Celery"""
    inspect = celery_app.control.inspect()

    # Estatísticas ativas
    active_tasks = inspect.active()
    scheduled_tasks = inspect.scheduled()
    reserved_tasks = inspect.reserved()

    # Estatísticas por fila
    queue_stats = {}
    for worker, tasks in (active_tasks or {}).items():
        queue_stats[worker] = {
            "active": len(tasks),
            "scheduled": len(scheduled_tasks.get(worker, [])),
            "reserved": len(reserved_tasks.get(worker, []))
        }

    return {
        "workers": queue_stats,
        "total_active": sum(len(tasks) for tasks in (active_tasks or {}).values()),
        "total_scheduled": sum(len(tasks) for tasks in (scheduled_tasks or {}).values())
    }

@router.get("/task/{task_id}")
async def get_task_status(task_id: str):
    """Status de uma task específica"""
    result = AsyncResult(task_id, app=celery_app)

    return {
        "task_id": task_id,
        "status": result.status,
        "result": result.result,
        "traceback": result.traceback,
        "date_done": result.date_done
    }
```

## 🔄 Versionamento de API

### 9.1. Estratégia de Versionamento
```python
# src/api/versioning.py
from fastapi import APIRouter, Header, HTTPException
from typing import Optional
import semver

class APIVersioning:
    CURRENT_VERSION = "1.0.0"
    SUPPORTED_VERSIONS = ["1.0.0"]
    DEPRECATED_VERSIONS = []

    @staticmethod
    def validate_version(version: Optional[str] = Header(None, alias="API-Version")):
        if not version:
            version = APIVersioning.CURRENT_VERSION

        if version not in APIVersioning.SUPPORTED_VERSIONS:
            raise HTTPException(
                status_code=400,
                detail=f"API version {version} not supported. Supported versions: {APIVersioning.SUPPORTED_VERSIONS}"
            )

        if version in APIVersioning.DEPRECATED_VERSIONS:
            # Log warning para versões depreciadas
            logger.warning(f"Deprecated API version {version} used")

        return version

# Uso nos routers
v1_router = APIRouter(prefix="/api/v1", dependencies=[Depends(APIVersioning.validate_version)])

@v1_router.get("/gnre")
async def list_gnres_v1():
    # Implementação v1
    pass

# Quando houver v2
v2_router = APIRouter(prefix="/api/v2")

@v2_router.get("/gnre")
async def list_gnres_v2():
    # Implementação v2 com breaking changes
    pass
```

---

*Este documento é atualizado com cada release do backend e revisado pela equipe de desenvolvimento.*
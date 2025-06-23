# Planejamento Detalhado do Banco de Dados e Armazenamento da Aplicação de Geração de GNRE

## 📊 Visão Geral

Este documento detalha a estratégia de dados para o sistema GNRE, incluindo modelagem, segurança, performance, backup/recovery e observabilidade.

## 🎯 SLA/SLO para Dados

| Métrica | Target | Medição |
|---------|--------|---------|
| Disponibilidade | 99.9% | Uptime mensal |
| Tempo de Resposta | P95 < 100ms | Queries simples |
| Backup Recovery | RTO: 15min, RPO: 1h | Testes mensais |
| Consistência | 100% | Transações ACID |

## 5. Banco de Dados e Armazenamento

### 5.1. Banco de Dados (Supabase / PostgreSQL)

*   **Escolha:** `Supabase (PostgreSQL)` é a escolha unânime e robusta para o banco de dados, oferecendo PostgreSQL gerenciado, autenticação e RLS.
*   **Modelo de Dados:**
    *   Tabelas: `companies`, `users`, `gnres`, `certificates`, `subscriptions`, `api_tokens`, `audit_log`, `gnre_batches`, `gnre_batch_items`.
    *   Campos de auditoria (`created_at`, `updated_at`, `user_id` para ações críticas).
    *   Enums para status (`GNREStatus`, `UserRole`).
*   **Segurança:**
    *   **Row-Level Security (RLS):** Ativação e políticas explícitas em todas as tabelas com dados de clientes para garantir que usuários só acessem seus próprios dados.
    *   **Criptografia em Nível de Coluna:** Uso de `pgcrypto` para criptografar campos ultra-sensíveis (ex: senha do certificado digital).
    *   **Gerenciamento de Acesso:** Roles no PostgreSQL com permissões limitadas para a API.
    *   **Backups:** Backups diários criptografados e geograficamente redundantes.
    *   **MFA:** Habilitar MFA para administradores do Supabase.
    *   **Políticas de Senha Forte:** Integradas com o Supabase Auth.
*   **Performance:**
    *   Criação de índices em colunas frequentemente consultadas (`company_id`, `status`, `created_at`).
    *   Índices para busca de texto (`to_tsvector`).
    *   Otimização de queries.

### 5.2. Armazenamento de Objetos (MinIO)

*   **Escolha:** `MinIO` é a escolha consistente para armazenamento de objetos S3-compatível e auto-hospedado.
*   **Estrutura de Buckets:**
    *   `gnre-xmls`: XMLs originais.
    *   `gnre-pdfs`: PDFs das guias.
    *   `digital-certificates`: Certificados digitais (acesso restrito).
*   **Estrutura de Pastas (Object Keys):** Organização por `{company_id}/{year}/{month}/{file_id}.ext`.
*   **Segurança e Acesso:**
    *   Buckets **NÃO PÚBLICOS**.
    *   Acesso via API usando chaves de acesso (`Access Key/Secret Key`).
    *   `URLs Pré-Assinadas (Presigned URLs)` com tempo de expiração curto para downloads no frontend.
    *   Políticas de acesso restritivas (`IAM`) no MinIO, com "negação por padrão".
    *   Validação do lado do servidor e varredura de malware (`ClamAV`) em uploads.
    *   Criptografia em repouso (`SSE-S3`) e em trânsito (`HTTPS/TLS`).
    *   Logs de acesso e alertas para atividades anormais.

## 2.3. Definir Modelos de Dados (Supabase/PostgreSQL):

*   **Revisar e Finalizar Schema SQL:**
    *   `companies`: `id`, `owner_id`, `name`, `cnpj`, `created_at`, `updated_at`.
    *   `users`: `id`, `email`, `password_hash`, `name`, `role`, `company_id`, `created_at`, `updated_at`.
    *   `gnres`: `id`, `company_id`, `nfe_key`, `nfe_number`, `client_name`, `uf`, `due_date`, `payment_date`, `amount`, `status`, `protocol_number`, `xml_storage_path`, `pdf_storage_path`, `created_at`, `updated_at`.
    *   `certificates`: `id`, `company_id`, `name`, `validity`, `encrypted_file_path`, `encrypted_password`, `is_active`, `created_at`.
    *   `api_tokens`: `id`, `company_id`, `token_hash`, `created_at`, `last_used_at`.
    *   `gnre_batches`: `id`, `company_id`, `status`, `sent_at`, `response_message`.
    *   `gnre_batch_items`: `gnre_id`, `batch_id` (tabela de junção).
    *   `subscriptions`: `id`, `company_id`, `gateway_customer_id`, `plan_type`, `status`, `current_period_end`.
    *   `audit_log`: `id`, `user_id`, `company_id`, `action`, `details` (JSONB), `created_at`.
*   **Definir Enums:** `UserRole` (`ADMIN`, `USER`), `GNREStatus` (`PENDENTE`, `PROCESSANDO`, `GERADO`, `PAGO`, `CANCELADO`, `ERRO`).
*   **Índices para Performance:**
    *   `idx_gnres_company_id` on `gnres(company_id)`.
    *   `idx_gnres_status` on `gnres(status)`.
    *   `idx_gnres_created_at` on `gnres(created_at DESC)`.
    *   `idx_gnres_compound` on `gnres(company_id, status, created_at DESC)`.
    *   `idx_gnres_cliente_gin` on `gnres` using `gin(to_tsvector('portuguese', cliente))` for full-text search.
## 🔄 Estratégia de Backup e Disaster Recovery

### 6.1. Backup Strategy
```yaml
backup_strategy:
  frequency:
    full_backup: "daily_at_02:00_UTC"
    incremental: "every_6_hours"
    transaction_log: "continuous"

  retention:
    daily: "30_days"
    weekly: "12_weeks"
    monthly: "12_months"
    yearly: "7_years"

  storage:
    primary: "supabase_managed"
    secondary: "aws_s3_cross_region"
    encryption: "AES_256"
```

### 6.2. Disaster Recovery
- **RTO (Recovery Time Objective):** 15 minutos
- **RPO (Recovery Point Objective):** 1 hora
- **Testes de Recovery:** Mensais automatizados
- **Failover:** Automático para região secundária

## 📊 Observabilidade e Monitoramento

### 7.1. Métricas Críticas
```sql
-- Métricas de Performance
SELECT
  schemaname,
  tablename,
  n_tup_ins as inserts,
  n_tup_upd as updates,
  n_tup_del as deletes,
  n_live_tup as live_tuples,
  n_dead_tup as dead_tuples
FROM pg_stat_user_tables;

-- Queries Lentas
SELECT
  query,
  calls,
  total_time,
  mean_time,
  rows
FROM pg_stat_statements
WHERE mean_time > 1000
ORDER BY mean_time DESC;
```

### 7.2. Alertas Configurados
- **Conexões ativas > 80%** da capacidade
- **Query time > 5 segundos**
- **Disk usage > 85%**
- **Backup failure**
- **Replication lag > 30 segundos**

## 🔧 Migrações e CI/CD

### 8.1. Estratégia de Migração
```python
# alembic/versions/001_initial_schema.py
from alembic import op
import sqlalchemy as sa

def upgrade():
    # Forward migration
    op.create_table(
        'companies',
        sa.Column('id', sa.UUID(), primary_key=True),
        sa.Column('name', sa.String(255), nullable=False),
        sa.Column('cnpj', sa.String(14), nullable=False, unique=True),
        sa.Column('created_at', sa.DateTime(), server_default=sa.func.now()),
        sa.Column('updated_at', sa.DateTime(), onupdate=sa.func.now())
    )

def downgrade():
    # Rollback migration
    op.drop_table('companies')
```

### 8.2. Pipeline de Migração
```yaml
# .github/workflows/db-migration.yml
name: Database Migration
on:
  push:
    paths: ['migrations/**']

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - name: Run Migration (Staging)
        run: alembic upgrade head
        env:
          DATABASE_URL: ${{ secrets.STAGING_DB_URL }}

      - name: Run Tests
        run: pytest tests/integration/

      - name: Run Migration (Production)
        if: github.ref == 'refs/heads/main'
        run: alembic upgrade head
        env:
          DATABASE_URL: ${{ secrets.PROD_DB_URL }}
```

## 📈 Escalabilidade e Performance

### 9.1. Estratégia de Scaling
```mermaid
graph TD
    A[Current: Single PostgreSQL] --> B{Load > 80%?}
    B -->|Yes| C[Read Replicas]
    C --> D{Still Overloaded?}
    D -->|Yes| E[Horizontal Partitioning]
    E --> F[Sharding by company_id]

    B -->|No| G[Optimize Queries]
    G --> H[Add Indexes]
    H --> I[Query Caching]
```

### 9.2. Particionamento de Tabelas
```sql
-- Particionamento da tabela gnres por data
CREATE TABLE gnres_2024 PARTITION OF gnres
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE TABLE gnres_2025 PARTITION OF gnres
FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```

## 🔒 Compliance e Retenção de Dados

### 10.1. Política de Retenção
```yaml
data_retention:
  gnres:
    active: "7_years"  # Obrigação fiscal
    archived: "indefinite"

  audit_logs:
    security: "5_years"
    access: "2_years"

  user_data:
    active_users: "while_active"
    inactive_users: "2_years_after_last_login"
    deleted_users: "30_days_grace_period"
```

### 10.2. LGPD Compliance
```sql
-- Função para anonimização de dados
CREATE OR REPLACE FUNCTION anonymize_user_data(user_id UUID)
RETURNS VOID AS $$
BEGIN
    UPDATE users SET
        email = 'anonymized_' || user_id || '@deleted.local',
        name = 'Usuário Removido',
        phone = NULL,
        updated_at = NOW()
    WHERE id = user_id;

    -- Manter dados fiscais por obrigação legal
    UPDATE gnres SET
        client_name = 'Cliente Anonimizado'
    WHERE company_id IN (
        SELECT company_id FROM users WHERE id = user_id
    );
END;
$$ LANGUAGE plpgsql;
```

## 💰 Monitoramento de Custos

### 11.1. Cost Tracking
```yaml
cost_monitoring:
  supabase:
    database_size: "monitor_daily"
    bandwidth: "monitor_hourly"
    compute_hours: "track_per_project"

  minio:
    storage_used: "monitor_daily"
    requests: "monitor_hourly"
    bandwidth: "track_egress"

  alerts:
    monthly_budget_80_percent: "email_admin"
    unexpected_spike_50_percent: "slack_alert"
```

---

*Este documento é atualizado automaticamente com cada migração de schema e revisado mensalmente pela equipe de dados.*
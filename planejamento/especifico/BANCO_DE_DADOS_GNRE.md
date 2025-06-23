# Planejamento Detalhado do Banco de Dados e Armazenamento da Aplicação de Geração de GNRE

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
## Oportunidades de Aprimoramento (Banco de Dados e Armazenamento)

*   **Estratégia de Backup e Restore:** Além de backups diários, detalhar a estratégia de restore (RTO - Recovery Time Objective, RPO - Recovery Point Objective) e como os restores serão testados periodicamente.
*   **Monitoramento de Banco de Dados:** Especificar métricas cruciais a serem monitoradas (conexões ativas, queries lentas, uso de CPU/memória/disco, taxa de acertos de cache) e ferramentas para isso (Supabase Dashboard, Prometheus/Grafana).
*   **Estratégia de Migração de Schema:** Como as migrações de banco de dados serão gerenciadas em produção para garantir zero-downtime (ex: uso de ferramentas como `Alembic` com estratégias de migração reversíveis).
*   **Plano de Escalabilidade Futura:** Se o PostgreSQL se tornar um gargalo, quais seriam as próximas etapas (read replicas, sharding, particionamento de tabelas)?
*   **Gerenciamento de Dados Sensíveis:** Reforçar a política de retenção de dados e logs, e como dados sensíveis serão anonimizados ou excluídos após um período.
*   **Observabilidade Unificada:** Detalhar como o Banco de Dados e o Armazenamento contribuirão para o sistema de observabilidade (ex: logs de acesso, métricas de performance e uso).
*   **CI/CD por Componente:** Mencionar como o pipeline de CI/CD se aplica ao Banco de Dados (migrações automatizadas, testes de schema).
*   **Gerenciamento de Configurações e Segredos:** Detalhar a estratégia para gerenciar variáveis de ambiente e segredos em diferentes ambientes (desenvolvimento, homologação, produção) de forma segura (ex: `.env` para dev, Vault/KMS para produção).
*   **Controle de Versão e Branching:** Reforçar o uso de Git e uma estratégia de branching clara (ex: GitFlow, Trunk-Based Development com feature flags) para gerenciar o desenvolvimento em paralelo.
*   **Monitoramento de Custos:** Adicionar uma nota sobre o monitoramento contínuo de custos dos serviços (Supabase, MinIO) para otimização.
## Oportunidades de Aprimoramento (Banco de Dados e Armazenamento)

*   **Estratégia de Backup e Restore:** Além de backups diários, detalhar a estratégia de restore (RTO - Recovery Time Objective, RPO - Recovery Point Objective) e como os restores serão testados periodicamente.
*   **Monitoramento de Banco de Dados:** Especificar métricas cruciais a serem monitoradas (conexões ativas, queries lentas, uso de CPU/memória/disco, taxa de acertos de cache) e ferramentas para isso (Supabase Dashboard, Prometheus/Grafana).
*   **Estratégia de Migração de Schema:** Como as migrações de banco de dados serão gerenciadas em produção para garantir zero-downtime (ex: uso de ferramentas como `Alembic` com estratégias de migração reversíveis).
*   **Plano de Escalabilidade Futura:** Se o PostgreSQL se tornar um gargalo, quais seriam as próximas etapas (read replicas, sharding, particionamento de tabelas)?
*   **Gerenciamento de Dados Sensíveis:** Reforçar a política de retenção de dados e logs, e como dados sensíveis serão anonimizados ou excluídos após um período.
*   **Observabilidade Unificada:** Detalhar como o Banco de Dados e o Armazenamento contribuirão para o sistema de observabilidade (ex: logs de acesso, métricas de performance e uso).
*   **CI/CD por Componente:** Mencionar como o pipeline de CI/CD se aplica ao Banco de Dados (migrações automatizadas, testes de schema).
*   **Gerenciamento de Configurações e Segredos:** Detalhar a estratégia para gerenciar variáveis de ambiente e segredos em diferentes ambientes (desenvolvimento, homologação, produção) de forma segura (ex: `.env` para dev, Vault/KMS para produção).
*   **Controle de Versão e Branching:** Reforçar o uso de Git e uma estratégia de branching clara (ex: GitFlow, Trunk-Based Development com feature flags) para gerenciar o desenvolvimento em paralelo.
*   **Monitoramento de Custos:** Adicionar uma nota sobre o monitoramento contínuo de custos dos serviços (Supabase, MinIO) para otimização.
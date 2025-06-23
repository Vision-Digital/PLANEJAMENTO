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
## Oportunidades de Aprimoramento (Backend)

*   **Tratamento de Erros Padronizado:** Definir um formato padrão para respostas de erro da API (ex: seguir a especificação `Problem Details` RFC 7807) para facilitar o consumo pelo Frontend e Aplicação Local.
*   **Validação de Regras de Negócio Complexas:** Além da validação de DTOs com `Pydantic`, detalhar como as regras de negócio complexas (ex: cálculo de impostos, validações fiscais específicas do XML) serão implementadas e testadas, talvez com a menção de uma camada de domínio ou serviços de negócio.
*   **Gerenciamento de Transações:** Esclarecer a estratégia de gerenciamento de transações de banco de dados, especialmente em operações que envolvem múltiplos passos ou serviços externos (ex: integração SEFAZ, upload para MinIO). Como garantir atomicidade?
*   **Idempotência de Endpoints:** Para endpoints que podem ser chamados múltiplas vezes (ex: `POST /gnre/from-local-agent`), descrever como a idempotência será garantida para evitar duplicação de dados ou processamento.
*   **Monitoramento de Filas:** Detalhar como o monitoramento das filas (`Celery/Redis`) será implementado para garantir que os jobs estão sendo processados corretamente, identificar jobs falhos e gargalos.
*   **Estratégia de Versionamento de API:** Embora mencione `/api/v1/`, detalhar a política de versionamento (ex: major version changes only, deprecation policy).
*   **Observabilidade Unificada:** Detalhar como o Backend contribuirá para o sistema de observabilidade (ex: logs estruturados com `correlation IDs`, métricas específicas, instrumentação para `distributed tracing` com OpenTelemetry).
*   **CI/CD por Componente:** Mencionar como o pipeline de CI/CD se aplica ao Backend (testes automatizados, builds, deploys específicos).
*   **Gerenciamento de Configurações e Segredos:** Detalhar a estratégia para gerenciar variáveis de ambiente e segredos em diferentes ambientes (desenvolvimento, homologação, produção) de forma segura (ex: `.env` para dev, Vault/KMS para produção).
*   **Documentação de APIs (Swagger/OpenAPI):** Reforçar a importância de manter a documentação da API atualizada e como isso será parte do processo de desenvolvimento (ex: gerada automaticamente a partir do código, revisões periódicas).
*   **Controle de Versão e Branching:** Reforçar o uso de Git e uma estratégia de branching clara (ex: GitFlow, Trunk-Based Development com feature flags) para gerenciar o desenvolvimento em paralelo.
*   **Monitoramento de Custos:** Adicionar uma nota sobre o monitoramento contínuo de custos dos serviços (servidores, Redis, etc.) para otimização.
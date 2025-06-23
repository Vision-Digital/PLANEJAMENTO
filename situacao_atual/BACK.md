# Análise Detalhada do Projeto GNRE API

Este projeto é uma API para emissão, consulta e gerenciamento de Guias de Recolhimento GNRE (Guia Nacional de Recolhimento de Tributos Estaduais). Vou detalhar a stack tecnológica e a estrutura do projeto:

## Stack Tecnológica

### Backend
- **Python 3.11**: Linguagem principal do projeto
- **FastAPI (v0.115.12)**: Framework web para construção da API REST
- **Uvicorn**: Servidor ASGI para execução da aplicação
- **Gunicorn**: Servidor HTTP WSGI para produção
- **Pydantic**: Validação de dados e gerenciamento de configurações
- **Python-Jose**: Implementação de JWT para autenticação
- **Passlib/Argon2/Bcrypt**: Hashing seguro de senhas
- **Cryptography**: Operações criptográficas (provavelmente para certificados digitais)
- **MinIO Client**: Armazenamento de objetos compatível com S3

### Armazenamento
- **Sistema de arquivos JSON**: Armazenamento de dados em arquivos JSON (pasta 'data/')
- **MinIO**: Serviço de armazenamento de objetos para backup e sincronização

### Infraestrutura
- **Docker**: Containerização da aplicação
- **Docker Compose**: Orquestração de múltiplos containers (API + MinIO)
- **Heroku**: Plataforma de deploy em nuvem (suporte configurado)

## Estrutura do Projeto

### Componentes Principais
1. **API REST (FastAPI)**:
   - Rotas para gerenciamento de empresas
   - Rotas para emissão e consulta de GNRE
   - Sistema de autenticação baseado em tokens JWT
   - Middleware CORS para integração com frontends

2. **Serviço de Sincronização MinIO**:
   - Backup automático de arquivos (certificados, dados, uploads)
   - Estratégias de sincronização configuráveis
   - Intervalo de sincronização ajustável

3. **Integração com SEFAZ**:
   - Comunicação com endpoints da SEFAZ para emissão de GNRE
   - Suporte a ambientes de homologação e produção
   - Processamento de certificados digitais

### Arquivos Importantes
- `app/main.py`: Ponto de entrada da aplicação FastAPI
- `run.py`: Script para execução da API
- `Dockerfile`: Configuração para containerização
- `docker-compose.yml`: Orquestração de serviços (API + MinIO)
- `requirements.txt`: Dependências Python
- `Procfile`: Configuração para deploy no Heroku
- `.env`: Configurações de ambiente (portas, chaves, endpoints)

### Opções de Execução
1. **Versão Completa**: Com todas as funcionalidades (certificados, SEFAZ, PDF, MinIO)
2. **Versão Simplificada**: Versão mais leve sem dependências complexas
3. **Deploy Heroku**: Configuração otimizada para cloud

## Funcionalidades Principais

1. **Gerenciamento de Empresas**:
   - Cadastro de empresas e certificados digitais

2. **Emissão de GNRE**:
   - Criação e envio de lotes
   - Consulta de configurações por UF
   - Consulta de resultados de lotes
   - Geração de PDF das guias

3. **Autenticação e Segurança**:
   - Sistema de tokens JWT por empresa
   - Hashing seguro de senhas
   - Chaves secretas configuráveis

4. **Sincronização e Backup**:
   - Integração com MinIO para armazenamento
   - Sincronização automática de pastas importantes
   - Estratégias de sincronização configuráveis

5. **Ambientes**:
   - Suporte a ambientes de homologação e produção da SEFAZ

## Configurações
- Porta da API configurável (atualmente 8001)
- Chaves de segurança e algoritmos de criptografia
- Endpoints da SEFAZ para homologação e produção
- Configurações do MinIO (endpoint, credenciais, bucket)
- Estratégias e intervalos de sincronização

O projeto demonstra uma arquitetura bem estruturada para um sistema de emissão de GNRE, com foco em flexibilidade de implantação (local, Docker, Heroku) e facilidade de uso.
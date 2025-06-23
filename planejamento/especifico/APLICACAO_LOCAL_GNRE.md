# Planejamento Detalhado da Aplicação Local da Aplicação de Geração de GNRE

## 4. Aplicação Local (Monitor de Pasta)

A aplicação local será um agente leve e robusto, executado no ambiente do cliente, responsável por monitorar pastas e enviar XMLs para a API.

### 4.1. Stack Tecnológica Consolidada

*   **Linguagem:** `Go (Golang)` é a escolha mais robusta e segura, conforme `GEMINI`. Embora `APP.md` sugira `.NET 8.0` e `Claude` sugira `Node.js/Electron`, o `Go` gera binários compilados, leves, de alta performance e multiplataforma, com menor superfície de ataque e sem dependências pesadas.
*   **Bibliotecas:** `fsnotify` para monitoramento eficiente de arquivos, `net/http` para comunicação com a API, e bibliotecas seguras de parsing de XML.
*   **Distribuição:** Binário assinado digitalmente (`Code Signing`) para Windows, evitando alertas de segurança.
*   **Instalador:** `NSIS` para criação de instaladores Windows.

### 4.2. Funcionalidades Principais

*   **Monitoramento:** Monitoramento em tempo real de pasta configurável para novos XMLs.
*   **Processamento:** Leitura, parse e extração de dados do XML. Aplicação de regras de negócio para identificar necessidade de GNRE.
*   **Comunicação:** Envio de dados para a API via `HTTPS` com autenticação por `token de API`.
*   **Pós-processamento:** Movimentação de XMLs para pastas `_processing`, `_processed` ou `_error` com logs detalhados.
*   **Configuração:** Leitura de arquivo de configuração local.

### 4.3. Foco em Segurança (Aplicação Local)

*   **Configuração Segura:** Armazenamento de `token_da_api` em armazenamento de credenciais nativo do SO (`Windows Credential Manager`, `Keychain`, `Secret Service API/Keyring`), não em arquivos de texto plano.
*   **Comunicação Segura (Prevenção de MITM):** Implementação de `Certificate Pinning` para confiar apenas em certificados SSL específicos do domínio da API.
*   **Manuseio de Arquivos Seguro:**
    *   Sanitização de nomes de arquivos para prevenir `Path Traversal` (`../`).
    *   Configuração do parser XML para desabilitar resolução de entidades externas (`XXE`).
    *   Validação de tamanho e extensão do arquivo antes do processamento.
    *   Verificação de `magic number` (cabeçalho do arquivo) para garantir que é um XML válido.
*   **Integridade do Binário:** Fornecimento de `hash SHA-256` do binário para verificação de integridade pelo usuário.
*   **Permissões Mínimas:** Execução do agente com privilégios limitados, acessando apenas a pasta monitorada.
*   **Sandboxing:** Considerar isolamento via contêiner ou `AppContainer` (Windows) para reduzir danos em caso de falha.

## 2.2. Definir Estrutura de Projeto e Padrões (Aplicação Local)

*   **Organização de Pastas:** Definir estrutura de pastas (ex: `cmd/monitorxml`, `internal/config`, `internal/api`, `internal/xmlprocessor`).
*   **Design Patterns:** `Modular Design`.

## 6.2. Configuração de Docker (Aplicação Local)

*   [ ] Criar `Dockerfile` para a Aplicação Local (Go 1.22.x).

## 8.2. Configuração de Framework de Testes (Aplicação Local)

*   [ ] **Aplicação Local:** Configurar framework de testes nativo do Go.

## 8.3. Desenvolver Testes (Aplicação Local)

*   [ ] **Testes Unitários:**
    *   [ ] Escrever testes para funções e componentes individuais (Aplicação Local).
    *   [ ] Cobrir lógica de negócio, validações e utilitários.
*   [ ] **Testes de Integração:**
    *   [ ] Testar integração Aplicação Local-Backend.

## 9. Justificativas para Divergências (Aplicação Local)

*   **Aplicação Local (Go vs .NET/Windows Forms vs Node.js/Electron):**
    *   **Sua escolha (`APP.md`):** .NET 8.0, Windows Forms.
    *   **Outras sugestões:** Go (`GEMINI`), Node.js/Electron (`Claude`).
    *   **Justificativa:** A escolha do `Go` para a aplicação local é **fortemente recomendada** em detrimento de `.NET/Windows Forms` ou `Node.js/Electron`.
        *   **Performance e Leveza:** Binários Go são extremamente leves e performáticos, ideais para rodar em segundo plano sem consumir muitos recursos do cliente. `.NET/Windows Forms` pode ser mais pesado e `Electron` é notoriamente conhecido por alto consumo de RAM.
        *   **Multiplataforma:** Embora o foco seja Windows, Go facilita a criação de binários para Linux/macOS se houver necessidade futura.
        *   **Segurança:** Go oferece um controle mais granular sobre o sistema de arquivos e rede, facilitando a implementação de medidas de segurança como `Certificate Pinning` e sanitização de `Path Traversal/XXE` de forma mais robusta e com menor superfície de ataque do que um ambiente de runtime como Node.js ou .NET. A distribuição como um único binário simplifica a instalação e reduz a complexidade de dependências.
        *   **Distribuição:** A criação de instaladores com `NSIS` é compatível com binários Go.
    *   Portanto, a sugestão é **migrar a aplicação local para Go** para maximizar segurança, performance e facilidade de distribuição.
## Oportunidades de Aprimoramento (Aplicação Local)

*   **Resiliência e Tratamento de Falhas:** Expandir sobre como a aplicação local lida com falhas de rede, API offline ou erros de processamento de XML (mecanismos de retry com backoff exponencial, filas de mensagens locais para reprocessamento).
*   **Estratégia de Atualização:** Como a aplicação local será atualizada? Será um auto-update silencioso, notificação de nova versão com download manual, ou via instalador? Isso impacta a experiência do usuário e a segurança.
*   **Configuração Inicial e Gerenciamento de Credenciais:** Detalhar o fluxo de configuração inicial (como o usuário obtém e insere o `token_da_api` pela primeira vez) e reforçar o uso de armazenamento de credenciais nativo do SO.
*   **Logging e Telemetria:** Como os logs da aplicação local serão coletados e, se possível, centralizados (mesmo que de forma básica, enviando para a API ou um serviço de log). Considerar métricas básicas de uso e erros.
*   **Segurança do Processo:** Mencionar a execução com privilégios mínimos e, se aplicável, a assinatura de código para o binário final.
*   **Observabilidade Unificada:** Detalhar como a Aplicação Local contribuirá para o sistema de observabilidade (ex: logs estruturados com `correlation IDs`, métricas específicas).
*   **CI/CD por Componente:** Mencionar como o pipeline de CI/CD se aplica à Aplicação Local (testes automatizados, builds, deploys específicos).
*   **Gerenciamento de Configurações e Segredos:** Detalhar a estratégia para gerenciar variáveis de ambiente e segredos em diferentes ambientes (desenvolvimento, homologação, produção) de forma segura (ex: `.env` para dev, Vault/KMS para produção).
*   **Controle de Versão e Branching:** Reforçar o uso de Git e uma estratégia de branching clara (ex: GitFlow, Trunk-Based Development com feature flags) para gerenciar o desenvolvimento em paralelo.
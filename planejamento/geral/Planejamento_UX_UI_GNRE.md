### 3. Agente 3 – UX/UI Designer: Plano Detalhado

**Objetivo:** Criar design system, wireframes, layouts modernos e minimalistas, considerando Dark Mode, responsividade e usabilidade para a aplicação GNRE.

#### 3.1. Análise de Documentos (Requisitos + Arquitetura)

*   **Revisão Aprofundada:** Reanalisar o [`Planejamento_Detalhado_GNRE.md`](Planejamento_Detalhado_GNRE.md) para garantir que todos os fluxos de usuário, pontos de entrada e telas prioritárias (Login/Registro, Dashboard, Geração de GNRE, Configurações, Assinatura e Uso) sejam compreendidos em detalhes.
*   **Impacto da Segurança na UX:** Considerar como funcionalidades de segurança (MFA, políticas de senha, feedback de erros de autenticação, logs de auditoria) serão apresentadas ao usuário de forma clara e intuitiva.

#### 3.2. Design System (Base: Tailwind CSS, SHADCN/UI ou Tremor React)

O design system será a base para a consistência visual e funcional da aplicação.

*   **Paleta de Cores:**
    *   **Primárias:** Definir uma cor principal que represente a marca (ex: um tom de azul ou verde corporativo).
    *   **Secundárias:** Cores complementares para elementos de destaque.
    *   **Acento:** Cores para interações (botões de ação, links).
    *   **Status:** Cores para sucesso (verde), aviso (amarelo/laranja), erro (vermelho) e informação (azul claro).
    *   **Modo Escuro (Dark Mode):** Criar uma paleta correspondente para o modo escuro, garantindo contraste e legibilidade. As cores de fundo serão mais escuras, e as cores de texto e elementos, mais claras.
*   **Tipografia:**
    *   **Fontes:** Escolher uma família de fontes moderna e legível (ex: Inter, Poppins, Roboto).
    *   **Hierarquia:** Definir tamanhos e pesos para títulos (H1-H6), subtítulos, corpo de texto, legendas e elementos de UI, garantindo clareza e hierarquia visual.
*   **Ícones:**
    *   **Biblioteca:** Utilizar `Lucide React` para ícones, garantindo consistência e fácil integração.
    *   **Estilo:** Definir um estilo de ícones (outline, filled) e tamanhos padrão.
*   **Componentes Básicos (Inspirados em SHADCN/UI, Tremor React e inspirações do 21st MCP):**
    *   **Botões:**
        *   Tipos: Primário, Secundário, Perigo, Link, Ícone.
        *   Estados: Normal, Hover, Foco, Ativo, Desabilitado, Loading.
        *   Tamanhos: Pequeno, Médio, Grande.
    *   **Formulários:**
        *   Inputs: Texto, Número, E-mail, Senha, Textarea.
        *   Seletores: Checkboxes, Radio Buttons, Selects (Dropdowns), Date Pickers.
        *   Validação: Estados de erro, mensagens de feedback.
    *   **Modais e Dialogs:** Para confirmações, formulários complexos e exibições de detalhes.
    *   **Toasts e Notificações:** Para feedback rápido ao usuário (sucesso, erro, aviso).
    *   **Badges e Tags de Status:** Para indicar o status de GNREs (Pendente, Processando, Gerado, Pago, Cancelado, Erro) e outros elementos.
    *   **Tabelas:**
        *   Funcionalidades: Paginação, filtros (por status, data, nome do cliente), ordenação de colunas, seleção de linhas, ações por linha (download PDF, visualizar detalhes).
        *   Design: Linhas zebradas, cabeçalhos fixos, responsividade.
    *   **Cards e Painéis:** Para organizar informações no Dashboard e em outras telas.
        *   *Inspiração:* Os componentes "Stats Section" e "Stats" do 21st MCP são excelentes exemplos de como exibir KPIs em cards, com indicadores de mudança e descrições claras. Serão adaptados para exibir métricas como "GNREs Geradas no Mês", "Valor Total Processado", "GNREs Pendentes", etc.
    *   **Loaders e Spinners:** Para indicar estados de carregamento.
    *   **Navegação (Sidebar/Header):
        *   *Inspiração:* O componente "Toggle nav" do 21st MCP pode ser adaptado para a barra lateral de navegação, permitindo que o usuário a recolha ou expanda, otimizando o espaço da tela.
        *   Elementos: Links de navegação, ícones, indicadores de notificação, avatar do usuário.
*   **Guia de Estilo:** Documentar todos os componentes, diretrizes de uso, exemplos de código e princípios de design para garantir a adesão da equipe de desenvolvimento.

#### 3.3. Layouts e Wireframes (Protótipos Interativos)

Serão criados protótipos de alta fidelidade no Figma (ou ferramenta similar), com foco em responsividade e acessibilidade.

*   **Fluxo de Autenticação:**
    *   **Login:** Tela simples e limpa com campos de e-mail/usuário e senha, opção "Esqueci a senha", e link para registro.
    *   **Registro:** Formulário para criação de conta (nome, e-mail, senha, confirmação de senha, nome da empresa, CNPJ).
    *   **Recuperação de Senha:** Fluxo de duas etapas (e-mail para redefinição, tela de nova senha).
    *   **MFA:** Tela para inserção de código de autenticação.
*   **Dashboard:**
    *   **Layout:** Visão geral com cards de KPIs (inspirados nos componentes "Stats Section" e "Stats"), gráficos de tendências (GNREs geradas por período, valor total), e um resumo das últimas GNREs ou lotes.
    *   **Elementos:** Header com navegação principal, sidebar (adaptada do "Toggle nav"), área de conteúdo.
*   **Tela de Geração de GNRE:**
    *   **Upload:** Área de drag & drop para XMLs, com feedback visual de upload e validação.
    *   **Tabela de GNREs:** Exibição das GNREs com colunas configuráveis (Nº NFe, Cliente, UF, Vencimento, Valor, Status, Protocolo), filtros avançados, paginação e ordenação.
    *   **Modais de Ação:** Modais para visualizar detalhes da GNRE, editar informações, baixar PDF, cancelar.
*   **Tela de Configurações:**
    *   **Abas:** Navegação por abas para:
        *   **Dados da Empresa:** Formulário para editar informações da empresa.
        *   **Certificado Digital:** Área para upload seguro, visualização de validade e status do certificado.
        *   **E-mail:** Configurações de notificação por e-mail.
        *   **Usuários:** Gerenciamento de usuários da empresa (adicionar, editar, remover, gerenciar papéis).
        *   **Integrações:** Exibição e geração de `API Token` para a aplicação local.
*   **Tela de Assinatura e Uso:**
    *   **Detalhes do Plano:** Exibição do plano atual, data de renovação, funcionalidades inclusas.
    *   **Medidor de Uso:** Gráfico ou barra de progresso mostrando o consumo de GNREs geradas no período.
    *   **Histórico de Faturas:** Tabela com faturas anteriores e opção de download.
    *   **Gerenciamento de Assinatura:** Botões para upgrade/downgrade de plano, cancelamento.
*   **Layout Responsivo:**
    *   **Abordagem Mobile-First:** Projetar primeiro para telas menores e depois adaptar para tablets e desktops.
    *   **Breakpoints:** Definir breakpoints para garantir que o layout se ajuste fluidamente a diferentes tamanhos de tela.
*   **Guidelines de Acessibilidade (WCAG 2.1 AA):**
    *   **Cores e Contraste:** Garantir contraste suficiente entre texto e fundo.
    *   **Navegação por Teclado:** Todos os elementos interativos devem ser acessíveis via teclado.
    *   **Leitores de Tela:** Utilizar atributos `aria-label`, `alt text` para imagens e estrutura semântica HTML.
    *   **Estados de Foco:** Indicar claramente o elemento focado.

#### 3.4. Documentação Final de UX/UI

*   **Entrega:**
    *   Link para o protótipo interativo no Figma (ou ferramenta similar).
    *   Documento detalhado do Design System (incluindo paleta de cores, tipografia, ícones, e especificações de todos os componentes).
    *   Wireframes e fluxos de usuário em formato visual.

**Artefato de Saída:** Documento/Protótipo de UX/UI + arquivos do design system (se gerados programaticamente ou mocks).

---

**Diagrama de Fluxo de Telas (Mermaid):**

```mermaid
graph TD
    A[Início] --> B{Usuário Logado?};
    B -- Não --> C[Login/Registro];
    C --> D[Dashboard];
    B -- Sim --> D;
    D --> E[Geração de GNRE];
    D --> F[Configurações];
    D --> G[Assinatura e Uso];
    E --> H[Detalhes da GNRE/Modal];
    F --> I[Dados da Empresa];
    F --> J[Certificado Digital];
    F --> K[E-mail];
    F --> L[Usuários];
    F --> M[Integrações/API Token];
    G --> N[Histórico de Faturas];
    G --> O[Gerenciamento de Assinatura];
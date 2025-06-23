# Planejamento Detalhado da Aplicação Local da Aplicação de Geração de GNRE

## 📱 Visão Geral

A aplicação local será um agente leve, robusto e auto-gerenciável, executado no ambiente do cliente, responsável por monitorar pastas, processar XMLs e comunicar-se de forma segura com a API.

## 🎯 Objetivos e SLA

| Métrica | Target | Medição |
|---------|--------|---------|
| Uptime | 99.9% | Monitoramento local |
| Latência de Detecção | < 5 segundos | Novo arquivo → processamento |
| Uso de CPU | < 5% | Em idle |
| Uso de RAM | < 50MB | Operação normal |
| Auto-recovery | < 30 segundos | Após falha de rede |

## 4. Aplicação Local (Monitor de Pasta)

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
## 🔄 Auto-Update e Gerenciamento de Versão

### 5.1. Estratégia de Auto-Update
```go
// auto_updater.go
package main

import (
    "crypto/sha256"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "os"
    "os/exec"
    "path/filepath"
    "time"
)

type VersionInfo struct {
    Version     string `json:"version"`
    DownloadURL string `json:"download_url"`
    Checksum    string `json:"checksum"`
    Critical    bool   `json:"critical"`
    ReleaseNotes string `json:"release_notes"`
}

func (a *Agent) checkForUpdates() error {
    resp, err := a.httpClient.Get(a.config.UpdateURL + "/latest")
    if err != nil {
        return fmt.Errorf("failed to check updates: %w", err)
    }
    defer resp.Body.Close()

    var versionInfo VersionInfo
    if err := json.NewDecoder(resp.Body).Decode(&versionInfo); err != nil {
        return fmt.Errorf("failed to decode version info: %w", err)
    }

    if versionInfo.Version != a.currentVersion {
        return a.performUpdate(&versionInfo)
    }

    return nil
}

func (a *Agent) performUpdate(version *VersionInfo) error {
    // Download new version
    newBinary, err := a.downloadUpdate(version.DownloadURL)
    if err != nil {
        return err
    }

    // Verify checksum
    if !a.verifyChecksum(newBinary, version.Checksum) {
        return fmt.Errorf("checksum verification failed")
    }

    // Replace current binary and restart
    return a.replaceAndRestart(newBinary)
}
```

### 5.2. Configuração Inicial via CLI
```go
// cli.go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strings"
    "syscall"

    "golang.org/x/term"
)

func (a *Agent) configure() error {
    fmt.Println("=== Configuração Inicial do GNRE Agent ===")

    // API Token
    fmt.Print("Token da API: ")
    tokenBytes, err := term.ReadPassword(int(syscall.Stdin))
    if err != nil {
        return err
    }
    token := string(tokenBytes)

    // Pasta de monitoramento
    fmt.Print("\nPasta para monitoramento: ")
    reader := bufio.NewReader(os.Stdin)
    watchPath, _ := reader.ReadString('\n')
    watchPath = strings.TrimSpace(watchPath)

    // Salvar configuração segura
    config := &Config{
        APIToken:  token,
        WatchPath: watchPath,
        APIBaseURL: "https://api.gnre.com/v1",
    }

    return a.saveSecureConfig(config)
}

func (a *Agent) saveSecureConfig(config *Config) error {
    // Salvar token no Windows Credential Manager
    return a.credentialManager.Store("gnre_agent", config.APIToken)
}
```

## 🛡️ Resiliência e Tratamento de Falhas

### 6.1. Retry com Backoff Exponencial
```go
// retry.go
package main

import (
    "context"
    "math"
    "time"
)

type RetryConfig struct {
    MaxRetries  int
    BaseDelay   time.Duration
    MaxDelay    time.Duration
    Multiplier  float64
}

func (a *Agent) sendWithRetry(ctx context.Context, data []byte) error {
    config := RetryConfig{
        MaxRetries: 5,
        BaseDelay:  1 * time.Second,
        MaxDelay:   30 * time.Second,
        Multiplier: 2.0,
    }

    var lastErr error

    for attempt := 0; attempt <= config.MaxRetries; attempt++ {
        if attempt > 0 {
            delay := time.Duration(float64(config.BaseDelay) *
                math.Pow(config.Multiplier, float64(attempt-1)))
            if delay > config.MaxDelay {
                delay = config.MaxDelay
            }

            select {
            case <-time.After(delay):
            case <-ctx.Done():
                return ctx.Err()
            }
        }

        if err := a.sendToAPI(data); err != nil {
            lastErr = err
            a.logger.Warn("Attempt failed",
                "attempt", attempt+1,
                "error", err,
                "next_retry_in", delay)
            continue
        }

        return nil
    }

    // Salvar na fila local para retry posterior
    return a.queueForLaterRetry(data, lastErr)
}
```

### 6.2. Fila Local para Offline
```go
// local_queue.go
package main

import (
    "encoding/json"
    "os"
    "path/filepath"
    "time"
)

type QueueItem struct {
    ID        string    `json:"id"`
    Data      []byte    `json:"data"`
    Timestamp time.Time `json:"timestamp"`
    Retries   int       `json:"retries"`
    LastError string    `json:"last_error"`
}

func (a *Agent) queueForLaterRetry(data []byte, err error) error {
    item := QueueItem{
        ID:        generateID(),
        Data:      data,
        Timestamp: time.Now(),
        Retries:   0,
        LastError: err.Error(),
    }

    queueFile := filepath.Join(a.config.DataDir, "queue", item.ID+".json")
    file, err := os.Create(queueFile)
    if err != nil {
        return err
    }
    defer file.Close()

    return json.NewEncoder(file).Encode(item)
}

func (a *Agent) processQueue() {
    ticker := time.NewTicker(5 * time.Minute)
    defer ticker.Stop()

    for {
        select {
        case <-ticker.C:
            a.retryQueuedItems()
        case <-a.ctx.Done():
            return
        }
    }
}
```

## 📊 Observabilidade e Telemetria

### 7.1. Logging Estruturado
```go
// logging.go
package main

import (
    "context"
    "log/slog"
    "os"

    "go.opentelemetry.io/otel/trace"
)

func (a *Agent) setupLogging() {
    handler := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
        Level: slog.LevelInfo,
    })

    logger := slog.New(handler)
    a.logger = logger
}

func (a *Agent) logWithTrace(ctx context.Context, level slog.Level, msg string, args ...any) {
    span := trace.SpanFromContext(ctx)
    if span.IsRecording() {
        args = append(args,
            "trace_id", span.SpanContext().TraceID().String(),
            "span_id", span.SpanContext().SpanID().String())
    }

    a.logger.Log(ctx, level, msg, args...)
}
```

### 7.2. Métricas Locais
```go
// metrics.go
package main

import (
    "encoding/json"
    "os"
    "sync"
    "time"
)

type Metrics struct {
    mu                sync.RWMutex
    FilesProcessed    int64     `json:"files_processed"`
    FilesErrored      int64     `json:"files_errored"`
    APICallsSuccess   int64     `json:"api_calls_success"`
    APICallsError     int64     `json:"api_calls_error"`
    LastActivity      time.Time `json:"last_activity"`
    UptimeStart       time.Time `json:"uptime_start"`
}

func (m *Metrics) IncrementFilesProcessed() {
    m.mu.Lock()
    defer m.mu.Unlock()
    m.FilesProcessed++
    m.LastActivity = time.Now()
}

func (m *Metrics) SaveToFile(path string) error {
    m.mu.RLock()
    defer m.mu.RUnlock()

    file, err := os.Create(path)
    if err != nil {
        return err
    }
    defer file.Close()

    return json.NewEncoder(file).Encode(m)
}
```

## 🔧 CI/CD e Deployment

### 8.1. Pipeline de Build
```yaml
# .github/workflows/local-agent.yml
name: Local Agent Build

on:
  push:
    paths: ['local-agent/**']
    tags: ['v*']

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v4
        with:
          go-version: '1.22'

      - name: Run Tests
        run: |
          cd local-agent
          go test -v -race -coverprofile=coverage.out ./...
          go tool cover -html=coverage.out -o coverage.html

      - name: Upload Coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./local-agent/coverage.out

  build:
    needs: test
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v4
        with:
          go-version: '1.22'

      - name: Build Windows Binary
        run: |
          cd local-agent
          go build -ldflags="-s -w" -o gnre-agent.exe ./cmd/agent

      - name: Sign Binary
        uses: azure/trusted-signing-action@v0.3.16
        with:
          azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}
          azure-client-secret: ${{ secrets.AZURE_CLIENT_SECRET }}
          endpoint: ${{ secrets.CODE_SIGNING_ENDPOINT }}
          code-signing-account-name: ${{ secrets.CODE_SIGNING_ACCOUNT }}
          certificate-profile-name: ${{ secrets.CERTIFICATE_PROFILE }}
          files-folder: ./local-agent
          files-folder-filter: exe

      - name: Create Installer
        run: |
          makensis installer.nsi

      - name: Upload Artifacts
        uses: actions/upload-artifact@v3
        with:
          name: gnre-agent-installer
          path: |
            gnre-agent-setup.exe
            gnre-agent.exe
```

### 8.2. Instalador NSIS
```nsis
; installer.nsi
!define APPNAME "GNRE Agent"
!define COMPANYNAME "GNRE Solutions"
!define DESCRIPTION "Agente local para processamento de XMLs GNRE"
!define VERSIONMAJOR 1
!define VERSIONMINOR 0
!define VERSIONBUILD 0

Name "${APPNAME}"
OutFile "gnre-agent-setup.exe"
InstallDir "$PROGRAMFILES\${COMPANYNAME}\${APPNAME}"

Section "Install"
    SetOutPath $INSTDIR
    File "gnre-agent.exe"

    ; Criar serviço Windows
    ExecWait '"$INSTDIR\gnre-agent.exe" install-service'

    ; Criar atalho no menu iniciar
    CreateDirectory "$SMPROGRAMS\${COMPANYNAME}"
    CreateShortCut "$SMPROGRAMS\${COMPANYNAME}\${APPNAME}.lnk" "$INSTDIR\gnre-agent.exe"

    ; Registrar no Add/Remove Programs
    WriteRegStr HKLM "Software\Microsoft\Windows\CurrentVersion\Uninstall\${APPNAME}" "DisplayName" "${APPNAME}"
    WriteRegStr HKLM "Software\Microsoft\Windows\CurrentVersion\Uninstall\${APPNAME}" "UninstallString" "$INSTDIR\uninstall.exe"

    WriteUninstaller "$INSTDIR\uninstall.exe"
SectionEnd

Section "Uninstall"
    ExecWait '"$INSTDIR\gnre-agent.exe" remove-service'
    Delete "$INSTDIR\gnre-agent.exe"
    Delete "$INSTDIR\uninstall.exe"
    RMDir "$INSTDIR"
    DeleteRegKey HKLM "Software\Microsoft\Windows\CurrentVersion\Uninstall\${APPNAME}"
SectionEnd
```

## 🔐 Melhorias de Segurança Avançada

### 10.1. Sandboxing e Isolamento
```go
// sandbox.go - Isolamento de processos
package main

import (
    "os"
    "syscall"
    "unsafe"
)

// Configurar processo com privilégios mínimos
func (a *Agent) setupSandbox() error {
    // Remover privilégios desnecessários
    if err := syscall.Setgroups([]int{}); err != nil {
        return err
    }

    // Configurar AppContainer no Windows
    if runtime.GOOS == "windows" {
        return a.setupWindowsAppContainer()
    }

    return nil
}

func (a *Agent) setupWindowsAppContainer() error {
    // Implementar AppContainer para isolamento
    // Limitar acesso apenas à pasta monitorada
    return nil
}
```

### 10.2. Verificação de Integridade Contínua
```go
// integrity.go
func (a *Agent) verifyBinaryIntegrity() error {
    executable, err := os.Executable()
    if err != nil {
        return err
    }

    // Verificar hash do binário atual
    currentHash, err := calculateFileHash(executable)
    if err != nil {
        return err
    }

    // Comparar com hash conhecido
    expectedHash := a.config.ExpectedBinaryHash
    if currentHash != expectedHash {
        return fmt.Errorf("binary integrity check failed")
    }

    return nil
}
```

## 📱 Melhorias de UX e Usabilidade

### 11.1. Interface Gráfica Opcional
```go
// gui.go - Interface gráfica simples com Fyne
package main

import (
    "fyne.io/fyne/v2/app"
    "fyne.io/fyne/v2/widget"
    "fyne.io/fyne/v2/container"
)

func (a *Agent) showGUI() {
    myApp := app.New()
    myWindow := myApp.NewWindow("GNRE Agent")

    // Status atual
    statusLabel := widget.NewLabel("Status: Monitorando...")

    // Estatísticas
    statsLabel := widget.NewLabel("Arquivos processados: 0")

    // Botões de controle
    pauseBtn := widget.NewButton("Pausar", func() {
        a.pause()
        statusLabel.SetText("Status: Pausado")
    })

    resumeBtn := widget.NewButton("Retomar", func() {
        a.resume()
        statusLabel.SetText("Status: Monitorando...")
    })

    content := container.NewVBox(
        statusLabel,
        statsLabel,
        container.NewHBox(pauseBtn, resumeBtn),
    )

    myWindow.SetContent(content)
    myWindow.ShowAndRun()
}
```

### 11.2. Notificações do Sistema
```go
// notifications.go
import "github.com/gen2brain/beeep"

func (a *Agent) notifyUser(title, message string, level NotificationLevel) {
    var icon string
    switch level {
    case NotificationSuccess:
        icon = "assets/success.png"
    case NotificationError:
        icon = "assets/error.png"
    case NotificationWarning:
        icon = "assets/warning.png"
    }

    err := beeep.Notify(title, message, icon)
    if err != nil {
        a.logger.Error("Failed to send notification", "error", err)
    }
}

// Uso
func (a *Agent) onFileProcessed(filename string, success bool) {
    if success {
        a.notifyUser(
            "GNRE Agent",
            fmt.Sprintf("Arquivo %s processado com sucesso", filename),
            NotificationSuccess,
        )
    } else {
        a.notifyUser(
            "GNRE Agent",
            fmt.Sprintf("Erro ao processar %s", filename),
            NotificationError,
        )
    }
}
```

## 🔧 Melhorias Operacionais

### 12.1. Configuração Avançada
```toml
# config.toml - Configuração mais rica
[agent]
name = "GNRE-Agent-001"
version = "1.0.0"
update_check_interval = "24h"

[monitoring]
watch_paths = [
    "C:/XMLs/Entrada",
    "C:/XMLs/Backup"
]
file_patterns = ["*.xml", "*.XML"]
ignore_patterns = ["*_processed.xml", "temp_*"]
max_file_size = "50MB"
processing_timeout = "30s"

[api]
base_url = "https://api.gnre.com/v1"
timeout = "30s"
retry_attempts = 5
retry_base_delay = "1s"
retry_max_delay = "30s"

[security]
verify_ssl = true
certificate_pinning = true
expected_cert_fingerprint = "sha256:..."

[logging]
level = "info"
format = "json"
max_file_size = "100MB"
max_files = 10
send_to_api = true

[notifications]
enabled = true
on_success = false
on_error = true
on_warning = true
```

### 12.2. Health Check Endpoint Local
```go
// health_server.go
func (a *Agent) startHealthServer() {
    http.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
        status := a.getHealthStatus()
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(status)
    })

    http.HandleFunc("/metrics", func(w http.ResponseWriter, r *http.Request) {
        metrics := a.getMetrics()
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(metrics)
    })

    go http.ListenAndServe(":8080", nil)
}

type HealthStatus struct {
    Status      string    `json:"status"`
    Uptime      string    `json:"uptime"`
    LastActivity time.Time `json:"last_activity"`
    APIStatus   string    `json:"api_status"`
    QueueSize   int       `json:"queue_size"`
}
```

---

*Este documento é atualizado com cada release da aplicação local e revisado pela equipe de desenvolvimento.*
# AI Workspace — Dev Container

Um Dev Container para VS Code pronto para uso em desenvolvimento de IA e software em geral. Oferece um ambiente consistente e reproduzível com Python, Node.js, .NET e um conjunto de ferramentas — incluindo Claude Code e Ollama — rodando em um stack Docker com hardening de segurança.

## Serviços

O `docker-compose.yml` sobe três containers:

| Serviço | Imagem | Descrição |
|---|---|---|
| **devcontainer** | build local | Ambiente de desenvolvimento (VS Code se conecta aqui) |
| **ollama** | `ollama/ollama:latest` | Servidor de LLMs locais |
| **open-webui** | `ghcr.io/open-webui/open-webui:main` | Interface web para o Ollama (acesse em `localhost:3000`) |

## O que está incluído no devcontainer

| Camada | Detalhes |
|---|---|
| **Imagem base** | `mcr.microsoft.com/devcontainers/base:ubuntu-24.04` |
| **Python** | Python 3 (`python3`, `python3-dev`, `python3-venv`) + [`uv`](https://github.com/astral-sh/uv) |
| **Node.js** | Node 20 LTS (via NodeSource) + npm atualizado |
| **.NET** | .NET 10 SDK (via repositório oficial da Microsoft) |
| **Claude Code** | `@anthropic-ai/claude-code` instalado globalmente via npm |
| **Ollama CLI** | Binário `ollama` copiado da imagem oficial; aponta para o serviço `ollama` via `OLLAMA_HOST` |
| **Ferramentas de build** | `build-essential`, `make`, `git`, `git-lfs`, `jq`, `curl`, `wget`, `unzip` |
| **Usuário** | `vscode` (não-root) |

### Extensões VS Code

| Categoria | Extensões |
|---|---|
| Python | Python, Black formatter, Pylint, Pylance |
| JS / TS | ESLint, Prettier |
| Git | GitLens, Git Graph |
| Containers | Docker, DotEnv |
| Produtividade | Code Spell Checker, Path Intellisense |

### Portas encaminhadas

| Porta | Serviço |
|---|---|
| `3000` | Open WebUI |
| `8000` | Python / FastAPI |
| `8080` | HTTP alternativo |
| `11434` | Ollama |

## Pré-requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (ou Docker Engine no Linux)
- [VS Code](https://code.visualstudio.com/) com a [extensão Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

## Como começar

1. **Clone o repositório**

   ```bash
   git clone https://github.com/francisco-alexandre-alencar-ribeiro/ai-workspace.git
   cd ai-workspace
   ```

2. **Configure as variáveis de ambiente**

   ```bash
   cp .devcontainer/.env.example .devcontainer/.env
   ```

   Edite `.devcontainer/.env` conforme necessário. Os valores padrão já funcionam para a maioria dos casos.

3. **Crie a rede Docker compartilhada** (somente uma vez)

   ```bash
   docker network create shared-network
   ```

4. **Crie a pasta `workspace/`** (somente na primeira vez)

   ```bash
   mkdir workspace
   ```

5. **Abra no VS Code e reabra no container**

   ```
   Ctrl+Shift+P → Dev Containers: Reopen in Container
   ```

   O VS Code irá construir a imagem e iniciar os três containers automaticamente. A pasta `workspace/` do host é montada em `/workspace` dentro do container.

## Encaminhamento do agente SSH

O container encaminha o agente SSH do host para que operações como `git push` funcionem sem precisar incluir chaves privadas na imagem. Certifique-se de que o agente SSH está rodando no host antes de abrir o container:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519   # ou a sua chave de preferência
```

## Suporte a GPU (NVIDIA)

O serviço `ollama` já tem a configuração de GPU comentada no `docker-compose.yml`. Para ativá-la, basta descomentar o bloco `deploy.resources.reservations.devices` no serviço `ollama` e garantir que o [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) esteja instalado no host.

## Limites de recursos

Os limites do devcontainer são configuráveis via variáveis de ambiente no `.env` (valores padrão entre parênteses):

| Variável | Padrão | Descrição |
|---|---|---|
| `DEVCONTAINER_MEMORY_LIMIT` | `8g` | Limite de memória |
| `DEVCONTAINER_MEMORY_RESERVE` | `512m` | Reserva de memória |
| `DEVCONTAINER_CPU_LIMIT` | `4` | Limite de CPUs |
| `DEVCONTAINER_CPU_RESERVE` | `0.5` | Reserva de CPUs |

## Hardening de segurança

- Executa como usuário não-root `vscode`
- Flag `no-new-privileges` previne escalada de privilégios via binários setuid
- Todas as capabilities Linux são removidas; somente `CHOWN`, `SETUID`, `SETGID` e `DAC_OVERRIDE` são adicionadas de volta

## Volumes persistentes

| Volume | Conteúdo |
|---|---|
| `vscode` | Home do usuário `vscode` (histórico de shell, cache do `uv`) |
| `ollama-models` | Modelos baixados pelo Ollama |
| `open-webui-data` | Dados e configurações do Open WebUI |

## Estrutura do projeto

```
.
├── .devcontainer/
│   ├── devcontainer.json   # Configuração do Dev Container para o VS Code
│   ├── Dockerfile          # Definição da imagem do container
│   ├── docker-compose.yml  # Compose com serviços, volumes, rede e limites de recursos
│   └── .env.example        # Variáveis de ambiente (copie para .env)
└── workspace/              # Seus arquivos de trabalho (montado no container)
```

## Personalização

- **Adicionar pacotes do sistema**: edite o bloco `RUN apt-get install` no `.devcontainer/Dockerfile`.
- **Adicionar extensões VS Code**: inclua os IDs das extensões no array `extensions` do `.devcontainer/devcontainer.json`.
- **Expor mais portas**: adicione entradas em `forwardPorts` e `portsAttributes` no `devcontainer.json`.
- **Alterar limites de recursos**: ajuste as variáveis `DEVCONTAINER_*` no `.devcontainer/.env`.
- **Trocar versão do Node.js**: defina `NODE_MAJOR` no `.env` antes de reconstruir a imagem.

## Licença

MIT

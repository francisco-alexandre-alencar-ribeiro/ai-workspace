# AI Workspace — Dev Container

Um Dev Container para VS Code pronto para uso em desenvolvimento de IA e software em geral. Oferece um ambiente consistente e reproduzível com Python, Node.js, .NET e um conjunto de ferramentas — incluindo Claude Code, Continue IDE (AI assistant), e o CLI do Ollama — rodando em Docker com hardening de segurança.

## Arquitetura

O `docker-compose.yml` sobe **um container**:

| Serviço | Imagem | Descrição |
|---|---|---|
| **devcontainer** | build local | Ambiente de desenvolvimento (VS Code se conecta aqui) |

O devcontainer ingressa na rede externa `shared-network`, onde serviços como **Ollama** e **Open WebUI** são esperados. Esses serviços devem ser iniciados separadamente (por exemplo, em outro compose stack) e ficam acessíveis pelo hostname `ollama` na mesma rede Docker.

## O que está incluído no devcontainer

| Camada | Detalhes |
|---|---|
| **Imagem base** | `mcr.microsoft.com/devcontainers/base:ubuntu-24.04` |
| **Python** | Python 3.12 (`python3`, `python3-dev`, `python3-pip`, `python3-venv`) + [`uv`](https://github.com/astral-sh/uv) + symlink `python` → `python3` |
| **Node.js** | Node 20 LTS (via NodeSource) + npm atualizado |
| **.NET** | .NET 10 SDK (via repositório oficial da Microsoft) |
| **Claude Code** | `@anthropic-ai/claude-code` instalado globalmente via npm |
| **Ollama CLI** | Binário `ollama` copiado da imagem oficial; aponta para o serviço externo via `OLLAMA_HOST` |
| **Ferramentas de build** | `build-essential`, `make`, `git`, `git-lfs`, `jq`, `curl`, `wget`, `unzip` |
| **Usuário** | `vscode` (não-root, sem privilégios de sudo) |

### Extensões VS Code

| Categoria | Extensões |
|---|---|
| **AI / IDE** | **Continue** (AI coding assistant), **Claude Code** (CLI integrado) |
| Python | Python, Black formatter, Pylint, Pylance |
| JS / TS | ESLint, Prettier |
| Git | GitLens, Git Graph |
| Containers | Docker, DotEnv |
| Produtividade | Code Spell Checker, Path Intellisense |

> **Continue**: IDE assistant que trabalha com modelos locais (Ollama) ou cloud. Oferece autocomplete, chat, e refatoração com IA.  
> **Claude Code**: CLI da Anthropic para usar Claude diretamente no terminal.

### Portas encaminhadas

| Porta | Label | Uso típico |
|---|---|---|
| `3000` | Web App | Open WebUI ou outra aplicação web |
| `8000` | Python API | Python / FastAPI |
| `8080` | HTTP Alt | HTTP alternativo |

> A porta `11434` (Ollama API) não é encaminhada pelo devcontainer porque o Ollama roda em container separado; acesse-o pelo hostname `ollama` dentro da rede Docker.

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

4. **Inicie os serviços externos na `shared-network`** (Ollama, Open WebUI, etc.)

   Suba qualquer compose stack que forneça esses serviços na rede `shared-network`. O devcontainer os encontrará automaticamente pelo hostname.

5. **Abra no VS Code e reabra no container**

   ```
   Ctrl+Shift+P → Dev Containers: Reopen in Container
   ```

   O VS Code irá construir a imagem e iniciar o container automaticamente. A pasta `workspace/` do host é montada em `/workspace` dentro do container.

## Usando AI no desenvolvimento

### Continue (IDE AI Assistant)
Continue é um IDE assistant que se integra ao VS Code e oferece:
- **Autocomplete inteligente** com sugestões em tempo real
- **Chat com IA** dentro do editor (`Ctrl+Shift+I`)
- **Refatoração e geração de código** assistida
- **Suporte a modelos locais** (Ollama) ou cloud

Para começar, abra o Command Palette (`Ctrl+Shift+P`) e procure por "Continue".

### Claude Code (CLI)
Claude Code é a CLI oficial da Anthropic:

```bash
claude-code <arquivo>              # Editar arquivo com Claude
claude-code --help                 # Ver todas as opções
```

Dentro do container, use normalmente — o CLI já está instalado globalmente.

## Encaminhamento do agente SSH

O container encaminha o agente SSH do host para que operações como `git push` funcionem sem precisar incluir chaves privadas na imagem. Certifique-se de que o agente SSH está rodando no host antes de abrir o container:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519   # ou a sua chave de preferência
```

## Suporte a GPU (NVIDIA)

O suporte a GPU é configurado no compose stack do serviço **Ollama** (externo a este repositório). Para ativá-lo, adicione o bloco `deploy.resources.reservations.devices` no serviço `ollama` desse stack e garanta que o [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) esteja instalado no host.

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

> Volumes dos serviços externos (modelos do Ollama, dados do Open WebUI) são gerenciados pelos seus respectivos compose stacks.

## Estrutura do projeto

```
.
├── .devcontainer/
│   ├── devcontainer.json   # Configuração do Dev Container para o VS Code
│   ├── Dockerfile          # Definição da imagem do container
│   ├── docker-compose.yml  # Compose com o serviço, volume, rede e limites de recursos
│   └── .env.example        # Variáveis de ambiente (copie para .env)
├── .envs-ps1/              # Scripts de ambiente para sessões PowerShell (conteúdo ignorado pelo git)
└── workspace/              # Seus arquivos de trabalho (montado em /workspace no container; conteúdo ignorado pelo git)
```

## Personalização

- **Adicionar pacotes do sistema**: edite o bloco `RUN apt-get install` no `.devcontainer/Dockerfile`.
- **Adicionar extensões VS Code**: inclua os IDs das extensões no array `extensions` do `.devcontainer/devcontainer.json`.
- **Configurar Continue**: abra `~/.continue/config.json` dentro do container para customizar modelos, provedores e comportamentos.
- **Expor mais portas**: adicione entradas em `forwardPorts` e `portsAttributes` no `devcontainer.json`.
- **Alterar limites de recursos**: ajuste as variáveis `DEVCONTAINER_*` no `.devcontainer/.env`.
- **Trocar versão do Node.js**: defina `NODE_MAJOR` no `.env` antes de reconstruir a imagem.

## Conectar Continue com Ollama

Dentro do container, crie ou edite `~/.continue/config.json`:

```json
{
  "models": [
    {
      "title": "Ollama Local",
      "provider": "ollama",
      "model": "mistral",
      "apiBase": "http://ollama:11434"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Ollama Autocomplete",
    "provider": "ollama",
    "model": "mistral",
    "apiBase": "http://ollama:11434"
  }
}
```

Substitua `mistral` pelo modelo que você tem instalado no Ollama (ex: `llama2`, `neural-chat`, etc.).

## Troubleshooting

| Problema | Solução |
|---|---|
| Container não inicia | Verifique se Docker está rodando: `docker ps` |
| Continue não conecta ao Ollama | Verifique se Ollama está rodando: `docker ps \| grep ollama` e se `shared-network` existe: `docker network ls` |
| `python` não encontrado | O symlink `python → python3` é criado durante a build. Reconstrua a imagem: `Ctrl+Shift+P → Dev Containers: Rebuild Container` |
| Permissões de escrita em `/workspace` | O volume é montado com a opção `:z` para SELinux. Se ainda tiver problemas, verifique o usuário: `whoami` (deve ser `vscode`) |

## Recursos úteis

- [VS Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers)
- [Continue.dev Documentation](https://continue.dev/)
- [Claude API & Anthropic SDK](https://docs.anthropic.com/)
- [Ollama Documentation](https://github.com/ollama/ollama)

## Licença

MIT

# AI Workspace — Dev Container

Um Dev Container para VS Code pronto para uso em desenvolvimento de IA e software em geral. Oferece um ambiente consistente e reproduzível com Python, Node.js e um conjunto de extensões VS Code — incluindo o Claude Code — rodando em um container Docker com hardening de segurança.

## O que está incluído

| Camada | Detalhes |
|---|---|
| **Imagem base** | `mcr.microsoft.com/devcontainers/base:ubuntu-24.04` |
| **Python** | Python 3 + [`uv`](https://github.com/astral-sh/uv) (gerenciador de pacotes rápido) |
| **Node.js** | Node 20 LTS (via NodeSource) |
| **Ferramentas de build** | `build-essential`, `make`, `git`, `git-lfs`, `jq`, `curl`, `wget` |
| **Usuário** | `vscode` (não-root) |

### Extensões VS Code

| Categoria | Extensões |
|---|---|
| IA | Claude Code (`Anthropic.claude-code`), Continue (`Continue.continue`) |
| Python | Python, Black formatter, Pylint, Pylance |
| JS / TS | ESLint, Prettier |
| Git | GitLens, Git Graph |
| Containers | Docker, DotEnv |
| Produtividade | Code Spell Checker, Path Intellisense |

### Portas encaminhadas

| Porta | Serviço |
|---|---|
| `3000` | Aplicação web genérica |
| `8000` | Python / FastAPI |
| `8080` | HTTP alternativo |
| `11434` | [Ollama](https://ollama.com) (LLM local) |

## Pré-requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (ou Docker Engine no Linux)
- [VS Code](https://code.visualstudio.com/) com a [extensão Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

## Como começar

1. **Clone o repositório**

   ```bash
   git clone <url-do-seu-repositorio>
   cd ai-workspace
   ```

2. **Crie a rede Docker compartilhada** (somente uma vez)

   ```bash
   docker network create shared-network
   ```

3. **Abra no VS Code e reabra no container**

   ```
   Ctrl+Shift+P → Dev Containers: Reopen in Container
   ```

   O VS Code irá construir a imagem e iniciar o container automaticamente. A pasta `workspace/` do seu host é montada em `/workspace` dentro do container.

## Encaminhamento do agente SSH

O container encaminha o agente SSH do host para que operações como `git push` funcionem sem precisar incluir chaves privadas na imagem. Certifique-se de que o agente SSH está rodando no host antes de abrir o container:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519   # ou a sua chave de preferência
```

## Limites de recursos

Limites padrão definidos no `docker-compose.yml` (ajuste conforme a sua máquina):

| Recurso | Limite | Reserva |
|---|---|---|
| Memória | 8 GB | 512 MB |
| CPU | 4 núcleos | 0,5 núcleos |

## Hardening de segurança

- Executa como usuário não-root `vscode`
- Flag `no-new-privileges` previne escalada de privilégios via binários setuid
- Todas as capabilities Linux são removidas; somente `CHOWN`, `SETUID`, `SETGID` e `DAC_OVERRIDE` são adicionadas de volta

## Estrutura do projeto

```
.
├── .devcontainer/
│   ├── devcontainer.json   # Configuração do Dev Container para o VS Code
│   ├── Dockerfile          # Definição da imagem do container
│   └── docker-compose.yml  # Compose com volumes, rede e limites de recursos
└── workspace/              # Seus arquivos de trabalho (montado no container)
```

## Personalização

- **Adicionar pacotes**: edite o bloco `RUN apt-get install` no `.devcontainer/Dockerfile`.
- **Adicionar extensões VS Code**: inclua os IDs das extensões no array `extensions` do `.devcontainer/devcontainer.json`.
- **Expor mais portas**: adicione entradas em `forwardPorts` e `portsAttributes` no `devcontainer.json`.
- **Alterar limites de recursos**: edite o bloco `deploy.resources` no `docker-compose.yml`.

## Licença

MIT

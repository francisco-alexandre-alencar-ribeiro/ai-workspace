# AI Workspace — Dev Container

Um Dev Container para VS Code pronto para uso em desenvolvimento de IA e software em geral. Oferece um ambiente consistente e reproduzível com Python, Node.js, .NET, Java e um conjunto de ferramentas — incluindo Claude Code e Specify CLI (SDD) — rodando em Docker com hardening de segurança.

## Arquitetura

O `docker-compose.yml` sobe **um container**:

| Serviço          | Imagem      | Descrição                                             |
| ---------------- | ----------- | ----------------------------------------------------- |
| **devcontainer** | build local | Ambiente de desenvolvimento (VS Code se conecta aqui) |

O devcontainer ingressa na rede externa `shared-network`, onde serviços externos como o **Open WebUI** são esperados. Esses serviços devem ser iniciados separadamente (por exemplo, em outro compose stack) e ficam acessíveis pelo hostname do serviço na mesma rede Docker.

## O que está incluído no devcontainer

| Camada                   | Detalhes                                                                                                                                       |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Imagem base**          | `mcr.microsoft.com/devcontainers/base:ubuntu-24.04`                                                                                            |
| **Python**               | Python 3.12 (`python3`, `python3-dev`, `python3-pip`, `python3-venv`) + [`uv`](https://github.com/astral-sh/uv) + symlink `python` → `python3` |
| **Node.js**              | Node 22 LTS (via NodeSource)                                                                                                                   |
| **.NET**                 | .NET 10 SDK (via repositório oficial da Microsoft)                                                                                             |
| **Java**                 | Eclipse Temurin JDK 21 em `/opt/java/current` (`JAVA_HOME` já configurado e no `PATH`), com download verificado por SHA-256                    |
| **Claude Code**          | `@anthropic-ai/claude-code` instalado globalmente via npm                                                                                      |
| **Specify CLI**          | `specify` ([GitHub Spec Kit](https://github.com/github/spec-kit)) instalado globalmente via `uv tool`, para Spec-Driven Development (SDD)         |
| **Ferramentas de build** | `build-essential`, `make`, `git`, `git-lfs`, `jq`, `curl`, `wget`, `unzip`                                                                     |
| **Usuário**              | `vscode` (não-root, sem privilégios de sudo)                                                                                                   |

### Extensões VS Code

| Categoria     | Extensões                                                           |
| ------------- | ------------------------------------------------------------------- |
| **AI / IDE**  | **Claude Code** (CLI integrado)                                     |
| Python        | Python, Black formatter, Pylint, Pylance                            |
| Java          | Extension Pack for Java (`vscjava.vscode-java-pack`)                |
| JS / TS       | ESLint, Prettier                                                    |
| Git           | GitLens, Git Graph                                                  |
| Containers    | Docker, DotEnv                                                      |
| Produtividade | Code Spell Checker, Path Intellisense                               |

> **Claude Code**: CLI da Anthropic para usar Claude diretamente no terminal.

### Portas encaminhadas

| Porta  | Label      | Uso típico                        |
| ------ | ---------- | --------------------------------- |
| `3000` | Web App    | Open WebUI ou outra aplicação web |
| `8000` | Python API | Python / FastAPI                  |
| `8080` | HTTP Alt   | HTTP alternativo                  |

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

4. **Inicie os serviços externos na `shared-network`** (Open WebUI, etc.)

   Suba qualquer compose stack que forneça esses serviços na rede `shared-network`. O devcontainer os encontrará automaticamente pelo hostname.

5. **Abra no VS Code e reabra no container**

   ```
   Ctrl+Shift+P → Dev Containers: Reopen in Container
   ```

   O VS Code irá construir a imagem e iniciar o container automaticamente. A pasta `workspace/` do host é montada em `/workspace` dentro do container.

## Usando AI no desenvolvimento

### Claude Code (CLI)

Claude Code é a CLI oficial da Anthropic:

```bash
claude                             # Iniciar sessão interativa no diretório atual
claude "explique este projeto"     # Rodar um prompt direto
claude --help                      # Ver todas as opções
```

Dentro do container, use normalmente — o CLI já está instalado globalmente.

## Encaminhamento do agente SSH

O container encaminha o agente SSH do host para que operações como `git push` funcionem sem precisar incluir chaves privadas na imagem. Certifique-se de que o agente SSH está rodando no host antes de abrir o container:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519   # ou a sua chave de preferência
```

## Limites de recursos

Os limites do devcontainer são configuráveis via variáveis de ambiente no `.env` (valores padrão entre parênteses):

| Variável                      | Padrão | Descrição          |
| ----------------------------- | ------ | ------------------ |
| `DEVCONTAINER_MEMORY_LIMIT`   | `8g`   | Limite de memória  |
| `DEVCONTAINER_MEMORY_RESERVE` | `512m` | Reserva de memória |
| `DEVCONTAINER_CPU_LIMIT`      | `4`    | Limite de CPUs     |
| `DEVCONTAINER_CPU_RESERVE`    | `0.5`  | Reserva de CPUs    |

## Hardening de segurança

- Executa como usuário não-root `vscode`
- Flag `no-new-privileges` previne escalada de privilégios via binários setuid
- Todas as capabilities Linux são removidas; somente `CHOWN`, `SETUID`, `SETGID` e `DAC_OVERRIDE` são adicionadas de volta

## Volumes persistentes

| Volume   | Conteúdo                                                     |
| -------- | ------------------------------------------------------------ |
| `vscode` | Home do usuário `vscode` (histórico de shell, cache do `uv`) |

> Volumes dos serviços externos (dados do Open WebUI) são gerenciados pelos seus respectivos compose stacks.

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
- **Expor mais portas**: adicione entradas em `forwardPorts` e `portsAttributes` no `devcontainer.json`.
- **Alterar limites de recursos**: ajuste as variáveis `DEVCONTAINER_*` no `.devcontainer/.env`.
- **Trocar versão do Node.js**: defina `NODE_MAJOR` no `.env` antes de reconstruir a imagem.
- **Trocar versão do Specify CLI**: defina `SPEC_KIT_VERSION` (tag do [spec-kit](https://github.com/github/spec-kit/tags)) no `.env` antes de reconstruir a imagem.
- **Trocar versão do Java**: ajuste os build args `JAVA_VERSION`, `JAVA_SHA256_X64` e `JAVA_SHA256_AARCH64` no `.devcontainer/Dockerfile`. Os checksums precisam corresponder aos publicados na [release do Temurin 21](https://github.com/adoptium/temurin21-binaries/releases) — a build falha se não baterem.

## Troubleshooting

| Problema                              | Solução                                                                                                                        |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Container não inicia                  | Verifique se Docker está rodando: `docker ps`                                                                                  |
| `python` não encontrado               | O symlink `python → python3` é criado durante a build. Reconstrua a imagem: `Ctrl+Shift+P → Dev Containers: Rebuild Container` |
| Permissões de escrita em `/workspace` | O volume é montado com a opção `:z` para SELinux. Se ainda tiver problemas, verifique o usuário: `whoami` (deve ser `vscode`)  |

## Recursos úteis

- [VS Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers)
- [Claude API & Anthropic SDK](https://docs.anthropic.com/)

## Licença

MIT

FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

ARG NODE_MAJOR=20

# System dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    curl \
    git \
    git-lfs \
    gnupg \
    jq \
    lsb-release \
    make \
    python3 \
    python3-dev \
    python3-pip \
    python3-venv \
    unzip \
    wget \
    apt-transport-https \
    && git lfs install --system \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Microsoft repository (.NET)
RUN wget https://packages.microsoft.com/config/ubuntu/24.04/packages-microsoft-prod.deb \
    && dpkg -i packages-microsoft-prod.deb \
    && rm packages-microsoft-prod.deb

# .NET 10 SDK
RUN apt-get update \
    && apt-get install -y --no-install-recommends dotnet-sdk-10.0 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Node.js LTS
RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_MAJOR}.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && npm install -g npm@latest \
    && npm install -g @anthropic-ai/claude-code

# uv — fast Python package manager (installed system-wide)
RUN curl -LsSf https://astral.sh/uv/install.sh | UV_INSTALL_DIR=/usr/local/bin sh

# Ollama CLI — binary copied from the official image so `ollama` is available
# in the terminal. OLLAMA_HOST (set in docker-compose) points it at the
# ollama service container instead of trying to start a local server.
COPY --from=ollama/ollama:latest /bin/ollama /usr/local/bin/ollama

# Ensure workspace dir belongs to vscode user
RUN mkdir -p /workspace && chown vscode:vscode /workspace

# Create python -> python3 symlink for convenience (while still root)
RUN ln -s /usr/bin/python3 /usr/bin/python

# Drop to non-root for everything from here on
USER vscode
WORKDIR /workspace

# Shell convenience
RUN echo 'export PATH="$HOME/.local/bin:$PATH"' >> /home/vscode/.bashrc \
    && echo 'export PYTHONDONTWRITEBYTECODE=1' >> /home/vscode/.bashrc \
    && echo 'export PYTHONUNBUFFERED=1' >> /home/vscode/.bashrc
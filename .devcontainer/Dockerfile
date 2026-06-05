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
    && git lfs install --system \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Node.js LTS
RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_MAJOR}.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && npm install -g npm@latest

# uv — fast Python package manager (installed system-wide)
RUN curl -LsSf https://astral.sh/uv/install.sh | UV_INSTALL_DIR=/usr/local/bin sh

# Ensure workspace dir belongs to vscode user
RUN mkdir -p /workspace && chown vscode:vscode /workspace

# Drop to non-root for everything from here on
USER vscode
WORKDIR /workspace

# Shell convenience: activate uv-managed venv automatically
RUN echo 'export PATH="$HOME/.local/bin:$PATH"' >> /home/vscode/.bashrc \
    && echo 'export PYTHONDONTWRITEBYTECODE=1' >> /home/vscode/.bashrc \
    && echo 'export PYTHONUNBUFFERED=1' >> /home/vscode/.bashrc

FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

ARG NODE_MAJOR=22
ARG JAVA_VERSION=17.0.18+8
ARG JAVA_SHA256_X64=0c94cbb54325c40dcf026143eb621562017db5525727f2d9131a11250f72c450
ARG JAVA_SHA256_AARCH64=592a6702b3a07a0e0b82cb38aaab149bfce1b0c24d6b57ddb410bd9009333095

# --------------------------------------------------------------------
# Base system
# --------------------------------------------------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
    apt-transport-https \
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

# --------------------------------------------------------------------
# Microsoft Repository (.NET)
# --------------------------------------------------------------------
RUN wget https://packages.microsoft.com/config/ubuntu/24.04/packages-microsoft-prod.deb \
 && dpkg -i packages-microsoft-prod.deb \
 && rm packages-microsoft-prod.deb

# --------------------------------------------------------------------
# .NET 10 SDK
# --------------------------------------------------------------------
RUN apt-get update \
 && apt-get install -y --no-install-recommends dotnet-sdk-10.0 \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# --------------------------------------------------------------------
# Node.js 22 LTS + Claude Code
# --------------------------------------------------------------------
RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_MAJOR}.x | bash - \
 && apt-get update \
 && apt-get install -y --no-install-recommends nodejs \
 && npm install -g @anthropic-ai/claude-code \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# --------------------------------------------------------------------
# uv (Python package manager)
# --------------------------------------------------------------------
RUN curl -LsSf https://astral.sh/uv/install.sh | \
    UV_INSTALL_DIR=/usr/local/bin sh

# --------------------------------------------------------------------
# Eclipse Temurin JDK 17
# --------------------------------------------------------------------
RUN set -eux; \
    case "$(dpkg --print-architecture)" in \
      amd64)  JDK_ARCH=x64;     JDK_SHA="${JAVA_SHA256_X64}" ;; \
      arm64)  JDK_ARCH=aarch64; JDK_SHA="${JAVA_SHA256_AARCH64}" ;; \
      *) echo "Unsupported architecture: $(dpkg --print-architecture)" >&2; exit 1 ;; \
    esac; \
    JDK_TAG="$(echo "${JAVA_VERSION}" | sed 's/+/%2B/')"; \
    JDK_FILE="$(echo "${JAVA_VERSION}" | tr '+' '_')"; \
    curl -fsSL -o /tmp/jdk.tar.gz \
      "https://github.com/adoptium/temurin17-binaries/releases/download/jdk-${JDK_TAG}/OpenJDK17U-jdk_${JDK_ARCH}_linux_hotspot_${JDK_FILE}.tar.gz"; \
    echo "${JDK_SHA}  /tmp/jdk.tar.gz" | sha256sum -c -; \
    mkdir -p /opt/java; \
    tar -xzf /tmp/jdk.tar.gz -C /opt/java; \
    mv "/opt/java/jdk-${JAVA_VERSION}" /opt/java/current; \
    rm /tmp/jdk.tar.gz

ENV JAVA_HOME=/opt/java/current
ENV PATH="${JAVA_HOME}/bin:${PATH}"

# --------------------------------------------------------------------
# Ollama CLI
# --------------------------------------------------------------------
COPY --from=ollama/ollama:latest /bin/ollama /usr/local/bin/ollama

# --------------------------------------------------------------------
# Workspace
# --------------------------------------------------------------------
RUN mkdir -p /workspace \
 && chown vscode:vscode /workspace \
 && ln -sf /usr/bin/python3 /usr/bin/python

# --------------------------------------------------------------------
# Non-root user
# --------------------------------------------------------------------
USER vscode
WORKDIR /workspace

# --------------------------------------------------------------------
# Shell configuration
# --------------------------------------------------------------------
RUN echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc \
 && echo 'export PYTHONDONTWRITEBYTECODE=1' >> ~/.bashrc \
 && echo 'export PYTHONUNBUFFERED=1' >> ~/.bashrc

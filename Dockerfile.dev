FROM ghcr.io/astral-sh/uv:python3.12-bookworm

ARG USER_ID=1000
ARG USER_NAME=devuser
ARG GROUP_ID=1000

# Avoid interactive prompts during install
ENV DEBIAN_FRONTEND=noninteractive

ENV RUNNING_IN_DOCKER=true
ENV NODE_PATH=/usr/lib/node_modules

EXPOSE 8080

RUN groupadd --gid $GROUP_ID $USER_NAME \
    && useradd -m -u $USER_ID -g $GROUP_ID $USER_NAME -s /bin/bash

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    gnupg ca-certificates \
    curl \
    git \
    bc \
    vim

# Install Python project dependencies
COPY pyproject.toml uv.lock ./
RUN uv export --format requirements.txt > requirements.txt \
    && uv pip install --system -r requirements.txt

# Install Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Antora and its dependencies
RUN npm i -g \
    http-serve@1.0.1 \
    @antora/cli@3.1.9 \
    @antora/lunr-extension@^1.0.0-alpha.8 \
    @antora/site-generator@3.1.9 \
    @mermaid-js/mermaid-cli@^11.4.2 \
    asciidoctor-kroki@^0.18.1 \
    @djencks/asciidoctor-mathjax@^0.0.9

# Install just
RUN curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to /usr/local/bin --tag 1.38.0

# Install shell completions for just
RUN just --completions bash >> /usr/share/bash-completion/completions/just \
    && echo 'source /usr/share/bash-completion/completions/just' >> /etc/bash.bashrc

# GitHub client
RUN (type -p wget >/dev/null || (apt update && apt-get install wget -y)) \
    && mkdir -p -m 755 /etc/apt/keyrings \
    && out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
    && cat $out | tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
    && chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
    && apt update \
    && apt install gh -y

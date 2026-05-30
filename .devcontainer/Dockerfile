FROM debian:trixie-slim AS devcontainer-debian

ARG TZ
ENV TZ="$TZ"
ENV DEBIAN_FRONTEND=noninteractive

# Define a non-root user for Visual Studio Code
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Ensure that we always use 'vscode' username
RUN apt-get update && apt-get install -y --no-install-recommends locales sudo zsh \
    && locale-gen en_US.UTF-8 \
    && EXISTING_USER=$(getent passwd "$USER_UID" | cut -d: -f1) \
    && if [ -n "$EXISTING_USER" ]; then \
           if [ "$EXISTING_USER" != "$USERNAME" ]; then \
               usermod  -l "$USERNAME" "$EXISTING_USER"; \
               usermod  -d "/home/$USERNAME" -m "$USERNAME"; \
               groupmod -n "$USERNAME" "$EXISTING_USER"; \
           fi; \
           usermod -s /usr/bin/zsh "$USERNAME"; \
       else \
           groupadd --gid "$USER_GID" "$USERNAME"; \
           useradd --uid "$USER_UID" --gid "$USER_GID" -m -s /usr/bin/zsh "$USERNAME"; \
       fi

# Allow passwordless sudo in the container
RUN echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/user-$USERNAME-nopasswd \
    && chmod 0440 /etc/sudoers.d/user-$USERNAME-nopasswd

# Install useful packages
RUN apt-get install -y --no-install-recommends \
    aggregate \
    bash \
    bash-completion \
    bubblewrap \
    build-essential \
    ca-certificates \
    curl \
    diffutils \
    dnsutils \
    fd-find \
    file \
    fzf \
    gawk \
    gh \
    git \
    git-delta \
    gnupg2 \
    iproute2 \
    ipset \
    isort \
    jq \
    less \
    librsvg2-bin \
    lsof \
    make \
    mc \
    man-db \
    procps \
    ripgrep \
    sed \
    shellcheck \
    socat \
    strace \
    tree \
    unzip \
    vim \
    wget \
    yamllint \
    yq \
    zip \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install opengrep (semgrep OSS fork). Pin version + SHA256 — the image build
# fails if the binary ever changes upstream. Single static binary, no runtime deps.
ARG OPENGREP_VERSION=1.22.0
ARG OPENGREP_SHA256=45bcd58440e397ed52c50e953ccf5948909ea77087c9186fc7d277216f62e319
RUN curl -fsSL "https://github.com/opengrep/opengrep/releases/download/v${OPENGREP_VERSION}/opengrep_manylinux_x86" -o /usr/local/bin/opengrep \
    && echo "${OPENGREP_SHA256}  /usr/local/bin/opengrep" | sha256sum -c - \
    && chmod +x /usr/local/bin/opengrep

ARG ASTGREP_VERSION=0.43.0
ARG ASTGREP_SHA256=a26253a9c821d935f7e383e40f0de7c2ca62a4121de1f73a6d81ec32eae631e0
RUN curl -fsSL "https://github.com/ast-grep/ast-grep/releases/download/${ASTGREP_VERSION}/app-x86_64-unknown-linux-gnu.zip" -o /tmp/ast.zip \
  && echo "${ASTGREP_SHA256}  /tmp/ast.zip" | sha256sum -c - \
  && unzip -p /tmp/ast.zip ast-grep > /usr/local/bin/ast-grep \
  && chmod +x /usr/local/bin/ast-grep \
  && rm /tmp/ast.zip

# Set environment variables to avoid noisy warnings
ENV DEVCONTAINER=true
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

# Fix shell for NVM installation and set paths
RUN ln -sf bash /usr/bin/sh
RUN echo 'export PATH="$HOME/.local/bin:$PATH"' | tee -a /home/$USERNAME/.bashrc /home/$USERNAME/.zshrc

# Create workspace and config directories and set permissions
RUN mkdir -p /workspaces
RUN [ -n "$USERNAME" ] && mkdir -p "/home/$USERNAME"/{.claude,.codex,.config,.docker}
RUN [ -n "$USERNAME" ] && chown -R $USERNAME:$USERNAME /workspaces "/home/$USERNAME"/{.claude,.codex,.config,.docker}

WORKDIR /workspaces

# Run as a non-root user
USER $USERNAME

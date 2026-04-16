FROM ubuntu:24.04 AS devcontainer-ubuntu
#FROM debian:stable-slim AS devcontainer-debian

ARG TZ
ENV TZ="$TZ"
ENV DEBIAN_FRONTEND=noninteractive

# Create non-root user
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Check if this user exists to satisfy both Ubuntu and Debian images
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
RUN echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Install useful packages on Ubuntu/Debian
RUN apt-get install -y --no-install-recommends \
    aggregate \
    bash \
    bash-completion \
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
    iptables \
    isort \
    jq \
    less \
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
    zip \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Set environment variables to avoid noisy warnings
ENV DEVCONTAINER=true
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

# Fix shell for NVM installation
RUN ln -sf bash /usr/bin/sh

# Create workspace and config directories and set permissions
RUN mkdir -p /workspaces /home/$USERNAME/.claude /home/$USERNAME/.config
RUN echo 'export PATH="$HOME/.local/bin:$PATH"' | tee -a /home/$USERNAME/.bashrc /home/$USERNAME/.zshrc
RUN chown -R $USERNAME:$USERNAME /workspaces /home/$USERNAME/.claude /home/$USERNAME/.config

WORKDIR /workspaces

# Run as a non-root user
USER $USERNAME

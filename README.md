# Devcontainer

My opionated minimalistic Debian or Ubuntu-based devcontainer setup with Go, Node.js, and Claude Code.

## Setup

### Git identity

Set these environment variables on your host before building the container:

```bash
export GIT_AUTHOR_NAME="Your Name"
export GIT_AUTHOR_EMAIL="you@example.com"
export GIT_COMMITTER_NAME="Your Name"
export GIT_COMMITTER_EMAIL="you@example.com"
```

Add them to your shell profile (`~/.bashrc`, `~/.zshrc`) to persist across sessions.

### GitHub CLI

This container uses HTTPS exclusively for git operations. SSH is not installed and SSH agent forwarding is disabled.

Authenticate with GitHub for HTTPS push/pull:

```bash
gh auth login
gh auth setup-git
```

The `gh` config is stored in a named volume and persists across container rebuilds.

### Claude Code

Claude Code is installed automatically via `postCreateCommand`. To authenticate:

```bash
claude login
```

The Claude config is stored in a named volume at `/home/vscode/.claude` and persists across container rebuilds.

## Security

### Unpinned images

The base image (`ubuntu:24.04`) and devcontainer features use mutable tags. To pin the base image to a specific digest:

```bash
docker pull ubuntu:24.04
docker inspect --format='{{index .RepoDigests 0}}' ubuntu:24.04
```

Then update the Dockerfile:

```dockerfile
FROM ubuntu:24.04@sha256:<digest>
```

### SSH disabled

SSH is not installed. SSH agent forwarding from the host is disabled via `remoteEnv` and `remote.SSH.enableAgentForwarding`. All git operations use HTTPS via `gh auth`.

### Claude Code

Claude Code runs with `bypassPermissions` mode and `allowDangerouslySkipPermissions` enabled. This means it can execute any command — including `sudo` — without user confirmation. Do not mount sensitive host directories or credentials into this container.
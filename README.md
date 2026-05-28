# PixivUtil2 Docker Build

Automated build system for [PixivUtil2](https://github.com/Nandaka/PixivUtil2) Docker images.

## Prerequisites

- [Task](https://taskfile.dev/)
- Docker with buildx support
- SSH access to remote host

## Setup

Copy `.env.example` to `.env` and configure:

```
APP_NAME=
REGISTRY=
APT_CACHER=
AMD64_BUILDER=
REMOTE_HOST=
```

## Usage

```bash
# Fetch source + build amd64 + push + deploy
task build

# Fetch source only
task prepare
```

## Project Structure

```
.
├── Taskfile.yaml      # Build automation
├── app/               # Files overlaid on upstream source
│   ├── Dockerfile
│   └── .dockerignore
└── build/             # Temporary build directory (auto-generated)
```

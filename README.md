# PixivUtil2 Docker Build

Automated build system for creating and deploying multi-architecture Docker images of [PixivUtil2](https://github.com/Nandaka/PixivUtil2).

## Overview

This project automates the process of:
1. Fetching the latest PixivUtil2 source code from GitHub
2. Building Docker images for multiple architectures (amd64/arm64)
3. Pushing images to a private registry
4. Deploying to remote hosts

## Prerequisites

- [Task](https://taskfile.dev/) - Task runner
- Docker with buildx support
- SSH access to remote host (for deployment)
- Private Docker registry

## Setup

1. Clone this repository
2. Copy `.env.example` to `.env` and configure:
   ```bash
   REGISTRY=your-registry.example.com
   APT_CACHER=your-apt-cacher.example.com
   AMD64_BUILDER=your-builder-name
   REMOTE_HOST=your-remote.example.com
   ```
3. Ensure Docker buildx builder is configured

## Usage

### Build and Deploy

```bash
# Full build: fetch source + build amd64 + push + deploy
task build

# Build arm64 image
task arm

# Prepare source only (without building)
task prepare
```

### Build Process

The build task performs the following steps:

1. **Prepare**: Downloads latest PixivUtil2 source from GitHub master branch
2. **Overwrite**: Copies custom files from `overwrite/` directory (Dockerfile, entrypoint.sh, .dockerignore)
3. **Build**: Creates Docker image using buildx for specified architecture
4. **Push**: Uploads image to private registry
5. **Deploy**: (amd64 only) SSH to remote host and pulls the new image

### Running the Container

The built image includes an entrypoint that allows passing arguments to PixivUtil2:

```bash
# Run with arguments
docker run ${REGISTRY}/pixivutil2:latest --download --user=username

# Run interactively
docker run -it ${REGISTRY}/pixivutil2:latest

# Mount data directory
docker run -v /path/to/data:/data ${REGISTRY}/pixivutil2:latest --download
```

## Architecture

- **Python-based**: Uses `python:slim` as base image
- **APT Caching**: Optional APT-Cacher NG support for faster builds
- **Multi-arch**: Supports both amd64 and arm64 platforms
- **Automated**: Complete CI/CD pipeline using Task

## Project Structure

```
.
├── Taskfile.yaml      # Build automation tasks
├── overwrite/         # Files to overlay on upstream PixivUtil2
│   ├── Dockerfile     # Container definition
│   ├── entrypoint.sh  # Entrypoint script for passing arguments
│   └── .dockerignore  # Docker build exclusions
├── .env              # Environment configuration
└── build/            # Temporary build directory (auto-generated)
```

## License

This build system is provided as-is. PixivUtil2 is licensed under its own terms - see the [upstream repository](https://github.com/Nandaka/PixivUtil2) for details.

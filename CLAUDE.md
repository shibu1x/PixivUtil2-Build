# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a build automation project for creating and deploying Docker images of PixivUtil2 (https://github.com/Nandaka/PixivUtil2). The project fetches the upstream source code, builds Docker images for multiple architectures, and pushes them to a private registry.

## Architecture

The build process follows this workflow:

1. **Source Preparation**: Downloads latest PixivUtil2 source from GitHub, extracts it to a temporary build directory
2. **Docker Build**: Builds multi-architecture images (amd64/arm64) using buildx
3. **Registry Push**: Pushes images to private registry and triggers remote host to pull the latest image

### Key Components

- **Taskfile.yaml**: Task automation using Task (taskfile.dev). Contains all build and deployment logic
- **overwrite/**: Directory containing files to overlay on top of upstream PixivUtil2 source
  - **Dockerfile**: Python-based container with APT caching support and entrypoint configuration
  - **entrypoint.sh**: Shell script that passes arguments to PixivUtil2.py
  - **.dockerignore**: Excludes dotfiles and build artifacts from Docker context
- **.env**: Environment configuration for registry, builders, and remote hosts
- **build/**: Temporary working directory created during builds (gitignored)

## Common Commands

### Building and Deploying

```bash
# Full build: fetch source + build amd64 image + push to registry
task build

# Prepare source code only (fetch and extract)
task prepare

# Build amd64 image (internal task, use `task build` instead)
task amd

# Build arm64 image
task arm
```

### Environment Variables

Required variables in `.env`:
- `REGISTRY`: Docker registry URL
- `APT_CACHER`: APT-Cacher NG proxy for faster package downloads
- `AMD64_BUILDER`: Docker buildx builder name for amd64 builds
- `REMOTE_HOST`: SSH host that pulls the built image

## Build System Details

### Task Dependencies

The `build` task depends on `prepare`, ensuring source code is always fresh before building.

### Overwrite Pattern

The build process uses an "overwrite" pattern to customize upstream PixivUtil2:

1. `prepare` task downloads upstream PixivUtil2 source to `{{.WORK_DIR}}`
2. Contents of `overwrite/` directory are copied on top, replacing/adding files
3. This allows customization (Dockerfile, entrypoint, .dockerignore) without forking PixivUtil2

To add custom files to the Docker image, place them in the `overwrite/` directory.

### Docker Build Context

- Build commands `cd` into `{{.WORK_DIR}}` before running docker buildx
- Build context is `.` (current directory after cd), not a subdirectory
- The Dockerfile uses `--build-arg apt_cacher` to optionally configure APT proxy
- entrypoint.sh allows passing arguments to PixivUtil2.py: `docker run <image> --download --user=username`

### Multi-Architecture Support

- **amd64**: Uses custom builder specified in `AMD64_BUILDER` env var
- **arm64**: Uses default builder, tags as `latest`

Both platforms push to registry immediately after build completes. The amd64 build additionally triggers the remote host to pull the new image via SSH.

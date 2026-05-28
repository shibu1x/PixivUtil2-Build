# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a build automation project for creating and deploying Docker images of PixivUtil2 (https://github.com/Nandaka/PixivUtil2). The project fetches the upstream source code, builds a Docker image for amd64, and pushes it to a private registry.

## Architecture

The build process follows this workflow:

1. **Source Preparation**: Downloads latest PixivUtil2 source from GitHub, extracts it to a temporary build directory
2. **Docker Build**: Builds amd64 image using buildx
3. **Registry Push**: Pushes image to private registry and triggers remote host to pull the latest image

### Key Components

- **Taskfile.yaml**: Task automation using Task (taskfile.dev). Contains all build and deployment logic
- **app/**: Directory containing files to overlay on top of upstream PixivUtil2 source
  - **Dockerfile**: Python-based container (`python:3.14-slim`); installs pip dependencies and sets entrypoint to `PixivUtil2.py`
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

# Build arm64 image (internal task, manual use only)
task arm
```

### Environment Variables

Required variables in `.env`:
- `APP_NAME`: Docker image name
- `REGISTRY`: Docker registry URL
- `APT_CACHER`: APT-Cacher NG proxy (passed as build arg; optional)
- `AMD64_BUILDER`: Docker buildx builder name for amd64 builds
- `REMOTE_HOST`: SSH host that pulls the built image

## Build System Details

### Task Dependencies

The `build` task depends on `prepare`, ensuring source code is always fresh before building.

### App Overlay Pattern

The build process uses an overlay pattern to customize upstream PixivUtil2:

1. `prepare` task downloads upstream PixivUtil2 source to `{{.WORK_DIR}}`
2. Contents of `app/` directory are copied on top, replacing/adding files
3. This allows customization (Dockerfile, .dockerignore) without forking PixivUtil2

To add custom files to the Docker image, place them in the `app/` directory.

### Docker Build Context

- Build commands run with `dir: build` so the build context is `build/`
- The Dockerfile installs Python dependencies via `pip install -r requirements.txt` and precompiles bytecode with `python -m compileall`
- The container entrypoint is `python PixivUtil2.py`; pass arguments directly: `docker run <image> --download --user=username`

### Multi-Architecture Support

- **amd64**: Uses custom builder specified in `AMD64_BUILDER` env var; built by `task build`
- **arm64**: Uses default builder; run manually with `task arm`

Both platforms push to registry immediately after build. The amd64 build additionally triggers the remote host to pull the new image via SSH.

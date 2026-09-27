# Dockette Workflows Specification

This document describes how GitHub Actions workflows in Dockette repositories are written.

## Table of Contents

- [Rules](#rules)
- [Triggers](#triggers)
- [Jobs](#jobs)
- [Reusable Workflow](#reusable-workflow)
- [Single Image Template](#single-image-template)
- [Multi Version Template](#multi-version-template)
- [Inline Build](#inline-build)
- [Dependabot](#dependabot)
- [Secrets](#secrets)
- [Action Versions](#action-versions)
- [Checklist](#checklist)

## Rules

- Every repository that publishes an image has `.github/workflows/docker.yml` named `Docker`.
- The workflow has `permissions: contents: read` at the top level.
- Images are built and pushed by the org reusable workflow `dockette/.github/.github/workflows/docker.yml@master`.
- Jobs are `test` → `build` → `docs`, chained with `needs`.
- The `test` job runs `make build` and `make test` (see [MAKEFILE.md](MAKEFILE.md)).
- Only `master` pushes to Docker Hub: pass `push: ${{ github.ref == 'refs/heads/master' }}` to the reusable workflow.
- Images are published to Docker Hub as `dockette/<repo>:<tag>`. No GHCR.
- Tags are plain names set in the workflow (`latest`, `8.4`, `8.4-fpm`, `bookworm-slim`). No `docker/metadata-action`, no git tag triggers.
- Multi-arch by default: `linux/amd64,linux/arm64`. Drop `arm64` only when the base image or software doesn't support it.
- Multi version images use a `strategy.matrix` with `fail-fast: false`.
- The reusable workflow gets secrets with `secrets: inherit`.
- Every repository with a workflow has `.github/dependabot.yml` for `github-actions`.
- Values are quoted (`"master"`, `"latest"`, `"dockette/example"`).

## Triggers

```yaml
on:
  workflow_dispatch:

  push:
    branches: ["master"]

  schedule:
    - cron: "0 8 * * 1"
```

| Trigger | Why |
|---------|-----|
| `workflow_dispatch` | Rebuild by hand from the Actions tab |
| `push` to `master` | Build and publish every merged change |
| `schedule` `0 8 * * 1` | Weekly rebuild (Monday 08:00 UTC) to pick up base image and package updates |
| `pull_request` | Optional. Runs `test` and a build without push; `docs` is skipped |

Don't use a bare `push:` (all branches). Use `pull_request` to test branches.

## Jobs

| Job | Runs on | Does |
|-----|---------|------|
| `test` | every trigger | `make build` + `make test` on `linux/amd64` |
| `build` | after `test` | Calls the reusable workflow; builds all platforms, pushes on `master` only |
| `docs` | after `build`, `master` only | Updates the Docker Hub description from `README.md` |

- `make build` builds into the local Docker daemon, so `make test` can run the image right away.
  Don't add `docker/setup-buildx-action` before it: that switches to a `docker-container` builder and the image is not loaded unless the Makefile passes `--load`.
- Building the test image with `docker/build-push-action` (`load: true`) and then `make test` is also fine, as long as the tag matches the Makefile's `DOCKER_IMAGE:DOCKER_TAG`.
- Put smoke tests in the Makefile `test` target, not as `docker run` steps in the workflow.
- Repositories that don't publish an image (compose stacks, build-only tools) keep `test` and `build` and skip `docs`.

## Reusable Workflow

`dockette/.github/.github/workflows/docker.yml@master` checks out the code, logs in to Docker Hub
(only when pushing), sets up QEMU and Buildx, and runs `docker/build-push-action` with the GitHub
Actions cache (`type=gha`).

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `image` | yes | | Image name, `dockette/<repo>` |
| `tag` | yes | | One tag, e.g. `latest` or `${{ matrix.tag }}` |
| `context` | no | `.` | Build context folder |
| `dockerfile` | no | `<context>/Dockerfile` | Dockerfile path |
| `platforms` | no | `linux/amd64,linux/arm64` | Build platforms |
| `push` | no | `true` | Push the image; always pass `${{ github.ref == 'refs/heads/master' }}` |

It takes one tag per call. To publish `1.2.3` and `latest`, add both to the matrix.

## Single Image Template

```yaml
name: "Docker"

on:
  workflow_dispatch:

  push:
    branches: ["master"]

  schedule:
    - cron: "0 8 * * 1"

permissions:
  contents: read

jobs:
  test:
    name: "Test"
    runs-on: "ubuntu-latest"

    steps:
      - name: "Checkout"
        uses: actions/checkout@v7

      - name: "Build image"
        run: "make build"

      - name: "Test image"
        run: "make test"

  build:
    name: "Build"
    needs: ["test"]
    uses: dockette/.github/.github/workflows/docker.yml@master
    secrets: inherit
    with:
      image: "dockette/example"
      tag: "latest"
      context: "."
      platforms: "linux/amd64,linux/arm64"
      push: ${{ github.ref == 'refs/heads/master' }}

  docs:
    name: "Docs"
    runs-on: "ubuntu-latest"
    needs: ["build"]
    if: github.ref == 'refs/heads/master'

    steps:
      - name: "Checkout"
        uses: actions/checkout@v7

      - name: "Update Docker Hub description"
        uses: peter-evans/dockerhub-description@v5
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
          repository: "dockette/example"
```

## Multi Version Template

One folder per version (`./8.4/Dockerfile`), matching the Makefile [multi version template](MAKEFILE.md#multi-version-template).
`version` is the folder, `tag` is set only when it differs (e.g. `latest`).

```yaml
name: "Docker"

on:
  workflow_dispatch:

  push:
    branches: ["master"]

  schedule:
    - cron: "0 8 * * 1"

permissions:
  contents: read

jobs:
  test:
    name: "Test (${{ matrix.version }})"
    runs-on: "ubuntu-latest"
    strategy:
      fail-fast: false
      matrix:
        version: ["1.0", "2.0"]

    steps:
      - name: "Checkout"
        uses: actions/checkout@v7

      - name: "Build image"
        run: "make build VERSION=${{ matrix.version }}"

      - name: "Test image"
        run: "make test VERSION=${{ matrix.version }}"

  build:
    name: "Build (${{ matrix.tag || matrix.version }})"
    needs: ["test"]
    strategy:
      fail-fast: false
      matrix:
        include:
          - { version: "1.0" }
          - { version: "2.0" }
          - { version: "2.0", tag: "latest" }

    uses: dockette/.github/.github/workflows/docker.yml@master
    secrets: inherit
    with:
      image: "dockette/example"
      tag: "${{ matrix.tag || matrix.version }}"
      context: "${{ matrix.version }}"
      platforms: "linux/amd64,linux/arm64"
      push: ${{ github.ref == 'refs/heads/master' }}

  docs:
    name: "Docs"
    runs-on: "ubuntu-latest"
    needs: ["build"]
    if: github.ref == 'refs/heads/master'

    steps:
      - name: "Checkout"
        uses: actions/checkout@v7

      - name: "Update Docker Hub description"
        uses: peter-evans/dockerhub-description@v5
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
          repository: "dockette/example"
```

- Keep the `test` and `build` version lists in sync.
- Per version platforms or Dockerfiles go in the matrix (`platforms: ${{ matrix.platforms }}`, `dockerfile: ${{ matrix.dockerfile }}`).
- Add `max-parallel` only when many heavy builds hit rate limits.

## Inline Build

Use inline steps instead of the reusable workflow only when it can't do the job, e.g. the image needs
`build-args` or several tags from one build. Keep the same order and guards as the reusable workflow:

```yaml
  build:
    name: "Build"
    needs: ["test"]
    runs-on: "ubuntu-latest"

    steps:
      - name: "Checkout"
        uses: actions/checkout@v7

      - name: "Login to DockerHub"
        if: github.ref == 'refs/heads/master'
        uses: docker/login-action@v4
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: "Set up QEMU"
        uses: docker/setup-qemu-action@v4

      - name: "Set up Docker Buildx"
        uses: docker/setup-buildx-action@v4

      - name: "Build and push"
        uses: docker/build-push-action@v7
        with:
          context: "."
          push: ${{ github.ref == 'refs/heads/master' }}
          tags: |
            dockette/example:${{ env.EXAMPLE_VERSION }}
            dockette/example:latest
          platforms: "linux/amd64,linux/arm64"
          build-args: |
            EXAMPLE_VERSION=${{ env.EXAMPLE_VERSION }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

- Log in only when pushing, so pull requests and Dependabot (no secrets) still pass.
- Use `type=gha` cache, not `actions/cache` with `type=local`.
- Never `push: true` unconditionally.

## Dependabot

`.github/dependabot.yml`:

```yaml
version: 2

updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: daily
```

Add more ecosystems (`npm`, `composer`, …) only for app code that lives in the repository. Base image
versions are managed by hand in the Dockerfiles and matrix, not by the `docker` ecosystem.

## Secrets

| Secret | Used by |
|--------|---------|
| `DOCKERHUB_USERNAME` | Docker Hub login, Docker Hub description |
| `DOCKERHUB_TOKEN` | Docker Hub access token (read/write), Docker Hub description |

Both are passed to the reusable workflow with `secrets: inherit`. No other secrets are needed.

## Action Versions

| Action | Version |
|--------|---------|
| `actions/checkout` | `v7` |
| `docker/setup-qemu-action` | `v4` |
| `docker/setup-buildx-action` | `v4` |
| `docker/login-action` | `v4` |
| `docker/build-push-action` | `v7` |
| `peter-evans/dockerhub-description` | `v5` |

Pin major versions only (`@v4`, not `@v4.6.0`). Dependabot keeps them current.

## Checklist

- [ ] `.github/workflows/docker.yml` exists, named `Docker`, with `permissions: contents: read`
- [ ] Triggers: `workflow_dispatch`, `push` to `master`, weekly `schedule` (`0 8 * * 1`)
- [ ] `test` job runs `make build` and `make test`
- [ ] `build` job uses `dockette/.github/.github/workflows/docker.yml@master` with `secrets: inherit`
- [ ] `push: ${{ github.ref == 'refs/heads/master' }}` is passed to the build
- [ ] Image is `dockette/<repo>`, platforms `linux/amd64,linux/arm64` unless unsupported
- [ ] Multi version images use a matrix with `fail-fast: false`
- [ ] `docs` job updates the Docker Hub description on `master`
- [ ] `.github/dependabot.yml` updates `github-actions`

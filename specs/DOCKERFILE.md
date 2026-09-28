# Dockette Dockerfile Specification

This document describes how Dockerfiles and related files (`.dockerignore`, `entrypoint.sh`) in Dockette repositories are written.

## Table of Contents

- [Rules](#rules)
- [Base Images](#base-images)
- [Instruction Order](#instruction-order)
- [Variables](#variables)
- [Installing Packages](#installing-packages)
- [Section Comments](#section-comments)
- [Files and Entrypoints](#files-and-entrypoints)
- [Debian Template](#debian-template)
- [Alpine Template](#alpine-template)
- [Thin Republish Template](#thin-republish-template)
- [Multi-stage Builds](#multi-stage-builds)
- [Folder Layout](#folder-layout)
- [Dockerignore](#dockerignore)
- [Checklist](#checklist)

## Rules

- Every image starts from a tagged base image. Never use an untagged image or `:latest`.
- Prefer `dockette/debian:<codename>[-slim]` or `dockette/alpine:<x.y>` as the base. Use an official upstream image when the image is built around that software (`postgres`, `mariadb`, `node`, ...).
- `LABEL maintainer="Milan Sulc <sulcmil@gmail.com>"` comes right after `FROM`. Do not use `MAINTAINER`.
- Use `ENV KEY=value`, never the legacy `ENV KEY value` form.
- Pinned upstream versions are named `<NAME>_VERSION` (`ADMINER_VERSION`, `DEPLOYER_VERSION`, `NODE_VERSION`).
- Install, configure and clean up in one `RUN`, chained with `&& \`. Cleanup in a later `RUN` does not make the image smaller.
- Use `apt-get` on Debian (not `apt`) and `apk add --no-cache` on Alpine.
- Use `COPY` for local files. Use `ADD` only when you need its remote URL or tar extract behaviour.
- `CMD` and `ENTRYPOINT` use the exec (JSON array) form.
- `WORKDIR /srv` is the default working directory for app and tool images.
- `EXPOSE` every port a service listens on.
- Multi-stage names use uppercase `AS` (`FROM golang:1.27 AS build`).
- Indent continuation lines with 4 spaces, one package per line in long install lists.

## Base Images

| Use case | Base |
|----------|------|
| General Debian image | `dockette/debian:bookworm`, `dockette/debian:trixie-slim` |
| General Alpine image | `dockette/alpine:3.23` |
| PHP tooling | `dockette/php:<x.y>` |
| Built around upstream software | Official image with explicit tag (`postgres:17`, `mariadb:11.8`, `node:20-bookworm-slim`) |
| Final stage of a static binary | `gcr.io/distroless/static-debian13:nonroot` or `scratch` |

- Pin to a Debian codename or an Alpine/upstream minor version (`trixie-slim`, `3.23`, `11.8`).
- Prefer the `-slim` variant when the full image is not needed.
- Do not use EOL aliases such as `dockette/jessie` or `dockette/stretch`; use `dockette/debian:<codename>`.
- To make the upstream tag configurable, declare `ARG <NAME>_VERSION=<default>` before `FROM` and repeat `ARG <NAME>_VERSION` after it if a later instruction needs it.

## Instruction Order

```
FROM
LABEL
ARG / ENV
RUN            (install + configure + clean up)
COPY
WORKDIR
EXPOSE
USER
HEALTHCHECK
ENTRYPOINT
CMD
```

Group related `ENV` lines under a short comment (`# PHP`, `# COMPOSER`) when there are more than a few.

## Variables

- `ENV` for values the running container also needs (paths, ports, defaults users may override).
- `ARG` for build-only values (`TARGETARCH`, base image tag).
- `TZ=Europe/Prague` when the image deals with time (PHP, cron, databases).
- Reference variables as `${VAR}`.

```dockerfile
ENV DEPLOYER_VERSION=8.0.5
ENV DEPLOYER_BIN=/usr/local/bin/dep
```

## Installing Packages

Debian:

```dockerfile
RUN apt-get update && \
    apt-get dist-upgrade -y && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl && \
    # CLEAN UP #################################################################
    apt-get clean -y && \
    apt-get autoclean -y && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/* /var/lib/log/* /tmp/* /var/tmp/*
```

Alpine:

```dockerfile
RUN apk update && \
    apk upgrade && \
    apk add --no-cache \
        bash \
        curl && \
    # CLEAN UP #################################################################
    rm -rf /var/cache/apk/* /tmp/*
```

- Remove build-only tools (`wget`, `unzip`) in the same `RUN` with `apt-get remove -y` or `apk del`.
- Never use Alpine cleanup (`/var/cache/apk`) in a Debian image, or the other way round.
- Extra repositories (`@community`, sury.org, nodesource) are added in the same `RUN`, before the install.

## Section Comments

Inside a long `RUN`, each logical step gets an uppercase comment padded with `#` to column 80:

```dockerfile
RUN apt-get update && apt-get dist-upgrade -y && \
    # DEPENDENCIES #############################################################
    apt-get install -y --no-install-recommends wget curl && \
    # COMPOSER #################################################################
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
    # CLEAN UP #################################################################
    apt-get clean -y && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

Outside `RUN`, use short uppercase comments (`# WORKDIR`, `# COMMAND`) or the same padded style.

## Files and Entrypoints

- Copy an entrypoint with its mode set: `COPY --chmod=0755 entrypoint.sh /entrypoint.sh`.
- Use `tini` as `ENTRYPOINT` when the container runs a server or more than one process.
- `entrypoint.sh` starts with `#!/bin/bash` and `set -Eeo pipefail`.
- The last command `exec`s the main process, so it receives signals as PID 1 (`exec "$@"`, `exec supervisord ...`).

```bash
#!/bin/bash
set -Eeo pipefail

# prepare config from env ...

exec "$@"
```

## Debian Template

```dockerfile
FROM dockette/debian:trixie-slim

LABEL maintainer="Milan Sulc <sulcmil@gmail.com>"

ENV EXAMPLE_VERSION=1.2.3
ENV TZ=Europe/Prague

# INSTALLATION
RUN apt-get update && \
    apt-get dist-upgrade -y && \
    # DEPENDENCIES #############################################################
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        tini && \
    # EXAMPLE ##################################################################
    curl -fsSL -o /usr/local/bin/example https://example.com/releases/v${EXAMPLE_VERSION}/example && \
    chmod +x /usr/local/bin/example && \
    # CLEAN UP #################################################################
    apt-get clean -y && \
    apt-get autoclean -y && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/* /var/lib/log/* /tmp/* /var/tmp/*

# FILES
COPY --chmod=0755 entrypoint.sh /entrypoint.sh

# WORKDIR
WORKDIR /srv

EXPOSE 8000

ENTRYPOINT ["tini", "--", "/entrypoint.sh"]
CMD ["example"]
```

## Alpine Template

```dockerfile
FROM dockette/alpine:3.23

LABEL maintainer="Milan Sulc <sulcmil@gmail.com>"

ENV EXAMPLE_VERSION=1.2.3

# INSTALLATION
RUN apk update && \
    apk upgrade && \
    # DEPENDENCIES #############################################################
    apk add --no-cache \
        bash \
        ca-certificates \
        curl && \
    # EXAMPLE ##################################################################
    curl -fsSL -o /usr/local/bin/example https://example.com/releases/v${EXAMPLE_VERSION}/example && \
    chmod +x /usr/local/bin/example && \
    # CLEAN UP #################################################################
    rm -rf /var/cache/apk/* /tmp/*

# WORKDIR
WORKDIR /srv

# COMMAND
CMD ["example"]
```

## Thin Republish Template

For images that only re-tag or lightly patch an upstream image:

```dockerfile
ARG EXAMPLE_VERSION=1.2.3

FROM ghcr.io/example/example:${EXAMPLE_VERSION}

ARG EXAMPLE_VERSION

LABEL maintainer="Milan Sulc <sulcmil@gmail.com>"
LABEL org.opencontainers.image.title="Example"
LABEL org.opencontainers.image.description="Thin republish of ghcr.io/example/example for Dockette"
LABEL org.opencontainers.image.version="${EXAMPLE_VERSION}"
LABEL org.opencontainers.image.source="https://github.com/dockette/example"
```

## Multi-stage Builds

Use a build stage when the image needs compilers or package managers that the runtime does not:

```dockerfile
FROM golang:1.27-alpine AS build

WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /out/app ./cmd/app

FROM gcr.io/distroless/static-debian13:nonroot

LABEL maintainer="Milan Sulc <sulcmil@gmail.com>"

COPY --from=build /out/app /app

EXPOSE 8080

ENTRYPOINT ["/app"]
```

- Name every build stage (`AS build`, `AS deps`) and copy with `--from=<name>`.
- Build stages do not need cleanup; the final stage does.
- Services run as a non-root `USER` when the base image provides one (`nonroot`, `bun`, `coder`). Tool, CI and base images may run as root.
- Long-running services should declare a `HEALTHCHECK`.

## Folder Layout

Single image, the Dockerfile is in the repository root:

```
.
├── .dockerignore
├── Dockerfile
├── entrypoint.sh
├── Makefile
└── README.md
```

Multiple versions or variants, one self-contained folder per tag:

```
.
├── 8.4
│   ├── Dockerfile
│   └── conf.d/
├── 8.4-fpm
│   ├── Dockerfile
│   └── conf.d/
├── 8.5
│   ├── Dockerfile
│   └── conf.d/
├── Makefile
└── README.md
```

- The folder name is the image tag (`8.4`, `8.4-fpm`, `3.23`, `bookworm`). Variant repos use the variant name (`adminer-postgres`, `debian-13`).
- Each folder has everything it needs, so it builds with `docker buildx build ./<tag>`.
- A slim variant of the same version lives next to it as `Dockerfile.slim`.
- When several versions share files, keep them in a `shared/` folder, build from the repository root with `-f ./<tag>/Dockerfile .`, and reference `./shared/...` in `COPY`.
- Do not hide the Dockerfile in `.docker/` unless the repository root is an application with its own code (then use `.docker/Dockerfile`).

## Dockerignore

A repository that copies files from its build context should have a `.dockerignore`:

```
.claude
.git
.github
.docs
.DS_Store
*.md
LICENSE
Makefile
docker-compose*.yml
fxnorm.yml
node_modules
tests
```

Do not ignore files the Dockerfile copies (for example a `README.md` used by the image).

## Checklist

- [ ] `FROM` uses an explicit tag, never `:latest` or no tag
- [ ] Base is `dockette/debian`, `dockette/alpine` or a tagged official image
- [ ] `LABEL maintainer="Milan Sulc <sulcmil@gmail.com>"` after `FROM`, no `MAINTAINER`
- [ ] `ENV KEY=value` form, versions named `<NAME>_VERSION`
- [ ] Install and cleanup in the same `RUN`
- [ ] Debian: `apt-get`, `--no-install-recommends`, `rm -rf /var/lib/apt/lists/*`
- [ ] Alpine: `apk add --no-cache`, `rm -rf /var/cache/apk/*`
- [ ] Long `RUN` steps have `# SECTION ####` comments
- [ ] `COPY` for local files, not `ADD`
- [ ] `WORKDIR` set (usually `/srv`)
- [ ] `EXPOSE` for every listening port
- [ ] `CMD` / `ENTRYPOINT` in exec form
- [ ] `entrypoint.sh` uses `set -Eeo pipefail` and `exec`s the main process
- [ ] Build stages named with uppercase `AS`
- [ ] One folder per version or variant, named after the tag
- [ ] `.dockerignore` present when the build context has files to copy

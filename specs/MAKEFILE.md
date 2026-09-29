# Dockette Makefile Specification

This document describes how Makefiles in Dockette repositories are written.

## Table of Contents

- [Rules](#rules)
- [Help](#help)
- [Environment](#environment)
- [Variables](#variables)
- [Single Image Template](#single-image-template)
- [Multi Version Template](#multi-version-template)
- [Target Names](#target-names)
- [Checking with fxnorm](#checking-with-fxnorm)
- [Checklist](#checklist)

## Rules

- Running `make` without a target prints the help, same as `make help`.
- Every public target has a `## Description` comment on its target line.
- Every public target has its own `.PHONY` line directly above it. Private `_` pattern rules need none.
- `build`, `test` and `run` always exist. There is no Dockette repository without a `test` target.
- Targets are grouped into sections with `##@ Section` headers.
- Env files are included at the top when the Makefile uses environment variables.
- Image name, tag and platforms are variables (`DOCKER_IMAGE`, `DOCKER_TAG`, `DOCKER_PLATFORMS`).
- Images are built with `docker buildx build --platform ${DOCKER_PLATFORMS}`.
- Private helper targets start with `_` and have no `##` comment, so they are hidden from the help.
- Variables are referenced as `${VAR}`.
- Recipes are indented with tabs.

## Help

Put this block after the env includes and variables:

```makefile
.DEFAULT_GOAL := help

##@ Help

.PHONY: help
help: ## Show this help
	@awk 'BEGIN {FS = ":.*##"; printf "Usage: make \033[36m<target>\033[0m\n"} /^[a-zA-Z0-9_.-]+:.*##/ { sub(/^ +/, "", $$2); printf "  \033[36m%-20s\033[0m %s\n", $$1, $$2 } /^##@/ { printf "\n\033[1m%s\033[0m\n", substr($$0, 5) }' $(firstword $(MAKEFILE_LIST))
```

- `.DEFAULT_GOAL := help` makes plain `make` show the help, regardless of target order.
- The awk script lists every `target: ## Description` line and prints `##@ Section` lines as headers.
- It reads `$(firstword $(MAKEFILE_LIST))`, so included `.env` files are never parsed.
- It works with GNU awk, mawk and BSD awk (macOS).

Output:

```
Usage: make <target>

Help
  help                 Show this help

Docker
  build                Build image
  test                 Test image
  run                  Run image
  push                 Push image
```

## Environment

When the image needs environment variables at build or run time (tokens, S3 credentials, ports),
include the env file and export it at the top of the Makefile:

```makefile
-include .env
export
```

- `.env` is local and listed in `.gitignore`.
- A committed `.env.dist` lists all variables, with empty or safe default values.
- `-include` (with the dash) keeps `make help` working before `.env` exists.
- `export` passes the loaded variables to every recipe.
- When the defaults from `.env.dist` must always be loaded, include them first and let `.env` override them:

```makefile
include .env.dist
-include .env
export
```

Pass variables to the container explicitly with `-e VAR=${VAR}` (or `--env-file .env`).

Makefiles that don't use any environment variables don't include env files.

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `DOCKER_IMAGE` | `dockette/<name>` | Image name, fixed (`=`) |
| `DOCKER_TAG` | `latest` | Image tag, overridable (`?=`) |
| `DOCKER_PLATFORMS` | `linux/amd64` | Build platforms, overridable (`?=`), e.g. `linux/amd64,linux/arm64` |

Use `DOCKER_PLATFORMS` (plural), not `DOCKER_PLATFORM`. Don't use bare `IMAGE`/`TAG`.

Multi version repositories use `VERSION?=<latest>` instead of `DOCKER_TAG` (see
[Multi Version Template](#multi-version-template)).

Override on the command line:

```bash
make build DOCKER_TAG=dev DOCKER_PLATFORMS=linux/arm64
```

## Single Image Template

```makefile
DOCKER_IMAGE=dockette/example
DOCKER_TAG?=latest
DOCKER_PLATFORMS?=linux/amd64

.DEFAULT_GOAL := help

##@ Help

.PHONY: help
help: ## Show this help
	@awk 'BEGIN {FS = ":.*##"; printf "Usage: make \033[36m<target>\033[0m\n"} /^[a-zA-Z0-9_.-]+:.*##/ { sub(/^ +/, "", $$2); printf "  \033[36m%-20s\033[0m %s\n", $$1, $$2 } /^##@/ { printf "\n\033[1m%s\033[0m\n", substr($$0, 5) }' $(firstword $(MAKEFILE_LIST))

##@ Docker

.PHONY: build
build: ## Build image
	docker buildx build --platform ${DOCKER_PLATFORMS} --pull -t ${DOCKER_IMAGE}:${DOCKER_TAG} .

.PHONY: test
test: ## Test image
	docker run --rm --platform ${DOCKER_PLATFORMS} ${DOCKER_IMAGE}:${DOCKER_TAG} example --version

.PHONY: run
run: ## Run image
	docker run --rm -it --platform ${DOCKER_PLATFORMS} -p 8000:80 ${DOCKER_IMAGE}:${DOCKER_TAG}

.PHONY: push
push: ## Push image
	docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
```

`test` is a smoke test: run the image and check the main binary answers (`--version`, config test, file exists).

- A service image starts the container in the background, waits for the port and checks one response, then
  removes the container: `docker run -d --name {name}-test ...`, a `curl` retry loop, `docker rm -f {name}-test`.
- When the workflow has its own smoke test steps, move them into `test` and let the workflow call `make test`
  ([WORKFLOWS.md](WORKFLOWS.md)). The workflow and the Makefile must not test different things.
- An image that has no test today adds `test` before anything else changes in the `Makefile`. Until it does,
  `AGENTS.md` shows the command that tests the image today (see
  [AGENTS.md](AGENTS.md#sections)).

## Multi Version Template

Repositories with one image per version or variant (`php`, `debian`, `adminer`, `postgres`, …) keep each
version in its own folder (`./8.4/Dockerfile`) and use private pattern rules. `build`, `test` and `run`
work on `VERSION`, which defaults to the latest version.

```makefile
DOCKER_IMAGE=dockette/example
DOCKER_PLATFORMS?=linux/amd64
VERSION?=2.0

.DEFAULT_GOAL := help

##@ Help

.PHONY: help
help: ## Show this help
	@awk 'BEGIN {FS = ":.*##"; printf "Usage: make \033[36m<target>\033[0m\n"} /^[a-zA-Z0-9_.-]+:.*##/ { sub(/^ +/, "", $$2); printf "  \033[36m%-20s\033[0m %s\n", $$1, $$2 } /^##@/ { printf "\n\033[1m%s\033[0m\n", substr($$0, 5) }' $(firstword $(MAKEFILE_LIST))

##@ Docker

.PHONY: build
build: _docker-build-${VERSION} ## Build image (VERSION=x)

.PHONY: test
test: _docker-test-${VERSION} ## Test image (VERSION=x)

.PHONY: run
run: _docker-run-${VERSION} ## Run image (VERSION=x)

.PHONY: build-all
build-all: _docker-build-1.0 _docker-build-2.0 ## Build all images

.PHONY: test-all
test-all: _docker-test-1.0 _docker-test-2.0 ## Test all images

_docker-build-%:
	docker buildx build --platform ${DOCKER_PLATFORMS} --pull -t ${DOCKER_IMAGE}:$* ./$*

_docker-test-%:
	docker run --rm --platform ${DOCKER_PLATFORMS} ${DOCKER_IMAGE}:$* example --version

_docker-run-%:
	docker run --rm -it --platform ${DOCKER_PLATFORMS} ${DOCKER_IMAGE}:$*
```

Usage:

```bash
make build              # latest version
make build VERSION=1.0  # specific version
make build-all          # all versions
```

## Target Names

| Target | Purpose |
|--------|---------|
| `help` | Show available targets (default) |
| `build` | Build image |
| `test` | Smoke test image |
| `run` | Run image interactively |
| `push` | Push image |
| `build-all` | Build all versions/variants |
| `test-all` | Test all versions/variants |
| `enter` | Open shell in running container |
| `up` | Start `docker compose` stack |

Avoid `docker-build`, `docker-push` and `docker-build-<version>`; use `build`, `push` and `VERSION=<version>`.

## Checking with fxnorm

`fxnorm check` checks the Makefile against this document. `fxnorm fix` adds `.DEFAULT_GOAL := help` and the
[help block](#help) when they are missing; the other findings are fixed by hand.

```bash
# Once per repository: write fxnorm.yml with the dockette-image preset
fxnorm init

# Check, or fix what can be fixed and check again
fxnorm check
fxnorm fix
```

| Rule | Checks |
|------|--------|
| `common/makefile-help` | `.DEFAULT_GOAL := help` and a `help` target (fixable) |
| `common/makefile-target-descriptions` | Every public target has a `## Description` comment |
| `dockette/makefile-exists` | A `Makefile` exists in the root |
| `dockette/makefile-phony` | Every public target has its own `.PHONY` line directly above it |
| `dockette/makefile-targets` | `build`, `test` and `run` exist |
| `dockette/makefile-target-names` | No public `docker-build`, `docker-push` or `docker-build-<version>` |
| `dockette/makefile-docker-image` | `DOCKER_IMAGE`, not a bare `IMAGE` |
| `dockette/makefile-docker-tag` | `DOCKER_TAG?=`, or `VERSION?=` in multi version repositories |
| `dockette/makefile-docker-platforms` | `DOCKER_PLATFORMS?=`, not `DOCKER_PLATFORM` |
| `dockette/makefile-buildx` | Every build uses `docker buildx build --platform ${DOCKER_PLATFORMS}` |
| `dockette/makefile-env-include` | `-include .env`, not `include .env` |

- `fxnorm.yml` is committed in the root and listed in `.dockerignore` when the Dockerfile copies the whole build
  context ([DOCKERFILE.md](DOCKERFILE.md#dockerignore)). In a Contributte library it is export-ignored in
  `.gitattributes`, together with `AGENTS.md`.
- The Makefile has no `fxnorm` target. fxnorm runs the same way in every repository.
- `fxnorm explain {rule id}` shows what a rule checks. When a rule and this document disagree, this document wins;
  report the rule.

## Checklist

- [ ] `make` prints the help
- [ ] Every public target has a `## Description`
- [ ] Every target has its own `.PHONY` (except `_` pattern rules)
- [ ] `DOCKER_IMAGE`, `DOCKER_TAG`, `DOCKER_PLATFORMS` variables are used
- [ ] `build`, `test` and `run` targets exist; `test` is the smoke test CI runs
- [ ] `fxnorm check` reports no findings in the `Makefile`
- [ ] `.env` is included (`-include .env` + `export`) if the image needs environment variables
- [ ] `.env.dist` lists the variables and `.env` is in `.gitignore`

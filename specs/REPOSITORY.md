# Dockette Repository Specification

This document describes the repository-level files of Dockette repositories: README, LICENSE, editor and git
config files, Compose examples and the folder layout. Dockerfiles are covered in [DOCKERFILE.md](DOCKERFILE.md),
Makefiles in [MAKEFILE.md](MAKEFILE.md), CI in [WORKFLOWS.md](WORKFLOWS.md) and agent files in
[AGENTS.md](AGENTS.md). How to write the text is described in [TONE.md](TONE.md).

## Table of Contents

- [Rules](#rules)
- [Layout](#layout)
- [README](#readme)
- [README Template](#readme-template)
- [Versions and Environment Tables](#versions-and-environment-tables)
- [LICENSE](#license)
- [EditorConfig](#editorconfig)
- [Gitignore](#gitignore)
- [Compose](#compose)
- [Agent Files](#agent-files)
- [Checklist](#checklist)

## Rules

- Every repository has `README.md`, `LICENSE`, `.editorconfig` and `Makefile` in the root.
- The README starts with the centered `Dockette / {Name}` header, the standard badge row and a short description.
- The README ends with the `## Maintenance` section and the standard footer text.
- The license is MIT, held by `Dockette`.
- Badges use `img.shields.io` and the GitHub Actions badge. No `badgen.net`, no Gitter, no Docker Hub stars.
- Support goes to GitHub Discussions (`https://github.com/orgs/dockette/discussions`), not Gitter.
- Secrets and local config (`.env`) are never committed; commit `.env.dist` instead.
- Compose files have no top-level `version:` key.

## Layout

Single image:

```
.github/workflows/docker.yml
.editorconfig
AGENTS.md
CLAUDE.md
Dockerfile
LICENSE
Makefile
README.md
```

Multiple versions or variants (`php`, `debian`, `nodejs`, `postgres`, …) keep one folder per image tag,
named after the tag, each with its own `Dockerfile`:

```
.github/workflows/docker.yml
8.4/Dockerfile
8.5/Dockerfile
.editorconfig
AGENTS.md
CLAUDE.md
LICENSE
Makefile
README.md
```

Optional files and folders:

| Path | Purpose |
|------|---------|
| `.docs/` | Images used by the README (screenshots, diagrams) |
| `.env.dist` | Committed list of environment variables, see [MAKEFILE.md](MAKEFILE.md#environment) |
| `.gitignore` | Local files that must not be committed |
| `.dockerignore` | Build context filter, see [DOCKERFILE.md](DOCKERFILE.md) |
| `docker-compose.yml` | Runnable example stack |
| `DESIGN.md` | Visual design, only for images that ship a web UI, see [DESIGN.md](DESIGN.md) |
| `PRD.md` + `TECH.md` | Product and technical design, only for app-like images and stacks, see [PRD.md](PRD.md) and [TECH.md](TECH.md) |
| `entrypoint.sh`, config files | Files copied into the image |

- README images live in `.docs/`, not in the root.
- Don't commit IDE or cloud workspace configs (`.gitpod.yml`, `.idea/`, `.vscode/`).
- Don't commit large binaries into git; if they must be versioned, use Git LFS via `.gitattributes`.

## README

Sections, in this order:

| Section | Required | Content |
|---------|----------|---------|
| Header | yes | Centered `<h1>`, badge row, description, `-----` |
| `## Usage` | yes | Copy-paste `docker run` commands, `docker compose` steps or a `FROM` example |
| `## Versions` | multi-tag repos | Table of tags, see below |
| `## Configuration` / `## Environment` | when relevant | Env variables table, mounted paths, ports |
| Other topic sections | optional | Documentation, tips, legacy notes |
| `## Development` | when a Makefile exists | `make build`, `make test`, `make run` |
| `## Maintenance` | yes | Standard footer, always last |

Header:

- Title is `<h1 align=center>Dockette / {Name}</h1>`, not a Markdown `# Title`.
- Badge row has exactly four badges, in this order: GitHub Actions (`docker.yml`), Docker Hub pulls, GitHub Sponsors, Discussions.
- Every badge `<img>` has an `alt` text.
- The description is one centered paragraph of one to three sentences: what is inside, what it is based on and who
  it is for. Upstream projects are linked.
- The author links line (`🕹 f3l1x.io | 💻 f3l1x | 🐦 @xf3l1x`) is optional; when used, it goes after the description.
- A screenshot (from `.docs/`) is optional and goes after the description.
- The header ends with a `-----` rule.
- Drop the Docker Hub pulls badge only when no image is published to Docker Hub.

Usage:

- Start with one runnable command, introduced by a sentence that ends with a colon.
- Follow it with one sentence on the base and what the image needs: "Based on Debian Bookworm. Mount your project
  to `/srv`."
- Use fenced code blocks (`sh`, `yaml`, `Dockerfile`) with real, runnable commands.
- Use `dockette/{name}:{tag}` image names. Split long `docker run` commands with `\`.
- List one command per published tag when the repo has many versions.

Maintenance footer, exactly:

```markdown
## Maintenance

See [how to contribute](https://github.com/dockette/.github/blob/master/CONTRIBUTING.md) to this package. Consider [supporting](https://github.com/sponsors/f3l1x) **f3l1x**. Thank you for using this package.
```

Don't use the old Contributte footer (`contributte.org/contributing.html`, maintainer avatars).

## README Template

````markdown
<h1 align=center>Dockette / {Name}</h1>

<p align=center>
   <a href="https://github.com/dockette/{name}/actions"><img src="https://github.com/dockette/{name}/actions/workflows/docker.yml/badge.svg" alt="GitHub Actions"></a>
   <a href="https://hub.docker.com/r/dockette/{name}"><img src="https://img.shields.io/docker/pulls/dockette/{name}.svg" alt="Docker Hub pulls"></a>
   <a href="https://github.com/sponsors/f3l1x"><img src="https://img.shields.io/badge/sponsor-GitHub%20Sponsors-ea4aaa" alt="GitHub Sponsors"></a>
   <a href="https://github.com/orgs/dockette/discussions"><img src="https://img.shields.io/badge/support-discussions-6f42c1" alt="Support/Discussions"></a>
</p>

<p align=center>
   {What is inside, with a link to <a href="{upstream-url}">{Upstream}</a>. What it is based on and who it is for.}
</p>

-----

## Usage

Run {what the command does}:

```sh
docker run -it --rm dockette/{name}:{tag} {command}
```

Based on `{base image}`. {What it needs: volumes, ports, a config file.}

## Versions

| Tag | Description |
|-----|-------------|
| `dockette/{name}:{tag}` | {What is inside} |
| `dockette/{name}:latest` | Same as `{tag}` |

## Environment

| Variable | Default | Description |
|----------|---------|-------------|
| `{VAR}` | `{default}` | {What it does} |

## Development

```sh
make build
make test
make run
```

## Maintenance

See [how to contribute](https://github.com/dockette/.github/blob/master/CONTRIBUTING.md) to this package. Consider [supporting](https://github.com/sponsors/f3l1x) **f3l1x**. Thank you for using this package.
````

Remove `## Versions` and `## Environment` when they don't apply.

## Versions and Environment Tables

- Tag tables use the columns `Tag | Description`. Republished upstream images use `Tag | Upstream`.
- Always show the full image reference (`dockette/{name}:{tag}`) in backticks and say what `latest` points to.
- Env tables use the columns `Variable | Default | Description`, with names and defaults in backticks.
- Keep tables in sync with the folders and the Makefile `VERSION` list.

## LICENSE

- MIT License, file named `LICENSE` (no extension).
- Holder is `Dockette`: `Copyright (c) {year} Dockette`.
- The year is the year the repository was created. Don't bump it.
- Exception: a repository that redistributes third-party binaries keeps the vendor's license (for example `oracle-instantclient`).

## EditorConfig

```ini
# EditorConfig is awesome: http://EditorConfig.org

root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 4

[Makefile]
indent_style = tab

[*.{json,yml,yaml}]
indent_size = 2
```

- Globs must not contain spaces: `[*.{json,yml}]`, not `[{*.json, *.yml}]` (the spaced form doesn't match).
- `Makefile` always uses tabs.
- Add sections for other languages in the repo (for example `[*.{js,ts}]` with 2 spaces).

## Gitignore

Add `.gitignore` only when the repository creates local files. Use root-anchored paths and group them with comments:

```gitignore
# Envs
.env

# Project
/.tmp
/.docker

# OS
.DS_Store
```

- `.env` must be ignored whenever `.env.dist` exists or the Makefile includes `.env`.
- Ignore build output (`/dist`), dependencies (`/node_modules`, `/vendor`) and runtime data (`/data`).

## Compose

`docker-compose.yml` is an example of how to run the image, not the build definition.

```yaml
services:
    {name}:
        image: dockette/{name}:{tag}
        ports:
            - "8000:8000"
        environment:
            - {VAR}=${VAR:-default}
```

- No top-level `version:` key (obsolete in Compose v2).
- Service names match what they run.
- Use `dockette/{name}` images; use `build:` only for local development stacks.
- Values that differ per user come from `.env` (`${VAR:-default}`), listed in `.env.dist`.
- The README shows how to start it (`docker compose up`).

## Agent Files

When a repository has agent instructions:

- `AGENTS.md` holds the instructions. Its content is described in [AGENTS.md](AGENTS.md).
- `CLAUDE.md` contains only the line `@AGENTS.md`.

## Checklist

- [ ] `README.md`, `LICENSE`, `.editorconfig` and `Makefile` exist in the root
- [ ] README header is `<h1 align=center>Dockette / {Name}</h1>`
- [ ] Badge row: GitHub Actions, Docker Hub pulls, GitHub Sponsors, Discussions (shields.io, with `alt`)
- [ ] No Gitter, badgen.net or Docker Hub stars badges
- [ ] Description of one to three sentences: what is inside, what it is based on, who it is for
- [ ] `## Usage` starts with a runnable command and a sentence on the base and requirements
- [ ] Tags and env variables are documented in tables when the image has them
- [ ] README ends with `## Maintenance` and the standard footer ("Consider supporting …")
- [ ] Text follows [TONE.md](TONE.md)
- [ ] LICENSE is MIT, `Copyright (c) {year} Dockette`
- [ ] `.editorconfig` globs have no spaces and `Makefile` uses tabs
- [ ] `.env` is ignored and `.env.dist` is committed when env variables are used
- [ ] `docker-compose.yml` has no `version:` key
- [ ] README images are in `.docs/`
- [ ] `AGENTS.md` exists and `CLAUDE.md` is `@AGENTS.md`

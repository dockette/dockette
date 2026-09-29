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
- [Screenshots](#screenshots)
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
- `## Usage` is one `docker run` with the untagged image, a short note and a docs link; `## Development` is a
  few `make` commands. Both stay short (see [README](#readme)).
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
Dockerfile
LICENSE
Makefile
README.md
fxnorm.yml
```

Multiple versions or variants (`php`, `debian`, `nodejs`, `postgres`, …) keep one folder per image tag,
named after the tag, each with its own `Dockerfile`:

```
.github/workflows/docker.yml
8.4/Dockerfile
8.5/Dockerfile
.editorconfig
AGENTS.md
LICENSE
Makefile
README.md
fxnorm.yml
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

- README images live directly in `.docs/`, not in the root and not in `.docs/assets/`. Variants of one screen may
  use a subfolder (`.docs/themes/`).
- Don't commit IDE or cloud workspace configs (`.gitpod.yml`, `.idea/`, `.vscode/`).
- Don't commit large binaries into git; if they must be versioned, use Git LFS via `.gitattributes`.

## README

Sections, in this order:

| Section | Required | Content |
|---------|----------|---------|
| Header | yes | Centered `<h1>`, badge row, description, `-----` |
| `## Usage` | yes | One `docker run` (or one `FROM` line for base images), a short note and a link to the docs |
| `## Versions` | multi-tag repos | Table of tags, see below |
| `## Environment` | only when the image reads env variables | Env variables table |
| Other topic sections | optional | Documentation, tips, legacy notes |
| `## Development` | when a Makefile exists | 3 to 5 `make` commands with comments, then "Run `make` to list every target." |
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

Usage is simple and flexible:

- One `docker run` with only the essential argument (the mount or the command the image needs) and the untagged
  image name `dockette/{name}`, introduced by a sentence that ends with a colon. A base image shows one `FROM`
  line instead.
- Then a short paragraph: what the image adds or leaves out, what it needs, and a link to the upstream docs or the
  configuration reference.
- One or two code blocks in total. No ports, container names, pinned tags, long config samples, multiple variants
  or step-by-step walkthroughs. Tags go to `## Versions`, env variables to `## Environment`, the upstream config
  to its own docs.
- A `docker-compose.yml`, when the repository has one, is started with one `docker compose up` line; the file itself
  is linked, not pasted.

Development is high level:

- One `sh` block of 3 to 5 `make` commands that exist in the `Makefile`, each with a short comment.
- Then exactly "Run `make` to list every target."
- No options (`VERSION=`, `DOCKER_TAG=`), paths or env variables; those live in the Makefile help and `AGENTS.md`.

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

<!-- optional: the author links line, then one screenshot from .docs/ -->

-----

## Usage

{Mount your config and run it / Run it}:

```sh
docker run {essential argument} dockette/{name}
```

{What the image adds or leaves out and what it needs, one to three sentences}, see the
[{upstream docs}]({docs-url}).

## Versions

| Tag | Description |
|-----|-------------|
| `dockette/{name}:{tag}` | {What is inside} |
| `dockette/{name}:latest` | Same as `{tag}` |

## Environment

<!-- only when the image reads env variables -->

| Variable | Default | Description |
|----------|---------|-------------|
| `{VAR}` | `{default}` | {What it does} |

## Development

```sh
make build   # build the image
make test    # smoke test it
make run     # run it locally
```

Run `make` to list every target.

## Maintenance

See [how to contribute](https://github.com/dockette/.github/blob/master/CONTRIBUTING.md) to this package. Consider [supporting](https://github.com/sponsors/f3l1x) **f3l1x**. Thank you for using this package.
````

`{...}` marks a placeholder: replace it with facts from the repository; delete lines that don't apply. Remove
`## Versions` when the repository has one tag and `## Environment` when the image reads no env variables. The
approved reference is the `dockette/pgbouncer` README. Topic sections (a local config, a Compose example, a
screenshot gallery) go after `## Environment` and before `## Development`, in the order a user needs them.

## Versions and Environment Tables

- Tag tables use the columns `Tag | Description`. Republished upstream images use `Tag | Upstream`.
- Always show the full image reference (`dockette/{name}:{tag}`) in backticks and say what `latest` points to.
- Env tables use the columns `Variable | Default | Description`, with names and defaults in backticks.
- An env table is its own `## Environment` section, never part of `## Usage`, and exists only when the image
  (its entrypoint or the upstream program) reads the variables. An image configured by a mounted file has no
  env table.
- Keep tables in sync with the folders and the Makefile `VERSION` list.

## Screenshots

- A README shows at most one screenshot in the header. More screens go to a topic section after
  `## Environment`.
- A gallery of different screens has one `###` heading per screen, a one-sentence lead-in that says what the
  screen does, then the image. Tables are not used for this ([TONE.md](TONE.md#formatting)).
- Variants of one screen (themes, sizes) may be a grid of thumbnails in HTML, `width="200"`, with the variant name
  as the caption.
- Every image in `.docs/` is listed with its date in `DESIGN.md` when the repository has one
  ([DESIGN.md](DESIGN.md#screenshots)). An image that shows an old name or version is retaken or removed.

## LICENSE

- MIT License, file named `LICENSE` (no extension).
- Holder is `Dockette`: `Copyright (c) {year} Dockette`.
- The year is the year the repository was created. Don't bump it.
- Take the year from the first commit (`git log --reverse --format=%as | head -1`). A shallow clone doesn't have
  it: fetch the full history, or read the creation date on the GitHub repository page. Never copy the year from
  another repository.
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
- The README shows how to start it with one `docker compose up` line and links the file; it doesn't paste it.

## Agent Files

Every repository has agent instructions:

- `AGENTS.md` holds the instructions. Its content is described in [AGENTS.md](AGENTS.md).
- `AGENTS.md` is the only agent file; no `CLAUDE.md`.
- `fxnorm.yml`, written by `fxnorm init`, sets the checks for the repository (see
  [AGENTS.md](AGENTS.md#checking-with-fxnorm)). `fxnorm check` runs the rules that enforce this document.

## Checklist

- [ ] `README.md`, `LICENSE`, `.editorconfig` and `Makefile` exist in the root
- [ ] README header is `<h1 align=center>Dockette / {Name}</h1>`
- [ ] Badge row: GitHub Actions, Docker Hub pulls, GitHub Sponsors, Discussions (shields.io, with `alt`)
- [ ] No Gitter, badgen.net or Docker Hub stars badges
- [ ] Description of one to three sentences: what is inside, what it is based on, who it is for
- [ ] `## Usage` has one `docker run` with the essential argument and the untagged image, a short note and a docs
  link; one or two code blocks, no ports, container names, pinned tags, config samples or variants
- [ ] Tags are in the `## Versions` table; env variables are in `## Environment` only when the image reads them
- [ ] `## Development` is 3 to 5 `make` commands with comments, then "Run `make` to list every target."
- [ ] README ends with `## Maintenance` and the standard footer ("Consider supporting …")
- [ ] Text follows [TONE.md](TONE.md)
- [ ] LICENSE is MIT, `Copyright (c) {year} Dockette`
- [ ] `.editorconfig` globs have no spaces and `Makefile` uses tabs
- [ ] `.env` is ignored and `.env.dist` is committed when env variables are used
- [ ] `docker-compose.yml` has no `version:` key
- [ ] README images are in `.docs/` (no `assets/` level); galleries use one `###` per screen
- [ ] `AGENTS.md` exists, there is no `CLAUDE.md` and `fxnorm.yml` is committed

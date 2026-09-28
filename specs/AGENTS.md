# Dockette AGENTS.md Specification

This document describes how `AGENTS.md`, `CLAUDE.md` and the optional `PRD.md`, `TECH.md` and `DESIGN.md` files are
written in Dockette repositories. `AGENTS.md` tells an AI coding agent what it can't learn quickly from the
Dockerfile: how to build and test the image, which tags exist and where the traps are. How to write the text is
described in [TONE.md](TONE.md). Commands come from [MAKEFILE.md](MAKEFILE.md), CI from [WORKFLOWS.md](WORKFLOWS.md)
and the tag lifecycle from [IMAGES.md](IMAGES.md).

## Table of Contents

- [Rules](#rules)
- [Files](#files)
- [Sections](#sections)
- [Writing Bullets](#writing-bullets)
- [Single Image Template](#single-image-template)
- [Multi Version Template](#multi-version-template)
- [Project Documents](#project-documents)
- [What Not to Include](#what-not-to-include)
- [Checklist](#checklist)

## Rules

- Every repository has an `AGENTS.md` in the root.
- `CLAUDE.md` contains exactly one line: `@AGENTS.md`. All instructions live in `AGENTS.md`.
- `AGENTS.md` is 50 to 100 lines. It is a signpost, not a manual. Longer content moves to `TECH.md` in stacks and
  workspaces, and to Dockerfile comments elsewhere.
- Every line is specific to the repository. If a line would be true for every Dockette image, it belongs in these
  specs, not in `AGENTS.md`.
- Bullets state a fact first, then the rule that follows from it, then the reason or the file to read.
- Link, don't repeat. Usage for image users stays in `README.md`, organization rules stay in these specs.
- The text is vendor neutral: "AI coding agent", not the name of one tool.
- Plain Markdown: `##` sections, bullets, one fenced command block, inline code for every file, variable, tag and
  command. No tables, no emoji, no headings below `###`.
- Agent files never end up in an image. When a Dockerfile copies the whole build context, `.dockerignore` excludes
  them (see [.dockerignore](#dockerignore)).
- A pull request that adds or removes a tag, changes a `make` target, the base image or a documented trap updates
  `AGENTS.md` in the same pull request.

## Files

| File | Required | Content |
|------|----------|---------|
| `AGENTS.md` | yes | Overview, commands, conventions, traps |
| `CLAUDE.md` | yes | The single line `@AGENTS.md` |
| `PRD.md` | stacks, workspaces, republishes with our own layer | What it is for and what is out of scope, see [PRD.md](PRD.md) |
| `TECH.md` | same as `PRD.md` | How it is built and started, and why, see [TECH.md](TECH.md) |
| `DESIGN.md` | images with a UI we customize or write | How the UI looks and behaves, see [DESIGN.md](DESIGN.md) |
| `.claude/` | no | Shared agent settings. `settings.local.json` is never committed |

- All files are in the root and use uppercase names.
- Don't add other agent files (`.cursorrules`, `.github/copilot-instructions.md`, `GEMINI.md`, `llms.txt`).

### .dockerignore

The [.dockerignore template](DOCKERFILE.md#dockerignore) already excludes `*.md`, which covers `AGENTS.md`,
`CLAUDE.md` and the project documents. Add the agent settings folder:

```
.claude
```

## Sections

Sections, in this order. Headings use the exact names below.

| Section | Required | Content |
|---------|----------|---------|
| `# {Title}` + purpose line | yes | `Dockette / {Name}` from the README, then one fixed sentence |
| `## Overview` | yes | One paragraph: what the image is and what it is not. Key facts as bullets |
| `## Documentation` | yes | Where to read before which change |
| `## Commands` | yes | One `bash` block with the `make` targets |
| `## Conventions` | yes | 2 to 5 bullets, only what differs from the specs or needs a reminder |
| `## Traps` | yes | 3 to 10 bullets with non-obvious invariants. The last bullet sets the scope |
| `## Ground Rules` | services and stacks | Security and runtime invariants (user, ports, secrets) |

- The purpose line is always: `Instructions for AI coding agents working in this repository.`
- `## Overview` names the [image class](IMAGES.md#image-classes), the base image, the published tags with what
  `latest` points to, and the platforms.
- `## Commands` uses the [target names](MAKEFILE.md#target-names) and the variables `DOCKER_IMAGE`, `DOCKER_TAG`,
  `DOCKER_PLATFORMS` and `VERSION`. It says which command CI runs.
- `## Traps` is the most useful section. When an image has nothing surprising, keep it short, but don't fill it
  with general advice.
- The last bullet of `## Traps` says what is not in the file and where it is: "Usage for image users (ports,
  volumes, environment variables) lives in `README.md`, not here."

## Writing Bullets

Each trap opens with a **bold claim** in one sentence. The rule and the reason follow in one or two sentences.

Good:

```markdown
- **Every version folder is a full copy.** There is no shared template; a change for all versions is made in
  every folder, then `make test-all` checks them.
```

Bad:

```markdown
- Keep the image small.
- Test your changes before pushing.
```

- Correct likely wrong assumptions directly: "PHP comes from `packages.sury.org`, not from the official `php`
  image."
- Name the file to read: "See `entrypoint.sh` before changing an environment variable."
- Say which tags a change affects, and whether `latest` moves.

## Single Image Template

A filled example for `dockette/pgbouncer`, a thin republish. Replace every fact with the repository's own. About
50 lines is typical.

````markdown
# Dockette / PgBouncer

Instructions for AI coding agents working in this repository.

## Overview

`dockette/pgbouncer` republishes `dhi.io/pgbouncer` under the Dockette name on Docker Hub. It is a thin
republish (see IMAGES.md): the Dockerfile only sets `FROM` and labels. It adds no config and no entrypoint.

- **Image**: `dockette/pgbouncer`, tags `1.26.0` and `latest` (same image)
- **Base**: `dhi.io/pgbouncer:${PGBOUNCER_VERSION}`
- **Platforms**: `linux/amd64`

## Documentation

- `README.md` is also the Docker Hub description; CI publishes it from `master`.
- Organization rules are in [dockette/dockette specs](https://github.com/dockette/dockette/tree/master/specs).

## Commands

```bash
# Build the image for the default tag, or for another upstream version
make build
make build DOCKER_TAG=1.25.1

# Smoke test (pgbouncer --version)
make test

# Run on port 6432 with a local config
make run PGBOUNCER_CONFIG=$(pwd)/pgbouncer.ini
```

CI runs `make build` and `make test`, then the reusable workflow builds and pushes from `master`.

## Conventions

- The image tag is the upstream version. `DOCKER_TAG` is passed to the build as `PGBOUNCER_VERSION`.
- Labels follow IMAGES.md; `org.opencontainers.image.version` comes from `PGBOUNCER_VERSION`.
- The `Dockerfile` holds only `ARG`, `FROM` and `LABEL`. Anything more makes it a build image, not a
  republish.

## Traps

- **A version bump touches four places:** `DOCKER_TAG` in the `Makefile`, the tag in the workflow, the
  `Dockerfile` default `ARG` and the README Versions table. Missing one publishes a tag that the README doesn't list.
- **`PGBOUNCER_CONFIG` must be an absolute path.** Docker reads a relative `-v` source as a named volume and
  mounts an empty directory instead of the file.
- **The weekly rebuild pulls the same pinned upstream tag.** A new upstream release is not picked up on its own;
  bump the version.
- **Old version tags stay on Docker Hub.** Don't delete them after a bump; users pin them (IMAGES.md, Tag Naming).
- **`linux/arm64` is not built.** Check that upstream publishes it before adding it to `DOCKER_PLATFORMS`.
- Usage for image users (config file, ports, userlist) lives in `README.md`, not here.
````

Single image specifics:

- For a service image, add `## Ground Rules`: the user it runs as, the ports it listens on, the `HEALTHCHECK`, and
  where secrets come from.
- For a republish, say what upstream is and that behaviour must match it.

## Multi Version Template

A filled example for `dockette/php`, one folder per tag. About 60 lines is typical.

````markdown
# Dockette / PHP

Instructions for AI coding agents working in this repository.

## Overview

`dockette/php` builds Debian based PHP images with CLI or FPM and Composer. It is a runtime image (see
IMAGES.md), the base for `dockette/deploy` and other tools. It ships no application code.

- **Image**: `dockette/php`, tags `5.6` to `8.5`, each also as `-fpm`; `latest` points to the newest CLI tag
- **Base**: `dockette/debian:bookworm`, PHP packages from `packages.sury.org`
- **Platforms**: `linux/amd64`, `linux/arm64`
- **Layout**: one folder per tag (`8.5/`, `8.5-fpm/`), each with its own `Dockerfile` and `conf.d/`

## Documentation

- `README.md` lists every tag with its state (Supported, Legacy, Frozen) and is the Docker Hub description.
- Supported versions and the deprecation steps are in
  [IMAGES.md](https://github.com/dockette/dockette/blob/master/specs/IMAGES.md).

## Commands

```bash
# Build and test the latest version, or one tag
make build
make test
make build VERSION=8.4-fpm
make test VERSION=8.4-fpm

# Build and test every tag (slow)
make build-all
make test-all

# Run the latest CLI image with the current folder mounted in /srv
make run
```

CI runs `make build` and `make test` for each tag in the matrix, in parallel, with `fail-fast: false`.

## Conventions

- Binaries are versioned: `php8.5`, `php-fpm8.5`. Config goes to `conf.d/custom.ini`, which is linked as
  `999-custom.ini` into the CLI, CGI and FPM config folders.
- A new PHP version is a new folder pair, a `VERSION` entry in the `Makefile`, a matrix entry and a README row.
  `latest` moves to it in the same pull request.

## Traps

- **Every version folder is a full copy.** There is no shared template; a change for all versions is made in
  every folder, then `make test-all` checks them.
- **PHP comes from `packages.sury.org`, not from the official `php` image.** Extension names are Debian packages
  (`php8.5-intl`), not `docker-php-ext-install`.
- **`pcov` is required from PHP 7.1 up.** `make test` fails when it is missing; older tags are expected not to
  have it.
- **`5.6` to `8.1` are Legacy.** They build while Bookworm is supported and are marked EOL in the README. Don't
  add features to them; fix only what breaks the build.
- **Child images depend on these tags.** After changing a base tag, trigger `dockette/deploy` by hand.
- Usage for image users (volumes, FPM setup, Composer) lives in `README.md`, not here.
````

Multi version specifics:

- `## Overview` lists the tag range and the `latest` tag instead of every tag. The full list is in the README.
- Name the lifecycle state of old tags from [IMAGES.md](IMAGES.md#lifecycle-states) and say what may change in
  them.
- When the folders share files (entrypoint, config), say whether they are copies or links, and how to keep them
  in sync.

## Project Documents

`PRD.md`, `TECH.md` and `DESIGN.md` hold knowledge that is too long for `AGENTS.md`. Their content is described in
[PRD.md](PRD.md), [TECH.md](TECH.md) and [DESIGN.md](DESIGN.md). In short:

- `PRD.md` and `TECH.md`: stacks, workspaces and republishes with our own layer
  ([when required](PRD.md#when-it-is-required)).
- `DESIGN.md`: services with a browser UI we customize or pages we write
  ([when required](DESIGN.md#when-it-is-required)).
- Base, runtime, tool and plain republish images have none of them. Their build notes stay in `AGENTS.md` and
  Dockerfile comments.

`AGENTS.md` links each existing document from `## Documentation`, with one line that says when to read it:

```markdown
- `TECH.md` explains the build stages and the entrypoint order. Read it before changing `Dockerfile` or
  `entrypoint.sh`.
```

Don't repeat their content in `AGENTS.md`. A trap that is explained in `TECH.md` gets one bullet with a link.

## What Not to Include

- Generic advice: "keep images small", "use multi-stage builds", "test your changes".
- Rules already in these specs, beyond one bullet with a link.
- The Versions or Environment tables from the README. Link to them instead.
- A copy of the Makefile or the workflow.
- Machine-local paths, personal preferences, Docker Hub tokens or other credentials.
- Text addressed to one tool ("This file provides guidance to …").
- Plans, TODO lists, status notes and references to work in progress. Describe the current state.

## Checklist

- [ ] `AGENTS.md` exists in the root and has 50 to 100 lines
- [ ] `CLAUDE.md` contains only `@AGENTS.md`
- [ ] Sections in order: title and purpose line, Overview, Documentation, Commands, Conventions, Traps,
      (Ground Rules)
- [ ] Overview states the image class, base image, tags with `latest`, platforms and, for multi version repos,
      the folder layout
- [ ] Commands use `make build`, `make test`, `make run` (and `VERSION=` or `build-all` for multi version repos)
      and say what CI runs
- [ ] Every trap opens with a bold claim and says why or where to read more
- [ ] The last trap bullet points to `README.md`
- [ ] No generic advice, no emoji, no tables, no tool-specific wording
- [ ] `PRD.md`, `TECH.md` and `DESIGN.md` exist where [Project Documents](#project-documents) requires them and are
      linked from `## Documentation`
- [ ] Agent files are excluded from the build context when the Dockerfile copies it
- [ ] `.claude/settings.local.json` is not committed

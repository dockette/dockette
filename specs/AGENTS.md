# Dockette AGENTS.md Specification

This document describes how `AGENTS.md` and `CLAUDE.md` are written in Dockette repositories. `AGENTS.md` is a
short development guide for AI coding agents and people: the stack, the commands and the principles. Usage for
image users lives in `README.md`, organization rules in these specs. How to write the text is described in
[TONE.md](TONE.md); commands come from [MAKEFILE.md](MAKEFILE.md).

## Table of Contents

- [Rules](#rules)
- [Sections](#sections)
- [Single Image Template](#single-image-template)
- [Multi Version Template](#multi-version-template)
- [Checking with fxnorm](#checking-with-fxnorm)
- [Checklist](#checklist)

## Rules

- Every repository has an `AGENTS.md` in the root.
- `AGENTS.md` has 20 to 45 lines. Plain English, no emoji, no tables.
- Sections exactly: title and one-line purpose, `## Stack`, `## Development`, `## Principles`.
- Only facts that are true for the repository: every command must exist.
- It stays high level. It does not describe the file or folder structure, architecture internals, traps, history,
  planned changes, TODOs or "what is changing".
- `CLAUDE.md` contains exactly one line: `@AGENTS.md`.
- Don't add other agent files (`.cursorrules`, `.github/copilot-instructions.md`, `GEMINI.md`, `llms.txt`).
- `AGENTS.md` doesn't link `PRD.md`, `TECH.md` or `DESIGN.md`. The README links them when they exist.
- Agent files never end up in an image. The [.dockerignore template](DOCKERFILE.md#dockerignore) excludes `*.md`;
  add the agent settings folder and the fxnorm config when the Dockerfile copies the whole build context:

```
.claude
fxnorm.yml
```

## Sections

- **Title and purpose**: `# Dockette / {Name}` from the README, then one sentence saying what the image provides.
- **`## Stack`**: how the image is built, the base image, the main software and its version, and where it is
  published. Versions come from the `Dockerfile`, the `Makefile` and `.github/workflows/docker.yml`.
- **`## Development`**: one fenced `bash` block with the `make` targets that exist in the `Makefile`. When the
  `Makefile` has no `test` target, write the command that tests the image today (from `docker.yml`) and add
  `test` in the next change to the `Makefile` ([MAKEFILE.md](MAKEFILE.md#target-names)).
- **`## Principles`**: KISS, DRY and YAGNI, plus at most two bullets on image rules from
  [DOCKERFILE.md](DOCKERFILE.md).

## Single Image Template

`{...}` marks a placeholder. Replace it with facts from the repository and delete lines that don't apply.

````markdown
# Dockette / {Name}

{One sentence: what the image provides.}

## Stack

- Docker image built with `docker buildx`, base {debian:trixie-slim / alpine:3.23 / upstream image}
- {Main software and version, e.g. PgBouncer 1.26}
- Published to Docker Hub as `dockette/{name}` by GitHub Actions

## Development

```bash
make build       # build the image
make test        # smoke test the image
make run         # run it locally
```

Run `make` to list every target.

## Principles

- KISS: one image does one job; no extra services or tools.
- DRY: shared steps live in the base image, not copied into every Dockerfile.
- YAGNI: add a package only when the image needs it.
- Pin versions, keep layers small, clean package caches in the same `RUN`.
- Every change is built and smoke tested with `make build test` before a commit.
````

## Multi Version Template

Repositories with one folder per tag (`php`, `debian`, `nodejs`, `postgres`, ...) add the `VERSION` line.

````markdown
# Dockette / {Name}

{One sentence: what the image provides.}

## Stack

- Docker image built with `docker buildx`, base {debian:trixie-slim / alpine:3.23 / upstream image}
- {Main software and versions, e.g. PHP 8.2 to 8.5 with Composer}
- Published to Docker Hub as `dockette/{name}` by GitHub Actions

## Development

```bash
make build       # build the image
make test        # smoke test the image
make run         # run it locally
```

`make build VERSION={8.4}` builds one version. Run `make` to list every target.

## Principles

- KISS: one image does one job; no extra services or tools.
- DRY: shared steps live in the base image, not copied into every Dockerfile.
- YAGNI: add a package only when the image needs it.
- Pin versions, keep layers small, clean package caches in the same `RUN`.
- Every change is built and smoke tested with `make build test` before a commit.
````

## Checking with fxnorm

`fxnorm` checks a repository against these specs and reports each deviation with the file, the line and the id of
the rule that found it. Set it up once per repository, then check after every change:

```bash
# Detect the kind of repository and write fxnorm.yml (preset dockette-image)
fxnorm init

# Report deviations, or apply the safe fixes and report what is left
fxnorm check
fxnorm fix
```

- `fxnorm.yml` is committed in the root. A setting that turns a rule off or lowers its severity has a comment
  with the reason.
- `fxnorm fix` writes `CLAUDE.md` (`common/claude-md-import`) and the Makefile help block. Everything else is
  fixed by hand.
- The rules for this document are `common/agents-md-exists`, `common/agents-md-length` (20 to 45 lines),
  `common/agents-md-sections`, `common/agents-md-no-structure`, `common/agents-md-no-emoji`,
  `common/claude-md-import` and `common/tone-words`.
- Fix the file instead of silencing the rule. A finding you accept gets `<!-- fxnorm:ignore {rule id} -->` on the
  line above it, with the reason in the same comment.
- `fxnorm explain {rule id}` shows what a rule checks. When a rule and these specs disagree, the specs win; report
  the rule.
- `fxnorm.yml`, `AGENTS.md` and `CLAUDE.md` never end up in an image. In a Contributte library the same files are
  export-ignored in `.gitattributes`.
- `AGENTS.md` doesn't list `fxnorm` in `## Development`. It is the same in every repository.

## Checklist

- [ ] `AGENTS.md` exists in the root and has 20 to 45 lines
- [ ] Sections in order: title and purpose, `## Stack`, `## Development`, `## Principles`
- [ ] Every command in `## Development` exists in the `Makefile` today (`VERSION=` only for multi version repos)
- [ ] No folder structure, architecture, traps, history, plans or TODOs
- [ ] No emoji, no tables, no placeholder left
- [ ] `CLAUDE.md` contains only `@AGENTS.md`
- [ ] Agent files are excluded from the build context when the Dockerfile copies it
- [ ] `fxnorm check` reports no findings in `AGENTS.md` and `CLAUDE.md`

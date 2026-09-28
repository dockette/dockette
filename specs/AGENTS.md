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
- [Placeholders](#placeholders)
- [Commands and CI](#commands-and-ci)
- [Single Image Template](#single-image-template)
- [Multi Version Template](#multi-version-template)
- [Project Documents](#project-documents)
- [What Not to Include](#what-not-to-include)
- [Checking with fxnorm](#checking-with-fxnorm)
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
| `PRD.md` | stacks, workspaces, republishes with our own layer; services per [PRD.md](PRD.md#when-it-is-required) | What it is for and what is out of scope, see [PRD.md](PRD.md) |
| `TECH.md` | same as `PRD.md` | How it is built and started, and why, see [TECH.md](TECH.md) |
| `DESIGN.md` | images with a UI we customize, rebrand or write | How the UI looks and behaves, see [DESIGN.md](DESIGN.md) |
| `.claude/` | no | Shared agent settings. `settings.local.json` is never committed |
| `fxnorm.yml` | yes | The fxnorm preset and rule settings, written by `fxnorm init`, see [Checking with fxnorm](#checking-with-fxnorm) |

- All files are in the root. The agent and project documents use uppercase names; `fxnorm.yml` is lowercase.
- [PRD.md](PRD.md#when-it-is-required) decides which repositories need `PRD.md` and `TECH.md`. When the task
  asks for them in a repository that doesn't need them, write them anyway; they then follow the same specs.
- Don't add other agent files (`.cursorrules`, `.github/copilot-instructions.md`, `GEMINI.md`, `llms.txt`).

### .dockerignore

The [.dockerignore template](DOCKERFILE.md#dockerignore) already excludes `*.md`, which covers `AGENTS.md`,
`CLAUDE.md` and the project documents. Add the agent settings folder and the fxnorm config:

```
.claude
fxnorm.yml
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
  `DOCKER_PLATFORMS` and `VERSION`. It says which command CI runs. What to write when a target is missing or CI
  doesn't call `make` is in [Commands and CI](#commands-and-ci).
- `## Traps` is the most useful section. When an image has nothing surprising, keep it short, but don't fill it
  with general advice.
- The last bullet of `## Traps` says what is not in the file and where it is: "Usage for image users (ports,
  volumes, environment variables) lives in `README.md`, not here." It stays the last bullet of `## Traps` when
  `## Ground Rules` follows; it closes the traps, not the file.

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

## Placeholders

The templates below are outlines, not examples to copy. Fixed wording is written out; everything in `{...}` is a
placeholder with a hint of what goes there.

- Replace every placeholder with facts from the repository: the `Dockerfile`, the `Makefile`,
  `.github/workflows/docker.yml`, the entrypoint and the README. Check each fact in the file named in the hint.
- Delete lines that don't apply. A republish has no entrypoint trap; a single image has no `VERSION=` command.
- Never keep a fact because it is in the template. A target, variable, platform or workflow step the repository
  doesn't have is a bug in `AGENTS.md`.
- The finished file has no placeholders left. Braces that belong to the content (`${VAR}`, Go templates) are not
  placeholders; a placeholder always holds a hint in plain words.

## Commands and CI

- `## Commands` lists only targets that exist in the `Makefile` today. Never write a target the repository doesn't
  have, even when the specs require it.
- **No `make test`.** Every repository must have a `test` target ([MAKEFILE.md](MAKEFILE.md#target-names); fxnorm
  reports `dockette/makefile-targets`). Until the `Makefile` has one, `## Commands` lists the targets that exist
  and, under `# Smoke test`, the command that tests the image today (the steps from `docker.yml`, or a
  `docker run ... --version`). A trap says it: "**There is no `make test`.** CI runs its own smoke test in
  `.github/workflows/docker.yml`; run the same `curl` loop by hand." When the pull request touches the `Makefile`,
  add `test` there and use it instead.
- Other missing targets (`help`, `push`, `run`) are handled the same way: real names in `## Commands`, the gap as
  a trap only when it surprises (plain `make` builds instead of printing help).
- The line under the command block says what CI runs, read from `docker.yml`. When CI doesn't call `make`, say so
  and name what it runs: "CI doesn't call `make`: `docker.yml` builds with `docker/build-push-action` and runs
  `curl` against port 80."
- An image without any smoke test says so in that line ("Nothing tests the image; CI only builds it.") and adds
  `test` to the `Makefile` in the same pull request when it can.

## Single Image Template

About 50 lines is typical when filled. `{...}` marks a placeholder: replace it with facts from the repository;
delete lines that don't apply (see [Placeholders](#placeholders)).

````markdown
# Dockette / {Name from the README header}

Instructions for AI coding agents working in this repository.

## Overview

`dockette/{name}` {what the image is, one sentence: what runs in it and what it is based on}. It is a
{image class from IMAGES.md, e.g. thin republish}: {what the Dockerfile adds, e.g. only `FROM` and labels}.
{What it doesn't add or do.}

- **Image**: `dockette/{name}`, tags {tags from docker.yml} (`latest` {what it points to})
- **Base**: `{FROM line, with the ARG it uses}`
- **Platforms**: `{platforms docker.yml builds}`

## Documentation

- `README.md` is also the Docker Hub description. {Whether docker.yml publishes it from `master`.}
- {`DESIGN.md`, `PRD.md`, `TECH.md` when they exist, with one line on when to read each.}
- Organization rules are in [dockette/dockette specs](https://github.com/dockette/dockette/tree/master/specs).

## Commands

```bash
# Build the image for the default tag, or for another upstream version
make build
make build DOCKER_TAG={an upstream tag that exists}

# Smoke test ({what the test target runs})
make test

# Run {on which port, with which mount or variable}
make run {VARIABLE=value the target reads}
```

{What CI runs, from docker.yml; say when it doesn't call make.}

## Conventions

- {How the image tag relates to the upstream version and which variable carries it.}
- {Labels and where their values come from.}
- {What the Dockerfile may contain, e.g. only `ARG`, `FROM` and `LABEL` for a republish.}

## Traps

- **{A version bump touches N places:}** {every place, named as it is in the files, e.g. `DOCKER_TAG` in the
  `Makefile`, `env.{NAME}_VERSION` in `docker.yml`, the `Dockerfile` default `ARG`, the README Versions table}.
- **{Bold claim about a variable, mount or port that fails silently}.** {What happens and why.}
- **{Bold claim about what the weekly rebuild does and doesn't pick up}.** {What to do instead.}
- **{Bold claim about platforms}.** {Why a platform is missing and what to check before adding it.}
- Usage for image users ({what the README covers}) lives in `README.md`, not here.
````

Single image specifics:

- For a service image, add `## Ground Rules`: the user it runs as, the ports it listens on, the `HEALTHCHECK`, and
  where secrets come from.
- For a republish, say what upstream is and that behaviour must match it.
- The traps in [Writing Bullets](#writing-bullets) show the tone. Don't copy them into another repository.

## Multi Version Template

About 60 lines is typical when filled. `{...}` marks a placeholder: replace it with facts from the repository;
delete lines that don't apply (see [Placeholders](#placeholders)).

````markdown
# Dockette / {Name from the README header}

Instructions for AI coding agents working in this repository.

## Overview

`dockette/{name}` builds {what, based on what}. It is a {image class from IMAGES.md}, {the images built on it}.
{What it doesn't ship.}

- **Image**: `dockette/{name}`, tags {range, from the folders}; `latest` points to {tag}
- **Base**: `{FROM line}`, {where the packages come from}
- **Platforms**: `{platforms docker.yml builds}`
- **Layout**: {one folder per tag, and what each folder holds}

## Documentation

- `README.md` lists every tag {with its lifecycle state} and is the Docker Hub description.
- Supported versions and the deprecation steps are in
  [IMAGES.md](https://github.com/dockette/dockette/blob/master/specs/IMAGES.md).

## Commands

```bash
# Build and test the latest version, or one tag
make build
make test
make build VERSION={a real tag}
make test VERSION={a real tag}

# Build and test every tag (slow)
make build-all
make test-all

# Run {what, with which mount}
make run
```

{What CI runs, from docker.yml: the matrix and whether it calls make.}

## Conventions

- {Naming inside the image: binaries, config paths.}
- {What a new version needs: folder, `VERSION` entry, matrix entry, README row, where `latest` moves.}

## Traps

- **{Bold claim about how the folders relate: full copies, a shared folder or links}.** {How a change reaches
  every version and which command checks it.}
- **{Bold claim correcting a likely wrong assumption about where packages come from}.** {The consequence.}
- **{Bold claim about lifecycle states of old tags}.** {What may change in them.}
- **{Bold claim about images that depend on these tags}.** {What to trigger after a change.}
- Usage for image users ({what the README covers}) lives in `README.md`, not here.
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

- `PRD.md` and `TECH.md`: stacks, workspaces, republishes with our own layer, and services whose README needs
  more than one Usage section ([when required](PRD.md#when-it-is-required)).
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

- `fxnorm.yml` is committed in the root. It names the preset and, when needed, rule settings. A setting that
  turns a rule off or lowers its severity has a comment with the reason.
- `fxnorm fix` writes `CLAUDE.md` (`common/claude-md-import`) and the Makefile help block. Everything else is
  fixed by hand.
- The rules for this document are `common/agents-md-exists`, `common/agents-md-length` (50 to 100 lines),
  `common/agents-md-no-emoji`, `common/claude-md-import` and `common/tone-words` (`README.md` and `AGENTS.md`).
  `dockette/design-md-exists` and `dockette/prd-tech-exist` check the project documents.
- Fix the file instead of silencing the rule. A finding you accept gets `<!-- fxnorm:ignore {rule id} -->` on the
  line above it, with the reason in the same comment.
- `fxnorm explain {rule id}` shows what a rule checks and which section of these specs it enforces. When a rule
  and these specs disagree, the specs win; report the rule.
- `fxnorm.yml`, `AGENTS.md` and `CLAUDE.md` never end up in an image: `.dockerignore` excludes them when the
  Dockerfile copies the whole context (see [.dockerignore](#dockerignore)). In a Contributte library the same
  files are export-ignored in `.gitattributes`.
- `AGENTS.md` doesn't list `fxnorm` in `## Commands`. It is the same in every repository and belongs in these
  specs.

## Checklist

- [ ] `AGENTS.md` exists in the root and has 50 to 100 lines
- [ ] `CLAUDE.md` contains only `@AGENTS.md`
- [ ] Sections in order: title and purpose line, Overview, Documentation, Commands, Conventions, Traps,
      (Ground Rules)
- [ ] Overview states the image class, base image, tags with `latest`, platforms and, for multi version repos,
      the folder layout
- [ ] Every fact comes from the repository; no placeholder and no template fact is left
- [ ] Commands exist in the `Makefile` today: `make build`, `make test`, `make run` (and `VERSION=` or `build-all`
      for multi version repos); a missing `test` is replaced by the real smoke test and named as a trap
- [ ] The line under the commands says what CI runs, including when it doesn't call `make`
- [ ] Every trap opens with a bold claim and says why or where to read more
- [ ] The last trap bullet points to `README.md`
- [ ] No generic advice, no emoji, no tables, no tool-specific wording
- [ ] `PRD.md`, `TECH.md` and `DESIGN.md` exist where [Project Documents](#project-documents) requires them and are
      linked from `## Documentation`
- [ ] Agent files are excluded from the build context when the Dockerfile copies it
- [ ] `.claude/settings.local.json` is not committed
- [ ] `fxnorm check` reports no findings in `AGENTS.md` and `CLAUDE.md`

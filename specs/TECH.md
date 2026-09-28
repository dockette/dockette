# Dockette TECH.md Specification

This document describes the `TECH.md` file in Dockette repositories that have a `PRD.md`: stacks, workspaces and
republished images with our own layer. The file is the technical design: build stages, what runs at start,
configuration, data, services, testing and the decisions behind them. Product intent is in [PRD.md](PRD.md), UI in
[DESIGN.md](DESIGN.md), Dockerfile rules in [DOCKERFILE.md](DOCKERFILE.md), image lifecycle in
[IMAGES.md](IMAGES.md), CI in [WORKFLOWS.md](WORKFLOWS.md).

## Table of Contents

- [Rules](#rules)
- [When It Is Required](#when-it-is-required)
- [Sections](#sections)
- [Decisions](#decisions)
- [Undated Decisions](#undated-decisions)
- [Writing Style](#writing-style)
- [Template](#template)
- [Checklist](#checklist)

## Rules

- `TECH.md` lives in the repository root, next to `README.md`, `PRD.md` and `AGENTS.md`.
- It is 50 to 150 lines. Detail about one part goes to `.docs/` and is linked from here.
- It describes the image as it is built today, not a plan.
- It links instead of repeating: variables are in the README tables and `.env.dist`, commands in the `Makefile`
  and `AGENTS.md`, tags in the Versions table.
- Versions come from `ENV *_VERSION` lines, `FROM` lines and Compose `image:` keys. When those change, `TECH.md`
  changes in the same pull request.
- A pull request that adds a build stage, a sidecar process, a service or reverses a decision adds a dated entry
  to `## Decisions`.
- It is never copied into the image.

## When It Is Required

`TECH.md` is required in the same repositories as `PRD.md` (see [PRD.md](PRD.md#when-it-is-required)). Other
images keep their build notes in `AGENTS.md` and the Dockerfile comments.

## Sections

Use these `##` sections in this order:

1. `# {Name} Tech` and one sentence: what runs in the container or stack.
2. `## Architecture` - a text diagram of build stages and runtime processes, then 2 to 4 bullets.
3. `## Stack` - base image, upstream application and every added binary, with versions.
4. `## Layout` - the repository tree and where each file lands in the image.
5. `## Configuration` - variables and defaults, how config files are rendered, link to `.env.dist`.
6. `## Data` - volumes, what is stored where, backup and restore.
7. `## Services` - Compose services, ports, networks, and what talks to what. For a single image: exposed
   ports and required external services.
8. `## Startup Flow` - what the entrypoint does, in order, until it `exec`s the main process.
9. `## Build and Publish` - platforms, CI matrix, tags, schedule.
10. `## Testing` - what `make test` and CI check, and what is only checked by hand.
11. `## Decisions` - dated entries, newest first.
12. `## Known Limits` - what is wrong or outdated today and why it is not fixed yet.

## Decisions

Each entry is short and never rewritten after it is merged. A later decision that reverses it adds a new entry
and marks the old one `Superseded by YYYY-MM-DD`.

```markdown
### 2026-09-28 Litestream as a wrapper process

- **Context:** Uptime Kuma keeps state in SQLite; a lost volume loses all monitors and history.
- **Decision:** Run Uptime Kuma under `litestream replicate -exec` when `LITESTREAM=1`.
- **Consequences:** (+) continuous S3 backup, restore on start; (-) one more binary to pin and update.
- **Rejected:** Cron `sqlite3 .backup` - loses up to one interval of data.
```

When the list passes about 10 entries or an entry needs more than 10 lines, move entries to
`.docs/decisions/YYYY-MM-DD-slug.md` with the same headings and keep a one-line index here.

### Undated Decisions

A `TECH.md` written for an existing image records decisions that were made before the file existed. Their date is
often unknown.

- Take the date from git: the commit that introduced the change (`git log --diff-filter=A --format=%as -- {file}`
  for a new file, `git log -S '{text}' --format=%as` for a line in the `Dockerfile`). Write it as the entry date.
- When git can't tell (a shallow clone, a squashed import), use the date the entry is written and add
  `(recorded)` after the title: `### 2026-09-28 Caddy as the static file server (recorded)`. The marker says the
  decision is older than the date.
- Never invent a date and never leave the heading without one. Sorting and `Superseded by` both need it.
- `(recorded)` entries keep the Context line short and say what is known: "Chosen before the first tag; no
  discussion is recorded."

## Writing Style

- Facts first, reason second: "**The entrypoint must `exec`.** Otherwise signals stop at the shell."
- Code spans for every file, variable, tag and path inside the image.
- Versions exactly as pinned (`1.23.16-debian`), not "latest".
- No emoji, no marketing words, no future tense except in `Known Limits`.

## Template

`{...}` marks a placeholder: replace it with facts from the repository; delete lines that don't apply. Text
outside braces is the fixed structure. Every version, variable, port and path comes from the `Dockerfile`, the
entrypoint, `.env.dist`, the Compose file and `docker.yml`, not from the template.

````markdown
# {Name} Tech

{What runs in the container or stack, in one sentence.}

## Architecture

```
build:  {stage base} -> {what the stage produces} --+--> {final base image and tag}
        {files copied from the repository}         --+
run:    {entrypoint} -> {condition} {what it starts}
                     -> {otherwise} {main process}
```

- {How many processes run and which one is PID 1.}
- {Where state lives, or that there is none.}

## Stack

- Upstream: `{image:tag}` ({runtime, port}).
- {Every added binary with its pinned version and where it comes from.}

## Layout

- `{file in the repository}` -> `{path in the image}`, {what renders it, if anything}.

## Configuration

- {Variables with their defaults, where the defaults are set, and the link to `.env.dist`.}
- {How config files are rendered at start.}

## Data

- {Volumes and what is stored in each; what is lost when a volume is lost.}

## Services

- {Ports, and the external services it needs.}

## Startup Flow

1. {What the entrypoint does first.} 2. {Next step.} 3. `exec` {the main process}.

## Build and Publish

- {Tags, platforms, context and schedule from docker.yml; whether CI calls make.}

## Testing

- {What `make test` checks, or what CI checks when there is no `make test`.}
- {What is only checked by hand.}

## Decisions

### {YYYY-MM-DD} {Decision title, then "(recorded)" when the date is not the decision date}

- **Context:** {Why a choice was needed.}
- **Decision:** {What was chosen.}
- **Consequences:** (+) {gain}; (-) {cost}.

## Known Limits

- {What is wrong or outdated today, with the file, and why it is not fixed yet.}
````

## Checklist

- [ ] The repository has a `PRD.md`, so `TECH.md` exists in the root
- [ ] Sections are in the order above
- [ ] The diagram matches the `FROM` lines, the entrypoint and the Compose file
- [ ] Versions match the pinned `ENV *_VERSION`, `FROM` and `image:` values
- [ ] Configuration links `.env.dist` and the README variable table
- [ ] Data section says what is lost when the volume is lost
- [ ] Every decision has a date, context, decision and consequences; a date that is not the real decision date is
      marked `(recorded)`
- [ ] No placeholder and no template fact is left
- [ ] Known limits are listed, not hidden
- [ ] The file is 50 to 150 lines and has no emoji

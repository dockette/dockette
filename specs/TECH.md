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

## Writing Style

- Facts first, reason second: "**The entrypoint must `exec`.** Otherwise signals stop at the shell."
- Code spans for every file, variable, tag and path inside the image.
- Versions exactly as pinned (`1.23.16-debian`), not "latest".
- No emoji, no marketing words, no future tense except in `Known Limits`.

## Template

A filled example for `dockette/kumatron`. Replace the facts, keep the order.

````markdown
# Kumatron Tech

Uptime Kuma in one container, with optional continuous SQLite replication to S3 by Litestream.

## Architecture

```
build:  debian:bullseye-slim -> litestream binary --+
        debian:bullseye-slim -> envsubst binary   --+--> louislam/uptime-kuma:1.23.16-debian
        litestream/*.yml.tpl, entrypoint.sh       --+
run:    entrypoint.sh -> [LITESTREAM=1] litestream restore -> litestream replicate -exec node server.js
                      -> [otherwise]    node /app/server/server.js
```

- One process tree; when enabled, Litestream is the parent and exits when Uptime Kuma exits.
- No database server; state is one SQLite file in `/app/data`.

## Stack

- Upstream: `louislam/uptime-kuma:1.23.16-debian` (Node.js, port 3001).
- Litestream `v0.3.13`, envsubst from `a8m/envsubst`, both from GitHub releases.

## Layout

- `Dockerfile` - three stages; the last one is the published image.
- `entrypoint.sh` -> `/entrypoint.sh`.
- `litestream/s3.yml.tpl` -> `/srv/litestream/`; rendered to `/srv/litestream/litestream.yml`.

## Configuration

- `DATA_DIR` (default `./data/`), `LITESTREAM`, `LITESTREAM_TEMPLATE`, `LITESTREAM_DB_FILE`,
  `LITESTREAM_S3_*`, retention and interval variables; defaults in `entrypoint.sh`, list in `.env.dist`.
- `LITESTREAM_TEMPLATE=s3` selects `s3.yml.tpl`; `envsubst` fills it at start.

## Data

- Mount a volume at `/app/data`. Without Litestream, that volume is the only copy.
- With Litestream, start restores from S3 when a replica exists (`-if-replica-exists`).

## Services

- Port 3001. External: an S3-compatible bucket when Litestream is on.

## Startup Flow

1. Print configuration. 2. Render the Litestream config if a template is set. 3. Restore. 4. `exec` the
   main process.

## Build and Publish

- CI matrix tags `latest` and `20250502`, context `.`, weekly on Monday 08:00 UTC and on push to `master`.

## Testing

- `make test` runs the image on port 3001; `make test-s3` adds Litestream with values from `.env`.
- No automated check that replication works; test by hand against a bucket.

## Decisions

### 2026-09-28 Litestream as a wrapper process

- **Context:** A lost volume loses all monitors and history.
- **Decision:** `litestream replicate -exec` when `LITESTREAM=1`.
- **Consequences:** (+) continuous backup; (-) one more pinned binary.

## Known Limits

- `ENVSUBST_VERSION` says `v1.4.2`, but the download URL is hard-coded to `v1.2.0`.
- `.env.dist` sets `LITESTREAM_TEMPLATE=basic`, but only `s3.yml.tpl` exists.
- `entrypoint.sh` runs with `xtrace` and prints `LITESTREAM_S3_SECRET_ACCESS_KEY` to the log.
- Downloads are not checksum-verified, and the build stages use Debian Bullseye.
````

## Checklist

- [ ] The repository has a `PRD.md`, so `TECH.md` exists in the root
- [ ] Sections are in the order above
- [ ] The diagram matches the `FROM` lines, the entrypoint and the Compose file
- [ ] Versions match the pinned `ENV *_VERSION`, `FROM` and `image:` values
- [ ] Configuration links `.env.dist` and the README variable table
- [ ] Data section says what is lost when the volume is lost
- [ ] Every decision has a date, context, decision and consequences
- [ ] Known limits are listed, not hidden
- [ ] The file is 50 to 150 lines and has no emoji

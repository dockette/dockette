# Dockette PRD.md Specification

This document describes the `PRD.md` file in Dockette repositories that behave like a product: Compose stacks,
workspaces and images that wrap an application with our own configuration. The file says what the image or stack
is for, who runs it and what it must and must not do. How it is built is in [TECH.md](TECH.md), UI changes in
[DESIGN.md](DESIGN.md), image rules in [IMAGES.md](IMAGES.md).

## Table of Contents

- [Rules](#rules)
- [When It Is Required](#when-it-is-required)
- [Sections](#sections)
- [Writing Style](#writing-style)
- [Template](#template)
- [Checklist](#checklist)

## Rules

- `PRD.md` lives in the repository root, next to `README.md`, `TECH.md` and `AGENTS.md`.
- It is 50 to 150 lines. It is a short product document, not a backlog.
- It describes the current image or stack. Plans go to issues; open questions go to its last section.
- It links instead of repeating: usage and variables are in the README, tags in the Versions table, commands in
  the `Makefile` and `AGENTS.md`.
- A pull request that adds or removes a service, tag, variant or user-facing variable updates `PRD.md` in the
  same PR.
- Every non-goal is a decision. Removing one needs the same review as adding a feature.
- It is never copied into the image; see [DESIGN.md](DESIGN.md#rules) for the `.dockerignore` rule.

## When It Is Required

Use the image class from [IMAGES.md](IMAGES.md#image-classes):

| Class | Examples | Required |
|-------|----------|----------|
| Stack | `devstack`, `metamcp`, `neko`, `drupalista` | yes |
| Workspace | `coder`, `vibestack`, `viewdoc` | yes |
| Republish with our own layer (entrypoint, sidecar, config) | `kumatron` | yes |
| Service with a documented product scope | `adminer`, `apidoc`, `packagist` | when its README needs more than one Usage section |
| Base, runtime, tool, plain republish | `debian`, `php`, `deploy`, `cadvisor` | no |

The product of an image is the running container. Its users are the people who write `docker run` or a
Compose file, and its success is measured by what works without reading the Dockerfile.

## Sections

Use these `##` sections in this order:

1. `# {Name} PRD` and one sentence: what the image or stack is, with its class.
2. `## Problem` - 2 to 4 sentences: what hurts without it.
3. `## Users` - who runs it, on what host, with what knowledge.
4. `## Goals` - 3 to 6 checkable bullets.
5. `## Non-goals` - what it will not try to be, each with a one-clause reason.
6. `## Scope` - services, variants or features as bullets or user stories, grouped by area.
7. `## Success Criteria` - observable facts: a command that starts it, a port that answers, a size, a time.
8. `## Out of Scope` - things users ask for that live in another image or upstream, with a pointer.
9. `## Open Questions` - dated bullets; remove each when decided and record it in `TECH.md`.

## Writing Style

- One sentence per bullet. Fact first, reason second.
- Name tags, services and ports in code spans: `dockette/devstack:php85-fpm`, port `8000`.
- Numbers over adjectives: "starts in under 30 seconds", "12 MB", not "fast" or "tiny".
- No emoji, no marketing words, no roadmap.

## Template

A filled example for `dockette/devstack`. Replace the facts, keep the order.

````markdown
# DevStack PRD

DevStack is a Compose stack for local PHP development: Apache, PHP-FPM, MariaDB and Adminer, managed by one
`devstack` shell script.

## Problem

Running several PHP projects on one laptop needs a web server, PHP with debug tools, a database and a DB UI.
Installing them on the host breaks on OS upgrades and mixes versions between projects.

## Users

- PHP developers on Linux or macOS with Docker and the Compose plugin installed.
- They know `docker compose` basics and edit `/etc/hosts`; they don't want to write Compose files.

## Goals

- One command (`devstack up`) starts the whole stack from `~/.devstack/docker-compose.yml`.
- `~/projects` is mounted as `/srv` and served by Apache, with no per-project config.
- Xdebug, Composer and a mail catcher work without extra setup.
- The same images and ports on every developer machine.

## Non-goals

- Not for production - default passwords are `root` and Xdebug is on.
- No per-project PHP version - one PHP-FPM service keeps the stack small.
- No GUI installer - the script and one Compose file are the whole interface.

## Scope

- Services: `apache`, `php85` (FPM), `adminer`, `mariadb`, plus `data` and `userdirs` volume holders.
- Optional services, commented out: `nodejs`, `postgresql`, `blackfire`.
- Script commands: start, restart, logs, build, destroy, upgrade, exec, attach as `dfx` user.
- Fixed IPs in `172.10.10.0/24` so Xdebug and `/etc/hosts` entries stay stable.
- SSH agent forwarded into PHP and Node.js containers.

## Success Criteria

- After the README install steps, `http://localhost` serves a project and `http://localhost:8000` shows Adminer.
- `make test` passes: Compose config is valid and `devstack` has no shell syntax errors.
- CI validates the Compose file and builds `apache`, `php85-fpm` and `nodejs` every week.

## Out of Scope

- Database UI details and themes - see `dockette/adminer`.
- Plain PHP images for CI or production - see `dockette/php` and `dockette/web`.
- Browser-based workspaces - see `dockette/coder`.

## Open Questions

- 2026-09-28: Move the MariaDB root password to `.env` instead of the Compose file?
````

## Checklist

- [ ] The repository is a stack, workspace or republish with our own layer, so `PRD.md` exists in the root
- [ ] Sections are in the order above
- [ ] Goals and success criteria can be checked by running a command or opening a port
- [ ] Every non-goal has a reason
- [ ] Out of scope items point to the image or upstream project where the thing lives
- [ ] Scope matches the Compose file, the Versions table and the README variables today
- [ ] Open questions are dated; decided ones are removed and recorded in `TECH.md`
- [ ] The file is 50 to 150 lines and has no emoji or marketing words

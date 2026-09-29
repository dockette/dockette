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
- The README links it. `AGENTS.md` doesn't link it; it covers development only.
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

fxnorm follows this table: `dockette/prd-tech-exist` (a warning) knows the repositories named in
the "yes" rows and treats a Compose file with two or more services and no Dockerfile as a stack. A service in the
conditional row gets the files when its README needs more than one Usage section, or when a task asks for them.

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
9. `## Open Questions` - bullets dated with the day the question was written down; remove each when decided and
   record it in `TECH.md` (a dated entry, see [TECH.md](TECH.md#decisions)).

## Writing Style

- One sentence per bullet. Fact first, reason second.
- Name tags, services and ports in code spans: `dockette/devstack:php85-fpm`, port `8000`.
- Numbers over adjectives: "starts in under 30 seconds", "12 MB", not "fast" or "tiny".
- No emoji, no marketing words, no roadmap.

## Template

`{...}` marks a placeholder: replace it with facts from the repository; delete lines that don't apply. Scope and
success criteria describe what the image or stack does today, checked by running it, not what the template or the
old README claims.

````markdown
# {Name} PRD

{Name} is {what the image or stack is, one sentence, with its class from IMAGES.md}.

## Problem

{2 to 4 sentences: what a user has to do without it, and what goes wrong.}

## Users

- {Who runs it, on what host, with what installed.}
- {What they know and what they don't want to do.}

## Goals

- {A checkable goal: one command that starts it.}
- {A checkable goal about what works without extra setup.}

## Non-goals

- {What it will not try to be} - {the reason in one clause}.

## Scope

- {Area, e.g. Services or Variants}: {what exists today, from the Compose file, the folders or docker.yml}.
- {Area}: {what exists}.

## Success Criteria

- {Observable fact: a URL or port that answers after the README steps.}
- {What `make test` or CI checks, and how often CI runs.}

## Out of Scope

- {What users ask for} - see {the image or upstream project where it lives}.

## Open Questions

- {YYYY-MM-DD}: {A question that is not decided yet.}
````

## Checklist

- [ ] The repository is a stack, workspace or republish with our own layer, so `PRD.md` exists in the root
- [ ] Sections are in the order above
- [ ] Goals and success criteria can be checked by running a command or opening a port
- [ ] Every non-goal has a reason
- [ ] Out of scope items point to the image or upstream project where the thing lives
- [ ] Scope matches the Compose file, the Versions table and the README variables today
- [ ] No placeholder and no template fact is left
- [ ] Open questions are dated; decided ones are removed and recorded in `TECH.md`
- [ ] The file is 50 to 150 lines and has no emoji or marketing words

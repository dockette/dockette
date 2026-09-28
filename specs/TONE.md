# Dockette Tone of Voice

This document describes how we write in Dockette repositories: `README.md`, `AGENTS.md`, `PRD.md`, `TECH.md`,
`DESIGN.md`, commit messages, issues and pull requests. The layout of these files is described in
[REPOSITORY.md](REPOSITORY.md) and [AGENTS.md](AGENTS.md). This document covers the words inside them. The
`README.md` is also the Docker Hub description, so it is often the first thing a user reads.

## Table of Contents

- [Rules](#rules)
- [Principles](#principles)
- [Person and Voice](#person-and-voice)
- [Sentences](#sentences)
- [Words to Avoid](#words-to-avoid)
- [Do and Don't](#do-and-dont)
- [Formatting](#formatting)
- [Callouts](#callouts)
- [README](#readme)
- [Agent and Project Documents](#agent-and-project-documents)
- [Commit Messages](#commit-messages)
- [Issues and Pull Requests](#issues-and-pull-requests)
- [Before and After](#before-and-after)
- [Checklist](#checklist)

## Rules

- Write in plain English. Use articles: "the latest version", "a Debian-based image".
- The first sentence says what the image is and what it does.
- Give facts instead of adjectives: versions, sizes, ports, base images.
- Say what the image does not include and what is dangerous.
- Address the reader as "you". Don't write "I", and don't write "our" about the team.
- Use active voice and present tense.
- Every code block is introduced by a short sentence that ends with a colon.
- No emoji in prose, headings or lists. The only exception is the optional author links line in the README header.
- No marketing words (see [Words to Avoid](#words-to-avoid)).
- Hints and warnings use GitHub alerts (see [Callouts](#callouts)).

## Principles

1. **Say what it is in the first sentence.** What is inside, what it is based on and who it is for: "PHP 8.5 CLI
   and FPM images on Debian Bookworm, with Composer installed."
2. **Start from the reader's problem.** One sentence on what is hard without the image, then what the image does
   about it.
3. **Show first, explain second.** Put one runnable `docker run` command before any table.
4. **Be concrete.** "Based on `dockette/debian:bookworm-slim`, 48 MB" beats "tiny image". "Listens on port 8000"
   beats "ready to use".
5. **Be honest about limits.** "`linux/amd64` only; upstream has no ARM build." "Runs as root." Say it.
6. **Keep it calm.** At most one exclamation mark per README. A dry joke is fine once, never in installation,
   security or upgrade text.
7. **One voice for all repositories.** A reader who moves from `dockette/php` to `dockette/adminer` should not
   notice a change in style.

## Person and Voice

| Who | Use | Example |
|-----|-----|---------|
| The reader | "you" | "You can mount your own `pgbouncer.ini`." |
| Walking through an example together | "let's", "we" | "Let's start the stack and open Adminer:" |
| The maintainer | name in third person | "Consider supporting **f3l1x**." |
| The author | not used | – |

- Don't write "I have prepared…", "our images" or "for you".
- State behaviour as fact: "The entrypoint renders `nginx.conf` from environment variables", not "The config
  will be generated".

## Sentences

- Aim for 12 to 20 words per sentence. A very short sentence can land a point: "Nothing else is needed."
- One idea per paragraph, two to four sentences per paragraph.
- Put the important word first: "`ADMINER_THEME` selects the theme", not "If you want another theme, you can set
  `ADMINER_THEME`".
- One rhetorical question may open a section ("Which tag should you use?"). Don't stack them.
- Don't use em dashes. Use a colon, a comma or a new sentence.
- Write "PHP 8.2 or later" in prose, not "PHP 8.2+".

## Words to Avoid

| Avoid | Use instead |
|-------|-------------|
| awesome, great, super, amazing, ultimate, first class | Say what it does, with a fact |
| tiny, tiniest, lightweight, blazing fast, lightning fast | A number: "48 MB", "starts in 1 s" |
| ready-to-use, batteries included, boxed | List what is installed |
| seamless, effortless, painless, magic | Describe the step the reader no longer takes |
| simply, just, easily, obviously | Leave it out |
| powerful, robust, modern, cutting-edge | Name the feature |
| leverage, utilize | use |
| dockerized | in a Docker image |
| home programming | local development |
| a couple of packages | Name the packages |
| setup (as a verb) | set up. "Setup" is the noun |
| consider to support | consider supporting |
| latest version (without "the") | the latest version |
| please note that, it should be noted | Leave it out, or use a `NOTE` alert |
| click here | Link the words that name the target |

## Do and Don't

| Do | Don't |
|----|-------|
| "Adminer with the MySQL driver, 9 MB." | "Tiniest boxed dockerized Adminer" |
| "Based on Debian Bookworm, with Composer installed." | "This super image has also preinstalled Composer." |
| "Run the image:" + code block | A code block with no lead-in |
| "Don't expose port 8080 to the internet." | "Use with care" + emoji |
| "Adds about 12 MB to the image." | "Lightweight" |
| "See the [Compose file reference](https://docs.docker.com/reference/compose-file/)." | "Click [here](…)." or a bare URL |
| `> [!TIP]` alert | `[**TIP**]`, bold "NOTE:", emoji |

## Formatting

- Inline code for every image, tag, variable, port, path and command: `dockette/php:8.5`, `PHP_VERSION`,
  `8000`, `/srv`, `make build`.
- Code blocks have a language: `sh`, `yaml`, `Dockerfile`, `ini`.
- Commands must run when copied. Use real image names and tags and no `...` inside commands. Split long
  `docker run` commands with `\`.
- Show output in a separate block introduced by "It prints:" instead of describing it.
- Tables are for tags and environment variables only (see [REPOSITORY.md](REPOSITORY.md#versions-and-environment-tables)).
  A screenshot gallery uses one `###` per screen; only thumbnails of variants of one screen may sit in an HTML
  grid (see [REPOSITORY.md](REPOSITORY.md#screenshots)).
- README headings use the fixed names from [REPOSITORY.md](REPOSITORY.md). Other headings use Title Case. No
  emoji, exclamation marks or links in headings.
- Link the exact page, not a home page.

## Callouts

Use GitHub alerts. Don't use bold labels, brackets or emoji for hints.

```markdown
> [!WARNING]
> This image is archived and no longer updated. Use `dockette/php:8.5` instead.
```

| Alert | Use for |
|-------|---------|
| `> [!NOTE]` | Background the reader may skip |
| `> [!TIP]` | A better way to do something |
| `> [!IMPORTANT]` | Something the reader must do for it to work |
| `> [!WARNING]` | Something that breaks or loses data when ignored; archived images |
| `> [!CAUTION]` | Security risks: open ports, root, default passwords |

- At most one alert per section. Two alerts in a row mean the text needs rewriting.
- The alert text is one to three sentences and says what to do.

## README

- The README follows [REPOSITORY.md](REPOSITORY.md): header with a description, Usage, Versions, Environment,
  topic sections, Development, Maintenance.
- The description is one to three sentences and answers three questions: what is inside, what it is based on and
  who it is for.
- `## Usage` starts with one runnable command, then says what the image is based on and what it needs (volumes,
  ports, a config file).
- Environment variables are explained in the Environment table, one row each, with the default.
- State the image class and the platforms when they are not the defaults: "Runs as root.", "`linux/amd64` only."

## Agent and Project Documents

`AGENTS.md`, `PRD.md`, `TECH.md` and `DESIGN.md` are read by people and AI coding agents. The rules above apply,
with these additions:

- State facts first, then the rule that follows: "Every version folder is a full copy. Apply a change to every
  folder."
- Give the reason in one clause: "…because users pin old tags".
- Correct wrong assumptions directly: "PHP comes from `packages.sury.org`, not from the official `php` image."
- Describe the current state. No plans, TODO lists or "we are working on…".
- Imperatives are short and absolute: "Never overwrite a frozen tag."
- The templates in these specs are outlines. `{...}` marks a placeholder; every fact comes from the repository,
  and a line that doesn't apply is deleted, not kept as a guess.
- Dates are real. A decision or screenshot whose date is unknown is marked (`(recorded)`, `undated`), never
  given an invented date.

## Commit Messages

```
{Area}: {what changed, imperative, lowercase}

Optional body that explains why. Wrap at 72 characters.
```

- The summary has at most 50 characters (without the pull request number GitHub adds).
- `{Area}` is the part of the repository: `Docker`, `CI`, `Makefile`, `Docs`, or a tag (`8.5`, `bookworm`).
- Use the imperative mood: "add", "fix", "drop", not "added", "fixes", "dropping".
- Match the style of the last 10 commits when the repository has its own (`git log -10 --oneline`).
- The body says why, not how. The diff shows how.
- Don't refer to chats, tickets in other systems or the state of your work ("wip", "final version", "as
  discussed").

Examples:

```
Docker: add pcov extension
8.5: switch base to trixie
CI: build linux/arm64
Docs: document ADMINER_THEME
```

## Issues and Pull Requests

- The title uses the commit summary style: `8.4-fpm: php-fpm fails to start on arm64`.
- A bug report lists the image and tag, the platform, the command, what happened and what you expected, in that
  order. Paste logs as text in a code block, not as a screenshot.
- A pull request says what changed and why in two to four sentences, then how it was checked (`make build`,
  `make test`).
- Link the issue it closes: `Closes #123`.
- Replies are short and factual. Thank the author once. Don't apologise at length or promise dates.
- No greetings like "Hi guys", no urgency ("ASAP", "please merge quickly").

## Before and After

Real text from our repositories, rewritten in the house style.

**dockette/devstack, description**

> Before: Great LAMP devstack based on **Docker** & **Docker Compose** for your home programming.

> After: A LAMP stack for local development, run by Docker Compose: Apache, PHP 8.5, MariaDB and Adminer, with
> Node.js and PostgreSQL ready to switch on. A single `devstack` script manages the containers.

**dockette/devstack, tip**

> Before: [**TIP**] There used to be a skeleton in ubuntu/debian/mint system.

> After:
>
> ```markdown
> > [!TIP]
> > Debian-based systems ship a default `.bashrc` in `/etc/skel`. Copy it into your home directory to start from.
> ```

**dockette/devstack, configuration**

> Before: I have prepared docker configuration file for you. You can download it here.

> After: Download the [`docker-compose.yml`](https://github.com/dockette/devstack/blob/master/docker-compose.yml)
> file:

**dockette/php, packages**

> Before: These images have preinstalled couple of linux packages: apt-transport-https ca-certificates git. This
> super image has also preinstalled Composer.

> After: The images include `apt-transport-https`, `ca-certificates`, `git` and Composer 2.

**dockette/adminer, description**

> Before: Tiniest boxed dockerized Adminer (MySQL, PostgreSQL, SQLite, Mongo, Oracle, MSSQL) Dockerfiles.

> After: Adminer, the single-file database manager, in Docker images from 8 MB. Each tag carries the driver for
> one database (`mysql`, `pgsql`, `mongo`, `mssql`, `oracle-19`) or for several (`full`).

**Footer**

> Before: Consider to support **f3l1x**. Thank you for using this package.

> After: Consider supporting **f3l1x**. Thank you for using this package.

**AGENTS.md bullet**

> Before: Be careful when changing Dockerfiles.

> After: **Every version folder is a full copy.** There is no shared template; a change for all versions is made
> in every folder, then `make test-all` checks them.

**Commit message**

> Before: `update`

> After: `8.5: add intl extension`

## Checklist

- [ ] The first sentence says what is inside, what it is based on and who it is for
- [ ] No words from [Words to Avoid](#words-to-avoid)
- [ ] "you" for the reader; no "I", "our" or "us"
- [ ] One runnable command comes before any table
- [ ] Every code block has a lead-in sentence ending with a colon
- [ ] Hints use GitHub alerts; no emoji outside the author links line
- [ ] Limits are stated plainly: root user, missing platforms, EOL tags
- [ ] Commit summaries are `{Area}: {imperative}` and at most 50 characters
- [ ] Pull requests say what, why and how it was checked

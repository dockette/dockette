# Dockette DESIGN.md Specification

This document describes the `DESIGN.md` file in Dockette repositories whose image serves a web UI. Most of those
UIs come from upstream software; the file records what we change on top of it (themes, plugins, login flow,
defaults, ports) and how to check that a rebuild did not break what users see. Image rules are in
[IMAGES.md](IMAGES.md), repository files in [REPOSITORY.md](REPOSITORY.md), app-like stacks in [PRD.md](PRD.md)
and [TECH.md](TECH.md).

## Table of Contents

- [Rules](#rules)
- [When It Is Required](#when-it-is-required)
- [Sections](#sections)
- [Rebranded Upstream UI](#rebranded-upstream-ui)
- [Writing Style](#writing-style)
- [Screenshots](#screenshots)
- [Template](#template)
- [Checklist](#checklist)

## Rules

- `DESIGN.md` lives in the repository root, next to `README.md` and `AGENTS.md`.
- The README links it. `AGENTS.md` doesn't link it; it covers development only.
- It is 50 to 150 lines, or 25 to 60 for a [rebranded upstream UI](#rebranded-upstream-ui). Longer material moves
  to `.docs/` and is linked.
- It describes our layer, not the upstream UI. Link the upstream project for everything we don't change.
- It links instead of repeating: environment variables are in the README tables, commands in the `Makefile`
  and `AGENTS.md`.
- CSS, themes and plugins are defined in files in the repository or in the upstream release. `DESIGN.md` names
  those files and never copies their values.
- It is updated in the same pull request as the change that makes it wrong, including an upstream version bump
  that changes the UI.
- It is never copied into the image. Dockerfiles copy named files only; if one copies the whole context, list
  `DESIGN.md` in `.dockerignore`.
- Screenshots live directly in `.docs/` (see [REPOSITORY.md](REPOSITORY.md#layout)). Variants of one screen may
  use a subfolder (`.docs/themes/`). Don't add an `assets/` level.
- Every screenshot is listed in `## Screenshots` with the date it was taken (see [Screenshots](#screenshots)).

## When It Is Required

| Image | UI | Required |
|-------|----|----------|
| Service with a browser UI we customize | `adminer` (themes, plugins) | yes |
| Service with pages we write (HTML, JS, Go templates) | `apidoc` (`html/`), `redoc` (`src/`), `viewdoc` | yes |
| Upstream UI with only our branding (title, logo, favicon, colors, a start page) | an image that sets its own name and logo on an upstream UI | yes, short: see [Rebranded Upstream UI](#rebranded-upstream-ui) |
| Republish or workspace with an upstream UI we don't change | `kumatron`, `packagist`, `neko`, `coder` | no; one README line links the upstream UI |
| Runtime, tool or base image | `php`, `web`, `nodejs`, `debian`, `deploy` | no |

If a later change adds a theme, plugin, custom CSS or default page to an image, the file becomes required.
Branding set only through an upstream variable (a title in `ENV`) with no file in the repository doesn't count;
one README line says what it changes.

fxnorm checks this with `dockette/design-md-exists` (a warning): it knows the repositories named above and
treats a repository with its own HTML pages (`html/`, `public/`, `www/`) served by Caddy, nginx or on port 80,
8080 or 3000 as one that needs the file.

## Sections

Use these `##` sections in this order. Leave a section out only when it does not apply, and say so in one line
("Dark mode is whatever the upstream theme provides.") instead of deleting it silently.

1. `# {Name} Design` and one sentence: what the UI is and who opens it.
2. `## Principles` - 3 to 5 bullets that decide real choices.
3. `## Inventory` - screens and variants, each with its source (upstream file, our plugin, our CSS).
4. `## Layout` - usually "upstream"; list only what we change.
5. `## Typography` - usually "upstream theme".
6. `## Colors and Themes` - where themes come from, how one is selected, what the default is.
7. `## States` - first start, login, empty, error, and what the container prints for each.
8. `## Accessibility` - what upstream provides and anything our layer removes or adds.
9. `## Dark Mode` - supported or not, and how it is switched.
10. `## Responsive` - what works on a narrow screen; "desktop only" is a valid answer.
11. `## Screenshots` - every file in `.docs/`, what it shows, when it was taken, how to retake it.
12. `## Changing the UI` - traps: what users rely on, what an upstream bump can break.
13. `## Checklist` - 4 to 8 items to verify before merging.

## Rebranded Upstream UI

An image that shows an upstream UI and changes only its branding (title, logo, favicon, colors, a start page
that links the upstream screens) needs a short `DESIGN.md`, 25 to 60 lines. It exists so that an upstream bump
doesn't silently drop or break the branding. The minimum content:

1. `# {Name} Design` and one sentence: which upstream UI, at which version, and what we change.
2. `## Principles` - at least "**Upstream UI, our branding only.**" and what must stay as upstream ships it.
3. `## Inventory` - every file we add or replace, where it lands in the image
   (`html/index.html` -> `/srv/www/index.html`), and each upstream screen we show with its version variable.
4. `## Colors and Themes` - where the branding is defined (file, variable, CSS selector) and what the default is.
5. `## Screenshots` - every file in `.docs/`, with its date (see [Screenshots](#screenshots)).
6. `## Changing the UI` - what an upstream bump can break: the paths, selectors or config keys the branding
   depends on, and where the upstream version is pinned.
7. `## Checklist` - 3 to 5 items: start the container, open every screen, check the name and logo, compare with
   the screenshots.

`## Layout`, `## Typography`, `## States`, `## Accessibility`, `## Dark Mode` and `## Responsive` stay in their
places with one line each, for example "Upstream ({upstream} {version}); we change nothing." Link the upstream
project once, in the first line.

## Writing Style

- Start each bullet with the fact, then the reason.
- Name files, variables and tags in code spans: `.plugins/adminer-server-list.php`, `ADMINER_THEME`, `:full`.
- Numbers over adjectives: "port 80", "200 px thumbnails", "8 workers".
- State what is not supported. No emoji, no marketing words.

## Screenshots

- One bullet per file: path, what it shows, the date it was taken and the tag it shows:
  "`.docs/landing.png` - start page with the five viewers, 2026-09-28, `latest`."
- A screenshot whose date is unknown is listed as `undated`. Compare it with the running container: retake it when
  it shows an old name, layout or version, otherwise date it on the day you checked it and add `(checked)`.
- An outdated screenshot that can't be retaken now goes to `## Changing the UI` (or `TECH.md` Known Limits) with
  what differs.
- The README gallery follows [REPOSITORY.md](REPOSITORY.md#screenshots): one `###` or one captioned line per
  screen; a grid of thumbnails only for variants of one screen.

## Template

`{...}` marks a placeholder: replace it with facts from the repository; delete lines that don't apply. Keep the
section order. Every claim is checked in the Dockerfile, the entrypoint and the running container, not taken from
the template or the old README.

````markdown
# {Name} Design

{What the UI is, which port serves it and who opens it.}

## Principles

- **{What stays upstream, e.g. the application code}.** {What we never patch.}
- **{How our layer is switched on, e.g. by environment variables}.** {What the default container looks like.}
- **{A security constraint, e.g. what never reaches the browser}.** {How.}

## Inventory

- {Screen or variant}: {upstream file, or our file in the repository}, served as `{path in the image}`.
- {Plugin, theme or page we add}: `{file}`, {which tags have it, when not all}.

## Layout

- {"Upstream" and only what our layer changes.}

## Typography

- {"Upstream theme", or the file that sets it.}

## Colors and Themes

- {Where themes or branding come from and where they land in the image.}
- {How one is selected: the variable and its default.}

## States

- {First start, login, empty, error: what the UI shows and what the container prints.}

## Accessibility

- {What upstream provides and what our layer adds or removes.}

## Dark Mode

- {Supported or not, and how it is switched.}

## Responsive

- {What works on a narrow screen, or "Desktop only, as upstream. Not tested below {width} px."}

## Screenshots

- `.docs/{file}` - {what it shows}, {YYYY-MM-DD or undated}, `{tag}`.
- {How to retake them, e.g. `make run` and the URL.}

## Changing the UI

- {Public names users rely on: variables, paths, tags; how to rename them.}
- {What an upstream bump can break and what to try after it.}

## Checklist

- [ ] {4 to 8 checks before merging, e.g. the default container looks like upstream}
- [ ] Screenshots in `.docs/` updated if the UI changed
````

## Checklist

- [ ] The image serves a UI we customize, so `DESIGN.md` exists in the root
- [ ] Sections are in the order above; skipped sections say why in one line
- [ ] The file describes our layer and links upstream for the rest
- [ ] Every inventory item names its source file
- [ ] Themes and CSS point to the file or folder that defines them
- [ ] Screenshots are in `.docs/` and each is listed with its date (or `undated`)
- [ ] No placeholder and no template fact is left
- [ ] `DESIGN.md` is not copied into the image
- [ ] The file is 50 to 150 lines (25 to 60 for a rebranded upstream UI) and has no emoji

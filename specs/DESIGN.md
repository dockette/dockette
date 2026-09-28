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
- [Writing Style](#writing-style)
- [Template](#template)
- [Checklist](#checklist)

## Rules

- `DESIGN.md` lives in the repository root, next to `README.md` and `AGENTS.md`.
- It is 50 to 150 lines. Longer material moves to `.docs/` and is linked.
- It describes our layer, not the upstream UI. Link the upstream project for everything we don't change.
- It links instead of repeating: environment variables are in the README tables, commands in the `Makefile`
  and `AGENTS.md`.
- CSS, themes and plugins are defined in files in the repository or in the upstream release. `DESIGN.md` names
  those files and never copies their values.
- It is updated in the same pull request as the change that makes it wrong, including an upstream version bump
  that changes the UI.
- It is never copied into the image. Dockerfiles copy named files only; if one copies the whole context, list
  `DESIGN.md` in `.dockerignore`.
- Screenshots live in `.docs/assets/` (see [REPOSITORY.md](REPOSITORY.md#layout)).

## When It Is Required

| Image | UI | Required |
|-------|----|----------|
| Service with a browser UI we customize | `adminer` (themes, plugins) | yes |
| Service with pages we write (HTML, JS, Go templates) | `apidoc` (`html/`), `redoc` (`src/`), `viewdoc` | yes |
| Republish or workspace with an upstream UI we don't change | `kumatron`, `packagist`, `neko`, `coder` | no; one README line links the upstream UI |
| Runtime, tool or base image | `php`, `web`, `nodejs`, `debian`, `deploy` | no |

If a later change adds a theme, plugin, custom CSS or default page to an image, the file becomes required.

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
11. `## Screenshots` - files in `.docs/assets/` and how to regenerate them.
12. `## Changing the UI` - traps: what users rely on, what an upstream bump can break.
13. `## Checklist` - 4 to 8 items to verify before merging.

## Writing Style

- Start each bullet with the fact, then the reason.
- Name files, variables and tags in code spans: `.plugins/adminer-server-list.php`, `ADMINER_THEME`, `:full`.
- Numbers over adjectives: "port 80", "200 px thumbnails", "8 workers".
- State what is not supported. No emoji, no marketing words.

## Template

A filled example for `dockette/adminer`. Replace the facts, keep the order.

````markdown
# Adminer Design

The Adminer images serve the Adminer database UI on port 80. Developers open it in a browser to inspect and edit
databases in local and staging stacks.

## Principles

- **Upstream UI, unmodified PHP.** We download the release file; we never patch `index.php`.
- **Everything is opt-in by environment.** Themes and plugins are off until a variable turns them on, so the
  default container looks like upstream Adminer.
- **Credentials never reach the browser.** Server-list and autologin plugins read DSNs from the environment on
  the server side.

## Inventory

- Login and database screens: upstream `adminer-{version}.php`, served as `/srv/index.php`.
- Server dropdown with Auto Sign-In: `.plugins/adminer-server-list.php`.
- Autologin (skips the login form): `.plugins/adminer-autologin.php`.
- MSSQL encryption options: `.plugins/adminer-mssql-encrypt.php` (`:mssql` only).
- Variants differ in drivers, not in UI: `full`, `editor`, `mysql`, `postgres`, `mongo`, `mssql`, `oracle-*`.

## Layout

- Upstream; our plugins add one dropdown and one button to the login form.

## Typography

- Upstream theme.

## Colors and Themes

- Themes come from the upstream release `designs/` folder, copied to `/srv/designs/` at build time.
- `ADMINER_THEME={name}` copies `adminer.css` (and `adminer-dark.css` when present) at start.
- Default: no theme file, upstream look.

## States

- Unknown theme: the container prints a warning and lists available themes; the UI stays default.
- Autologin and server list both enabled: only autologin is activated.
- Server without credentials in its DSN: listed, but needs manual login; no Auto Sign-In button.

## Accessibility

- Upstream HTML forms; our plugins use plain `<select>` and `<button>`, keyboard reachable.

## Dark Mode

- Only through a theme that ships `adminer-dark.css`.

## Responsive

- Desktop first, as upstream. Not tested below 1024 px.

## Screenshots

- `.docs/assets/adminer.png` (default) and `.docs/assets/themes/{name}.png` (one per theme, 200 px wide in
  the README table).
- After an Adminer version bump, run `make run` and retake the default screenshot.

## Changing the UI

- Variable names (`ADMINER_THEME`, `ADMINER_PLUGIN_*`, `ADMINER_SERVERS_*`) are public; rename only with a
  deprecation note in the README.
- A new upstream major can change the plugin API (`Adminer\Plugin`); start every variant and log in once.
- Adding a theme to the README table needs its screenshot in `.docs/assets/themes/`.

## Checklist

- [ ] Default container (no variables) looks like upstream Adminer
- [ ] Each changed plugin tested with and without credentials
- [ ] `ADMINER_THEME` with a valid and an invalid name
- [ ] No DSN or password appears in page source
- [ ] Screenshots updated if the UI changed
````

## Checklist

- [ ] The image serves a UI we customize, so `DESIGN.md` exists in the root
- [ ] Sections are in the order above; skipped sections say why in one line
- [ ] The file describes our layer and links upstream for the rest
- [ ] Every inventory item names its source file
- [ ] Themes and CSS point to the file or folder that defines them
- [ ] Screenshots are in `.docs/assets/`
- [ ] `DESIGN.md` is not copied into the image
- [ ] The file is 50 to 150 lines and has no emoji

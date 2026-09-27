# Dockette Images Specification

This document describes the lifecycle of Dockette images: which bases they build on, which versions
are supported, how tags are named, how images are deprecated and archived, and the security baseline
every published image meets. How a Dockerfile is written is in [DOCKERFILE.md](DOCKERFILE.md), how it is
built and pushed is in [WORKFLOWS.md](WORKFLOWS.md) and [MAKEFILE.md](MAKEFILE.md).

## Table of Contents

- [Rules](#rules)
- [Image Classes](#image-classes)
- [Base Image Lineage](#base-image-lineage)
- [Supported Versions](#supported-versions)
- [Tag Naming](#tag-naming)
- [Lifecycle States](#lifecycle-states)
- [Deprecation and Archiving](#deprecation-and-archiving)
- [OCI Labels](#oci-labels)
- [Security Baseline](#security-baseline)
- [Runtime Conventions](#runtime-conventions)
- [Platforms](#platforms)
- [Versions Table](#versions-table)
- [Checklist](#checklist)

## Rules

- Every tag on Docker Hub has a source: a folder (or Dockerfile) in the repository and an entry in the workflow matrix. No hand-pushed tags.
- Every folder with a Dockerfile is either built by CI or removed. Dead folders are deleted; git history keeps them.
- An image is only built on a base that is supported upstream (distro, runtime or application). See [Supported Versions](#supported-versions).
- `latest` always points to a supported tag that CI rebuilds. A `latest` older than the newest version tag is a bug.
- Images are rebuilt weekly (see [WORKFLOWS.md](WORKFLOWS.md#triggers)). A tag whose Hub `last_updated` is older than 30 days while it is in the matrix is a broken build and gets fixed or dropped.
- A tag is never deleted from Docker Hub while it is still being pulled. It is frozen and marked as deprecated instead.
- Downloads are pinned and verified. No `curl | sh`, no `--no-check-certificate`, no `[trusted=yes]`.
- No private keys, passwords or tokens in the build context or the image.
- Services run as a non-root user when the software allows it, and declare a `HEALTHCHECK`.
- Every image carries `org.opencontainers.image.source` and `org.opencontainers.image.title`.

## Image Classes

| Class | Examples | Base | Runs as | HEALTHCHECK |
|-------|----------|------|---------|-------------|
| Base | `debian`, `alpine` | Official distro image | root (with `dfx` uid 1000 created) | no |
| Runtime | `php`, `nodejs`, `ci` | `dockette/debian`, `dockette/alpine` | root | no |
| Tool | `deploy`, `hashicorp`, `mailx`, `phuml`, `vercel`, `flyio` | Runtime or base image | root or `dfx` | no |
| Service | `adminer`, `web`, `apidoc`, `redoc`, `packagist`, `mockbin` | Base, runtime or official image | non-root when possible | yes |
| Republish | `cadvisor`, `pgbouncer`, `timescaledb`, `litellm`, `nexus`, `repman` | Upstream image, pinned | upstream user | upstream or added |
| Workspace | `coder`, `vibestack`, `viewdoc`, `vagrant` | Upstream or distro | named user (`coder`, `vagrant`) | no |
| Stack | `metamcp`, `neko`, `drupalista`, `devstack` | Compose file only, or build-only | n/a | n/a |

The class decides the security baseline below. Write the class in the README intro sentence.

## Base Image Lineage

```
debian:<codename>[-slim]  (official)
└── dockette/debian:<codename>[-slim]
    ├── dockette/php:<x.y>[-fpm]
    │   └── dockette/deploy:deployer<n>
    ├── dockette/web:php-<xy>
    ├── dockette/adminer:{mssql,oracle-*}
    └── dockette/{ansible,httpdump,expose,...}

alpine:<x.y>  (official)
└── dockette/alpine:<x.y>
    ├── dockette/nodejs:v<n>
    ├── dockette/ci:{node<n>,php<xy>}
    └── dockette/{mailx,hugo,ssh,phuml,vercel,flyio,...}

<upstream>:<pinned>  (official or vendor image)
└── dockette/<republish>
```

- Use at most three levels: official → `dockette/<base>` → `dockette/<runtime>` → one tool image on top.
- A child image uses the newest supported tag of its parent, unless it needs an older runtime on purpose (`deploy:deployer6` on `php:7.4`).
- A child image never uses an older distro than its parent offers. Move all children when the parent gets a new codename.
- Use `alpine:<x.y>` or `debian:<codename>` directly only in multi-stage build stages, `brotli`-style export images and republishes. Final stages of Dockette-owned images build on `dockette/debian` or `dockette/alpine` ([DOCKERFILE.md](DOCKERFILE.md#base-images)).
- Legacy alias repositories (`dockette/jessie`, `dockette/stretch`, `dockette/sid`, `dockette/wheezy`, `dockette/edge`, `dockette/php71`, `dockette/jdk8`, ...) are frozen. Never build on them.
- Floating upstream tags (`ubuntu`, `ubuntu-xfce`, `base`, `edge`, `sid`, no tag) are allowed only for `edge`/`sid` tags of a base image. Everything else pins a version.
- When a child depends on a parent, the parent is built first; after a parent release, trigger the children (`workflow_dispatch`) instead of waiting for Monday.

## Supported Versions

A version is **supported** while its upstream still ships security fixes, as published on
[endoflife.date](https://endoflife.date). LTS/ELTS paid extensions don't count.

| Product | Supported (September 2026) | EOL, freeze | Source |
|---------|----------------------------|-------------|--------|
| Debian | `trixie` (13), `bookworm` (12, LTS until 2028-06-30) | `bullseye` (LTS ended 2026-08-31), `buster`, `stretch`, `jessie`, `wheezy` | [debian](https://endoflife.date/debian) |
| Alpine | `3.21` (until 2026-11-01), `3.22`, `3.23`, `3.24`, `edge` | `3.20` and older | [alpine](https://endoflife.date/alpine) |
| PHP | `8.2` (until 2026-12-31), `8.3`, `8.4`, `8.5` | `8.1` and older | [php](https://endoflife.date/php) |
| Node.js | `22`, `24` (LTS), `26` (current) | `25`, `23`, `21`, `20` (2026-04-30) and older | [nodejs](https://endoflife.date/nodejs) |
| PostgreSQL | `14` (until 2026-11-12), `15`, `16`, `17`, `18` | `13` and older | [postgresql](https://endoflife.date/postgresql) |
| MariaDB | `10.11`, `11.4`, `11.8` (LTS), current short-term | `10.6` (2026-07-06), `10.5`, `10.4`, `10.2`, `11.1`, `11.2`, `11.5`, `11.7` | [mariadb](https://endoflife.date/mariadb) |
| Docker CLI | the two newest majors | older majors | [docker-engine](https://endoflife.date/docker-engine) |

Rules:

- **New base versions**: add a new Debian codename or Alpine minor within 30 days of its release.
- **Distro EOL**: when a Debian codename or Alpine minor reaches EOL, all tags built on it move to [Deprecated](#lifecycle-states) the same month. Children move to the newest supported parent.
- **Runtime EOL on a supported distro**: legacy runtimes (`php:5.6`–`8.1`, `web:php-70`–`81`) may keep building as **Legacy** tags while the distro under them is supported and the packages still install. The README marks them as EOL. They stop building when the distro under them goes EOL.
- **Distro pinned by the runtime**: an image may only use an older distro for a supported runtime while the distro is supported (`dockette/php` on `bookworm` is fine until 2028-06-30; plan the move to `trixie` now).
- **Rolling tags** (`edge`, `sid`, `latest`) always track the newest; a failing rolling build is fixed within one week or removed from the matrix.
- **Review cadence**: once a quarter, compare every matrix entry with endoflife.date and update the [Versions Table](#versions-table).

## Tag Naming

| Kind | Pattern | Examples |
|------|---------|----------|
| Distro base | `<codename>[-slim]`, `<x.y>` | `trixie`, `bookworm-slim`, `3.23`, `edge` |
| Runtime version | `<x.y>[-<variant>]` | `8.4`, `8.4-fpm` |
| Major-only runtime | `v<major>` (existing repos only) | `v24` |
| Variant of an app | `<variant>` | `full`, `postgres`, `oracle-19`, `mcp` |
| Republish | upstream version, exactly as upstream | `1.26.0`, `0.60.6`, `v1.102.1`, `3.96.3-ubi` |
| Default | `latest` | the recommended supported tag |

- Folder name = tag ([DOCKERFILE.md](DOCKERFILE.md#folder-layout)). The workflow may map `debian-php-84` → `php-84` only in existing repos.
- Use a dot between major and minor (`8.4`), a dash before a variant (`-fpm`, `-slim`, `-systemd`). New repos don't glue versions (`php84`, `node24`, `deployer8`); existing repos keep their scheme, because renaming breaks users.
- A republish publishes the pinned version tag and `latest`. Previous version tags stay on Hub unchanged (frozen), so users can pin.
- Version tags that encode a version must contain that version. Tests assert it (`node -v` matches `v24`, `php -v` matches `8.4`).
- No date tags (`202403`, `20250502`) for new images; use the upstream version.

## Lifecycle States

| State | In matrix | Weekly rebuild | README | Hub |
|-------|-----------|----------------|--------|-----|
| **Supported** | yes | yes | listed | tag updated weekly |
| **Legacy** | yes | yes | listed with `EOL runtime` note | tag updated weekly |
| **Deprecated** | yes, max 6 months | yes, while it builds | listed with `deprecated, removed after <date>` | tag updated |
| **Frozen** | no | no | listed under "Frozen tags" with the last build date | tag kept, never overwritten |
| **Archived** (whole repo) | no workflow runs | no | banner at top: archived, what to use instead | description starts with `DEPRECATED:` |

- Tags move **Supported → Deprecated → Frozen**. Legacy is a Supported tag with an EOL runtime.
- A tag whose build breaks because its base is EOL (archived apt/apk mirrors) skips Deprecated and goes straight to Frozen.
- Frozen tags are never deleted from Hub while `tag_last_pulled` is within 6 months; after that they may be deleted with a note in the README.

## Deprecation and Archiving

Deprecating a tag:

1. Open a PR that marks the tag as deprecated in the README [Versions Table](#versions-table) with the removal date (EOL date + at most 6 months).
2. Point `latest` and any child image to a supported tag in the same PR.
3. On the removal date: delete the folder, remove the matrix entry, move the row to "Frozen tags" with its last build date.

Archiving a repository (all tags frozen, no successor inside the repo):

1. Add a banner at the top of `README.md`: `> [!WARNING] This image is archived and no longer updated. Use <replacement>.`
2. Remove `schedule` from the workflow or delete the workflow; update the Hub description once more so it starts with `DEPRECATED:`.
3. Archive the GitHub repository. Keep the Docker Hub repository; don't delete it while it is pulled.

Retiring an alias Hub repository (`dockette/jessie`, `dockette/jdk8`, `dockette/php71`, ...):

1. Set the Hub description to `DEPRECATED: use dockette/<repo>:<tag>`.
2. Never push to it again. Delete it only when it has no pulls for 6 months.

## OCI Labels

Every final stage sets these labels (the maintainer label stays, see [DOCKERFILE.md](DOCKERFILE.md#rules)):

```dockerfile
LABEL org.opencontainers.image.title="PHP 8.4"
LABEL org.opencontainers.image.description="PHP 8.4 CLI with Composer on Debian bookworm"
LABEL org.opencontainers.image.source="https://github.com/dockette/php"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.version="${PHP_VERSION}"
```

| Label | Required | Value |
|-------|----------|-------|
| `org.opencontainers.image.source` | yes | `https://github.com/dockette/<repo>` |
| `org.opencontainers.image.title` | yes | Human name incl. tag version |
| `org.opencontainers.image.description` | yes | One sentence |
| `org.opencontainers.image.licenses` | yes | License of the repository (`MIT`) |
| `org.opencontainers.image.version` | when a `<NAME>_VERSION` is pinned | The pinned version |
| `org.opencontainers.image.base.name` | recommended | Parent image, e.g. `dockette/debian:bookworm` |
| `org.opencontainers.image.created`, `revision` | set by the build | Passed by the reusable workflow, not written by hand |

Labels are inherited, so a child image must override `title`, `description`, `source` and `version`, or it
advertises its parent.

## Security Baseline

| Topic | Rule |
|-------|------|
| User | Services and workspaces set `USER` to a non-root user (`dfx` uid 1000 from `dockette/debian`/`dockette/alpine`, or the upstream user). Root is allowed for base, runtime, CI and tool images, and for services that need root to bind or drop privileges themselves (document it in the README). |
| Downloads | Every downloaded file has a pinned version and is verified: `sha256sum -c`, a signed apt/apk repository (`signed-by=`), or `ADD --checksum=sha256:...`. |
| Installers | No `curl ... \| sh`, `\| bash`, `\| php`. Use a package repo, a pinned release binary with checksum, or copy from an official image (`COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer`). |
| TLS | No `wget --no-check-certificate`, `curl -k`, `[trusted=yes]` repositories. |
| Secrets | No private keys, certificates, passwords or tokens in the repository or image. Generate at runtime or mount them. `.env.dist` holds placeholders only. |
| Build-only ENV | `DEBIAN_FRONTEND` is an `ARG`, not an `ENV`. Default passwords are not baked in as `ENV` (except ephemeral test databases, documented). |
| Size | Debian: `apt-get install --no-install-recommends`; remove build tools (`gcc`, `*-dev`, `unzip`, `wget`) in the same `RUN` or use a build stage ([DOCKERFILE.md](DOCKERFILE.md#installing-packages)). |
| Health | Services declare `HEALTHCHECK` against a local endpoint (`curl -fsS http://127.0.0.1:${PORT}/health`) or a process check. |
| SSH | Don't disable `StrictHostKeyChecking` globally; document `-o` flags in the README instead. |
| Scanning | The reusable workflow should run an image scan (e.g. Trivy/Docker Scout) on `master`, report-only at first. |

## Runtime Conventions

Signal handling and PID 1:

- A container that runs one server `exec`s it (last line of `entrypoint.sh` is `exec ...`, see [DOCKERFILE.md](DOCKERFILE.md#files-and-entrypoints)).
- A container that runs more than one process uses `tini` + `exec supervisord -n`, or `s6-overlay` when the upstream image already has it. Never background processes with `&` + `wait -n` in a shell that is PID 1 without `tini`.
- `supervisord` configs don't hard-code a socket password; use `[unix_http_server]` with file mode `0700` and no password.

Environment variables:

- Prefix settings with the image name: `ADMINER_THEME`, `VARNISH_PORT`, `PACKAGIST_*`. Unprefixed names (`PORT`, `MEMORY`, `UPLOAD`, `WORKERS`) are allowed only when they follow the upstream convention; new images add a prefixed name and keep the old one as a fallback.
- Booleans accept `1/0`; entrypoints may also accept `true/false/yes/no/on/off`.
- `<NAME>_DEBUG=1` enables `set -x` in the entrypoint.
- Document every variable in the README Environment table ([REPOSITORY.md](REPOSITORY.md)).

Config templating:

- Use `envsubst` on `*.template` files (with an explicit variable list) for config files.
- Don't `sed -i` generated config in the entrypoint when `envsubst` or a `conf.d` include will do.
- Don't dump the whole environment into files (`printenv > /etc/environment`); write only the variables the consumer needs (cron) and never secrets.

## Platforms

- Default is `linux/amd64,linux/arm64` ([WORKFLOWS.md](WORKFLOWS.md#rules)).
- `amd64` only is allowed when upstream has no `arm64` build (Oracle Instant Client, MSSQL drivers, Kasm). Write the reason as a comment next to `platforms:` and in the README.
- A republish of a multi-arch upstream image (`cadvisor`, `pgbouncer`, `timescaledb-ha`, `nexus`) publishes the same platforms as upstream.

## Versions Table

Every multi-version README lists its tags with a state column:

```markdown
| Tag | Base | Upstream EOL | State |
|-----|------|--------------|-------|
| `8.5`, `8.5-fpm` | `dockette/debian:bookworm` | 2029-12-31 | Supported |
| `8.1`, `8.1-fpm` | `dockette/debian:bookworm` | 2025-12-31 | Legacy (EOL runtime) |
| `5.5`, `5.5-fpm` | (no source) | 2016-07-21 | Frozen (last build 2021-08-09) |
```

## Checklist

- [ ] Every Hub tag has a folder and a matrix entry, or is listed as Frozen in the README
- [ ] Every folder with a Dockerfile is in the matrix
- [ ] No tag builds on an EOL distro (Debian, Alpine) or an EOL application base
- [ ] Legacy runtimes are marked in the README Versions table
- [ ] `latest` is built by CI and points to a supported tag
- [ ] Parent image is `dockette/debian` / `dockette/alpine` / a pinned official image, not a legacy alias
- [ ] Tag names follow [Tag Naming](#tag-naming); version tags are asserted by `make test`
- [ ] OCI labels `source`, `title`, `description`, `licenses` (and `version` when pinned)
- [ ] No `curl | sh`, unverified downloads, `--no-check-certificate` or `[trusted=yes]`
- [ ] No keys, certificates or passwords in the repo or image
- [ ] Services run as non-root (or the README says why not) and declare `HEALTHCHECK`
- [ ] Entrypoint `exec`s the main process, or `tini`/`supervisord` is PID 1
- [ ] Environment variables are prefixed and documented
- [ ] `linux/arm64` is built, or the reason for `amd64` only is written down
- [ ] Hub `last_updated` of every matrix tag is less than 30 days old

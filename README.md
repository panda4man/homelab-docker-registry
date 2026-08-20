# homelab-docker-registry

Self-hosted OCI registry for my local Docker projects, running [zot](https://github.com/project-zot/zot).

Built as an alternative to GHCR — local infra and storage already available, no need to push images off-box.

## Stack

- `docker-compose.yml` — zot service definition
- `config.json` — zot registry config (storage path, HTTP listener, search/UI extensions)
- `data/` — registry storage (images, metadata) — bind-mounted, gitignored
- `config/htpasswd` — basic-auth credentials file, gitignored (see Auth below)
- `.env` — local port/data-dir overrides, gitignored (copy from `.env.example`)

Bind mounts instead of named volumes, so the whole stack — config, data, compose file — lives in this one directory and backs up as a single unit.

## Usage

```bash
cp .env.example .env   # adjust ZOT_PORT / ZOT_DATA_DIR if needed
docker compose up -d
```

`ZOT_DATA_DIR` defaults to `./data`. On real hosts with dedicated storage, point it elsewhere, e.g. `ZOT_DATA_DIR=/srv/zot/data`.

Registry API + web UI: `http://localhost:${ZOT_PORT:-5050}`

Push/pull:

```bash
docker tag <image> localhost:5050/<name>:<tag>
docker push localhost:5050/<name>:<tag>
```

Registry is plain HTTP — add `localhost:5050` (or your host/port) to your Docker daemon's `insecure-registries` if pushing from another machine.

## Auth

LAN-only, not exposed to the internet, so anonymous pull stays open (`anonymousPolicy: read` in `config.json`) — no creds needed just to consume images (e.g. the-bridge).

Writers need an account. `github_actions` (self-hosted runner) is pre-scoped in `config.json` under `accessControl.repositories["**"].policies` with `read`/`create`/`update` — no delete, no admin.

Basic auth via `config/htpasswd` (bcrypt entries, one per line):

```bash
htpasswd -bBc config/htpasswd github_actions <password>
```

Use `-b` (no `-c`) for additional users so you don't overwrite the file. To grant full admin (incl. delete) to a user, add them to `http.accessControl.adminPolicy.users` in `config.json`.

## Config

Edit `config.json` for storage path, TLS, auth, or extensions — see [zot config docs](https://zotregistry.dev/latest/articles/configuration/). Recreate the container after changes:

```bash
docker compose up -d
```

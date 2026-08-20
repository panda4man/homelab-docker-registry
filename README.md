# homelab-docker-registry

Self-hosted OCI registry for my local Docker projects, running [zot](https://github.com/project-zot/zot).

Built as an alternative to GHCR — local infra and storage already available, no need to push images off-box.

## Stack

- `docker-compose.yml` — zot service definition
- `config.json` — zot registry config (storage path, HTTP listener, search/UI extensions)
- `data/` — registry storage (images, metadata) — bind-mounted, gitignored
- `.env` — local port override, gitignored (copy from `.env.example`)

Bind mounts instead of named volumes, so the whole stack — config, data, compose file — lives in this one directory and backs up as a single unit.

## Usage

```bash
cp .env.example .env   # adjust ZOT_PORT if needed
docker compose up -d
```

Registry API + web UI: `http://localhost:${ZOT_PORT:-5050}`

Push/pull:

```bash
docker tag <image> localhost:5050/<name>:<tag>
docker push localhost:5050/<name>:<tag>
```

Registry is plain HTTP — add `localhost:5050` (or your host/port) to your Docker daemon's `insecure-registries` if pushing from another machine.

## Config

Edit `config.json` for storage path, TLS, auth, or extensions — see [zot config docs](https://zotregistry.dev/latest/articles/configuration/). Recreate the container after changes:

```bash
docker compose up -d
```

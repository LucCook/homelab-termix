# Homelab deployment (dev-2.3.2)

This fork tracks [Termix `dev-2.3.2`](https://github.com/Termix-SSH/Termix/tree/dev-2.3.2), which includes merged **host-to-host file transfer**.

## Prerequisites

- Docker and Docker Compose v2 on the server
- Git clone of this repo at the desired tag/commit (e.g. `main` after syncing `upstream/dev-2.3.2`)
- Port **8080** free (or edit `docker-compose.homelab.yml`)

## First deploy or upgrade

From the **repository root**:

```bash
git fetch upstream
git checkout main
git pull origin main   # after you push updated main from your workstation

# Build image (10–20+ minutes on first run; needs ~4GB RAM for frontend build)
docker compose -f docker/docker-compose.homelab.yml build

# Start (or recreate containers after upgrade)
docker compose -f docker/docker-compose.homelab.yml up -d

# Check logs
docker compose -f docker/docker-compose.homelab.yml logs -f termix
```

Open `http://<server>:8080/` and complete initial setup if this is a fresh volume.

## Upgrade workflow

1. **Back up** the data volume (contains SQLite DB and uploads):
   ```bash
   docker run --rm -v termix-data:/data -v "$(pwd)":/backup alpine \
     tar czf /backup/termix-data-backup-$(date +%Y%m%d).tar.gz -C /data .
   ```
2. Pull latest `main` (synced with `upstream/dev-2.3.2`).
3. Rebuild and recreate:
   ```bash
   docker compose -f docker/docker-compose.homelab.yml build --no-cache
   docker compose -f docker/docker-compose.homelab.yml up -d
   ```
4. Smoke-test: login, File Manager, **Copy to host…** between two SSH hosts.

## Alternative: official image

`docker/docker-compose.yml` uses `ghcr.io/lukegus/termix:latest`. That image follows **upstream releases**, not this branch. Use the homelab compose file above until a **2.3.2** (or later) release image includes host-to-host transfer.

## Operator docs

See [readme/HOST-TO-HOST-TRANSFER.md](../readme/HOST-TO-HOST-TRANSFER.md) for transfer behaviour, limits, and troubleshooting.

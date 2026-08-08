# uptime-kuma-stack

Portainer Git stack for Uptime Kuma on `apple-pi.lan`.

- **Stack name in Portainer:** `uptime-kuma`
- **UI:** http://apple-pi.lan:3001
- **Data:** Docker volume `uptime-kuma_uptime-kuma-data` (Kuma 2.x — embedded MariaDB,
  ~30M). Declared `external: true` in the compose **on purpose**.

## The volume trap

Compose prefixes named volumes with the project name. The existing volume is
`uptime-kuma_uptime-kuma-data`, created when the stack was named `uptime-kuma`.
Deploying this as a stack named anything else (e.g. `uptime-kuma-stack`) without the
`external:` declaration would create a new empty volume — Kuma comes up looking freshly
installed, with every monitor and all history gone.

The `external: true` + explicit `name:` in `docker-compose.yml` pins it regardless of
stack name. **Do not remove it.**

## Backups

Not covered by the daily backup jobs (those are portainer/documenso/invoiceninja/homebridge).
Snapshot manually — cold, with the container stopped, since MariaDB is running:

```bash
docker stop uptime-kuma
docker run --rm -v uptime-kuma_uptime-kuma-data:/from:ro -v "$HOME/backups:/to" \
  alpine tar czf /to/uptime-kuma-$(date +%F).tgz -C /from .
docker start uptime-kuma
ssh -p 33 bheussler@nikodrive.lan "cat > uptime-kuma-backups/uptime-kuma-$(date +%F).tgz" \
  < "$HOME/backups/uptime-kuma-$(date +%F).tgz"
```

## Depends on

External network `abraham_abraham-network` (from the `abraham` stack) must exist first.

---
type: note
updated: 2026-10-02
aliases:
  - docker
  - media stack
  - gluetun
---

# Docker media stack

One Compose project (`media-stack`) on the NAS:
`/volume1/docker/projects/media-stack/compose.yaml` (shared `x-logging` anchor
at the top). All configs in `/volume1/docker/<app>`.
PUID **1027**, PGID **65536**, TZ Asia/Dubai.

**NPM is no longer in this project** — it moved to its own on 2 Oct 2026, so
rebuilding the media stack no longer takes down the `*.home` names. See
[[Reverse proxy & certificates]].

## Services
| Container | Network | Reached at |
|---|---|---|
| gluetun (ProtonVPN, WireGuard) | synobridge 172.20.1.10 | control server :8000 |
| qbittorrent | **inside gluetun** | :8080 via gluetun |
| prowlarr | **inside gluetun** | :9696 via gluetun |
| sonarr / radarr | synobridge .21 / .22 | :8989 / :7878 |
| jellyfin | synobridge .23 | :8096 (hardware transcoding via /dev/dri) |
| recyclarr | synobridge | nightly 04:00 |

Networks: **`synobridge` only** (external). The macvlan network that used to be
defined here was removed with NPM.

## How the VPN wiring works
- `network_mode: "service:gluetun"` means the container **has no network of its
  own**: no `networks:`, no `ports:`, no `extra_hosts:` (Docker rejects them).
  Everything is declared on gluetun instead.
- Kill switch is structural: if gluetun stops, those containers have no network.
- `FIREWALL_OUTBOUND_SUBNETS: 192.168.1.0/24,172.16.0.0/12` — without the second
  range, LAN/bridge clients get "connection reset by peer".
- **gluetun replaces Docker's DNS**, so containers inside it cannot resolve other
  container names. Fix: static IPs + `extra_hosts` **on gluetun** (inherited).

## Port forwarding
- `SERVER_COUNTRIES: Netherlands`, `PORT_FORWARD_ONLY: on`.
  **Iceland churned the NAT-PMP port** on every renewal (~45 s); Netherlands holds it.
- gluetun pushes the port into qBittorrent via `VPN_PORT_FORWARDING_UP_COMMAND`:
  `wget -q -O /dev/null --post-data … || true`. Writing to `-O /dev/null` with
  `|| true` matters: with `-O-` wget exits 1 on an empty 200 response and gluetun
  logs a false error.
- Healthy log: one "port forwarded is NNNNN", then silence (only failures are logged).
- Check: `docker exec gluetun cat /tmp/gluetun/forwarded_port` and qBittorrent
  `/api/v2/transfer/info` → `connected`, `dht_nodes` in the dozens+.

## Restart procedure
After **any** rebuild or gluetun restart:
```bash
cd /volume1/docker/projects/media-stack
sudo docker compose up -d --remove-orphans
sudo docker restart qbittorrent prowlarr      # rebind to gluetun's new namespace
```
Wait ~2 minutes before judging qBittorrent's status.

## Quality management
Recyclarr syncs TRaSH Guides profiles: Sonarr **WEB-1080p**, Radarr
**HD Bluray + WEB**, with custom formats and size limits. Don't hand-edit those
profiles — the nightly sync overwrites them; change `recyclarr.yml` instead.

## Monitoring
- HA alerts on qBittorrent **`disconnected`** only. `firewalled` flaps briefly
  during port renewals and on an idle client, so it's shown on the Media tile but
  doesn't page.
- VPN country / IP / forwarded port as REST sensors from gluetun's control server.

## Gotchas
- After any manual gluetun restart, **restart qbittorrent and prowlarr** too.
- `firewalled` with `dht_nodes: 0` right after a restart is normal for a minute or
  two, and on a client with no torrents.
- `/dev/net/tun` must exist for gluetun; DSM drops it on reboot, so a Task
  Scheduler boot task recreates it (runs as root).
- Media (`/volume1/data`) is **excluded** from backups; only configs are kept.
- **Back up `compose.yaml` before editing** — `compose.yaml.bak-YYYYMMDD`
  alongside it; check with `docker compose config --quiet` before `up`.

## To do
- [ ] `depends_on: gluetun: condition: service_healthy` on qbittorrent and
      prowlarr (fixes cold starts after a power cut)
- [ ] Optional `rebuild.sh` wrapping the restart procedure

## Related
[[Reverse proxy & certificates]] · [[Network & IPs]] · [[Home Assistant]] ·
[[NAS (Synology DS224+)]]

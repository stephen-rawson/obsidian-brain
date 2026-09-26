---
type: note
updated: 2026-09-25
---

# Docker media stack

One Compose project (`media-stack`) on the NAS, files under
`/volume1/docker/projects/media-stack`. All configs in `/volume1/docker/<app>`.
PUID **1027**, PGID **65536**, TZ Asia/Dubai.

## Services
| Container | Network | Reached at |
|---|---|---|
| gluetun (ProtonVPN, WireGuard) | synobridge 172.20.1.10 | — |
| qbittorrent | **inside gluetun** | :8080 via gluetun |
| prowlarr | **inside gluetun** | :9696 via gluetun |
| sonarr / radarr | synobridge .21 / .22 | :8989 / :7878 |
| jellyfin | synobridge .23 | :8096 (hardware transcoding via /dev/dri) |
| recyclarr | synobridge | nightly 04:00 |

## How the VPN wiring works
- `network_mode: "service:gluetun"` means the container **has no network of its
  own**: no `networks:`, no `ports:`, no `extra_hosts:` (Docker rejects them).
  Everything is declared on gluetun instead.
- Kill switch is structural: if gluetun stops, those containers have no network.
- `FIREWALL_OUTBOUND_SUBNETS: 192.168.1.0/24,172.16.0.0/12` — without the second
  range, LAN/bridge clients get "connection reset by peer".
- **gluetun replaces Docker's DNS**, so containers inside it cannot resolve other
  container names. Fix: static IPs + `extra_hosts` **on gluetun** (inherited).
- Port forwarding: Proton assigns a new port each session; gluetun's
  `VPN_PORT_FORWARDING_UP_COMMAND` pushes it into qBittorrent via its API.

## Quality management
Recyclarr syncs TRaSH Guides profiles: Sonarr **WEB-1080p**, Radarr
**HD Bluray + WEB**, with custom formats and size limits. Don't hand-edit those
profiles — the nightly sync overwrites them; change `recyclarr.yml` instead.

## Gotchas
- After any manual gluetun restart, **restart qbittorrent and prowlarr** too —
  they are bound to the old network namespace otherwise.
- `/dev/net/tun` must exist for gluetun; DSM drops it on reboot, so a Task
  Scheduler boot task recreates it (runs as root).
- Media (`/volume1/data`) is **excluded** from backups; only configs are kept.

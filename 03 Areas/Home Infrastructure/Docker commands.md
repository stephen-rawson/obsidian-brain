---
type: note
updated: 2026-10-03
aliases:
  - docker cheatsheet
  - docker cli
---

# Docker commands

Day-to-day Docker CLI for the two hosts that run containers. Stack-specific
procedures live in the stack's own note. This is the generic layer only.

## Where Docker runs
| Host | Engine | Prefix | Shell |
|---|---|---|---|
| NAS (DS224+) | Container Manager | `sudo docker …` | bash over SSH |
| Mac Mini | Docker Desktop | `docker …` (no sudo) | zsh over SSH |
| Pi | none. Pi-hole and Unbound are native (`systemctl`) | | |

Commands below are written for the NAS. On the Mini, drop `sudo`.

## Compose projects
| Host | Project | Compose file | Containers |
|---|---|---|---|
| NAS | `media-stack` | `/volume1/docker/projects/media-stack/compose.yaml` | 8: gluetun, qbittorrent, prowlarr, sonarr, radarr, jellyfin, recyclarr, `trawl` (FlareSolverr replacement, inside gluetun). See [[Docker media stack]] |
| NAS | `npm` | `/volume1/docker/projects/npm/compose.yaml` | 1: Nginx Proxy Manager. See [[Reverse proxy & certificates]] |
| NAS | `tools` | `/volume1/docker/projects/tools/compose.yaml` | 4: `paperless`, `paperless-db`, `paperless-redis`, `couchdb` (see [[Paperless]], [[Obsidian & LiveSync]]) |
| Mini | `pihole` | `~/pihole/docker-compose.yml` | Pi-hole secondary, nebula-sync |
| Mini | *none* (bare `docker run`) | none | `unifi`: `jacobalberty/unifi:latest`, MongoDB bundled, `~/unifi` → `/unifi`, `unless-stopped`. See [[Switch & UniFi controller]] |

NAS projects live in `/volume1/docker/projects/<project>/`; app configs in
`/volume1/docker/<app>`.

## Find things
| Task | Command |
|---|---|
| Running containers | `sudo docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"` |
| Include stopped | `sudo docker ps -a --format "table {{.Names}}\t{{.Status}}"` |
| All containers by project | `sudo docker ps -a --format 'table {{.Label "com.docker.compose.project"}}\t{{.Names}}\t{{.Status}}'` |
| Compose projects + file paths | `sudo docker compose ls` |
| One project's services (any folder) | `sudo docker compose -p tools ps` |
| Health of one container | `sudo docker inspect --format '{{.State.Health.Status}}' gluetun` |
| Resource use (snapshot) | `sudo docker stats --no-stream` |

## Logs and debugging
| Task | Command |
|---|---|
| Follow live, last 100 lines | `sudo docker logs -f --tail 100 <name>` |
| Since a time window | `sudo docker logs --since 30m <name>` |
| Shell inside a container | `sudo docker exec -it <name> sh` |
| Validate compose before applying | `sudo docker compose config --quiet` (silent = valid) |

## Change things (run from the project folder)
| Task | Command |
|---|---|
| Apply compose changes | `sudo docker compose up -d --remove-orphans` |
| Restart services | `sudo docker restart <name1> <name2>` |
| Update images | `sudo docker compose pull && sudo docker compose up -d` |
| Stop one service (disable) | `sudo docker compose stop <service>` |
| Remove a service for good | Delete it from `compose.yaml`, then `up -d --remove-orphans` |
| Clear dangling images | `sudo docker image prune` |

Media stack: after any `up`, `pull` or gluetun restart, follow
[[Docker media stack#Restart procedure]].

## Gotchas
- **`sudo` on the NAS, none on the Mini.** Don't "fix" this by adding the DSM
  user to the `docker` group: that group is root-equivalent.
- **Stop before removing.** `docker rm -f` on a compose-managed container is
  undone by the next `up`. On a container created outside compose it is gone for
  good, config included if it had no bind mount.
- **Containers inside gluetun** (qbittorrent, prowlarr) lose their network when
  gluetun restarts. Restart them too.
- **`tools` mixes Paperless and CouchDB.** A bare `up -d` there recreates any
  changed service, so a Paperless edit can bounce CouchDB and pause LiveSync.
  Name the service: `sudo docker compose up -d paperless`.
- **`unifi` on the Mini is a bare `docker run`.** The run command isn't stored
  anywhere, and `:latest` means a recreate can jump UniFi versions (no
  downgrade path for its backups). Don't `docker rm` it until it's in compose.
- **Format strings containing double quotes** (e.g. `.Label "…"`) must be wrapped
  in single quotes in bash.
- **`logs -f` without `--tail`** replays the whole log before following.
- **`image prune -a` and `system prune`** remove images of *stopped* containers
  too. The next `up` re-pulls whatever `latest` is then, which is an unplanned
  upgrade. Plain `image prune` (dangling only) is the safe default.
- **Back up `compose.yaml` before editing** (`compose.yaml.bak-YYYYMMDD`) and run
  `config --quiet` before `up`.
- **Docker Desktop NATs inbound traffic** on the Mini, so services there see
  Docker's gateway, not the real client IP.

## To do
- [x] Fill in NAS project paths (`docker compose ls`, 3 Oct 2026)
- [x] Confirm `tools` contents: Paperless ×3 + CouchDB (3 Oct 2026)
- [x] Identify 8th media-stack container: `trawl` (3 Oct 2026)
- [x] Mini `unifi`: bare `docker run`, healthy, controller answers 302 (3 Oct 2026)
- [ ] Move `unifi` to `~/unifi/compose.yaml` with a pinned image tag (after the trip; at home, not remotely)

## Related
[[Home Infrastructure]] · [[Docker media stack]] · [[Paperless]] ·
[[NAS (Synology DS224+)]] · [[Mac Mini]] · [[Switch & UniFi controller]] ·
[[Obsidian & LiveSync]] · [[Runbook]]

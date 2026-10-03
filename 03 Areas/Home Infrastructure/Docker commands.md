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
| Host | Engine | Prefix | Shell | Projects |
|---|---|---|---|---|
| NAS (DS224+) | Container Manager | `sudo docker …` | bash over SSH | media-stack, NPM, Paperless, CouchDB (paths: run `sudo docker compose ls`) |
| Mac Mini | Docker Desktop | `docker …` (no sudo) | zsh over SSH | `~/pihole` (Pi-hole secondary + nebula-sync), `unifi` |
| Pi | none. Pi-hole and Unbound are native (`systemctl`) | | | |

Commands below are written for the NAS. On the Mini, drop `sudo`.

## Find things
| Task | Command |
|---|---|
| Running containers | `sudo docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"` |
| Include stopped | `sudo docker ps -a --format "table {{.Names}}\t{{.Status}}"` |
| Compose projects + file paths | `sudo docker compose ls` |
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
- **`logs -f` without `--tail`** replays the whole log before following.
- **`image prune -a` and `system prune`** remove images of *stopped* containers
  too. The next `up` re-pulls whatever `latest` is then, which is an unplanned
  upgrade. Plain `image prune` (dangling only) is the safe default.
- **Back up `compose.yaml` before editing** (`compose.yaml.bak-YYYYMMDD`) and run
  `config --quiet` before `up`.
- **Docker Desktop NATs inbound traffic** on the Mini, so services there see
  Docker's gateway, not the real client IP.

## To do
- [ ] Run `sudo docker compose ls` on the NAS and fill in the project paths above
- [ ] Mini: confirm `unifi` is a compose project or a bare `docker run` (affects how to update it)

## Related
[[Home Infrastructure]] · [[Docker media stack]] · [[NAS (Synology DS224+)]] ·
[[Mac Mini]] · [[Runbook]]

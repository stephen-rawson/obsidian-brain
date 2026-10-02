---
type: note
updated: 2026-10-02
aliases: [mini, Mac mini, mini.lan, Mac Mini M1]
---

# Mac Mini

Headless, always-on second server. Wiped and rebuilt as a server on 1–2 Oct
2026. Holds the jobs that need macOS (Proton apps) or that shouldn't share a
failure with the Pi or the NAS.

## Hardware
|         |                                                   |
| ------- | ------------------------------------------------- |
| Model   | Mac mini, Apple **M1**, **8 GB** RAM              |
| Storage | 500 GB SSD (watch it — Bridge keeps a mail cache) |
| macOS   | 27.0.1                                            |
| Power   | PDU on the UPS — see [[UPS & power]]              |

## Network
| | |
|---|---|
| Address | **`192.168.1.15`** · `mini.lan` · reserved by MAC |
| Ethernet MAC | `14:98:77:83:ae:5f` (the reservation) |
| Wi-Fi MAC | `14:98:77:82:f4:59` (unused) |
| Switch | UniFi port 5, GbE, PoE off |
| Tailscale | `100.104.241.74` (`mini`) |
| Hostname | `mini` |

## Access
- **SSH:** `ssh mini` (user `stephen`), key-based from the Lenovo and the MacBook
- **Screen Sharing:** from the MacBook — Finder → Go → Connect to Server →
  `vnc://192.168.1.15`. Windows VNC viewers (RealVNC/TightVNC) fail with
  "unsupported security types" unless the legacy VNC password is enabled
  (Sharing → Screen Sharing ⓘ → "VNC viewers may control screen with password",
  ≤ 8 characters, LAN/Tailscale only)
- `~/.zshrc` has `setopt interactivecomments` so pasted commands with `#`
  comments work

## Headless configuration
- **Auto-login** as `stephen` — required so the Proton apps, Docker Desktop and
  launchd agents start after a reboot
- **FileVault off** — with it on, the Mini waits at an unlock screen after a power cut
- Energy: never sleep, **start up after power failure**, wake for network access
- Screen saver off; no password after screen saver
- Automatic security updates on; **automatic macOS upgrades off**
- Login Items: Proton Drive, Proton Mail Bridge, Docker Desktop

## What runs here
| Service | How | Notes |
|---|---|---|
| **UniFi Network controller** | Docker container `unifi` | `https://switch.home.stephenrawson.uk` (NPM) / `:8443`; switch informs on `:8080` direct |
| **Pi-hole (secondary DNS)** | Docker, `~/pihole/docker-compose.yml` | DNS `:53`, web `:8081`; second DNS server in router DHCP |
| **nebula-sync** | same compose project | hourly: gravity + `.lan` records from the Pi. Wildcard pinned via `FTLCONF_misc_dnsmasq_lines` (API can't write it) |
| **Ollama** | `brew services` | small models only (8 GB); `OLLAMA_HOST` not yet exposed |
| **Proton Drive** | app, login item | target for the off-site backup |
| **Proton Mail Bridge** | app, login item | IMAP on `127.0.0.1:1143` for Paperless email |
| **Off-site shipping** | launchd `com.stephen.offsite-ship` | 03:00 + at login — [[Off-site backup (Proton Drive)]] |
| **Paperless email intake** | launchd `com.stephen.paperless-mail` | every 15 min — [[Paperless]] |
| **Tailscale** | CLI daemon (`brew services`) | on the tailnet |

## Files and secrets
| Path | What |
|---|---|
| `~/bin/offsite-ship.sh` | off-site shipping script |
| `~/bin/paperless-mail.py` | email intake script (`--dry-run`, `--list`, `--mark-existing`) |
| `~/Library/LaunchAgents/com.stephen.*.plist` | the two schedules (paperless-mail runs via `/bin/bash`) |
| `~/pihole/docker-compose.yml`, `~/pihole/.env` | secondary Pi-hole; `.env` holds both Pi-hole passwords (mode 600) |
| `~/.config/paperless-mail/bridge-pass` | Bridge password (mode 600) |
| `~/Library/Application Support/paperless-mail/filed.json` | emails already filed |
| `~/Library/Logs/offsite-ship.log`, `paperless-mail.log` | logs |
| Keychain | SMB password for `mini@192.168.1.42` |

**SMB:** the Mini connects to the NAS as one DSM user, **`mini`** — read-only on
`offsite-staging`, read/write on `paperless-consume`, nothing else.
**Privacy:** **Full Disk Access granted to `/bin/bash`**; both launchd jobs run
through bash so they inherit it.

## Health check
```bash
ssh mini
pgrep -lf "Proton Drive"; pgrep -lf "Proton Mail Bridge"
launchctl list | grep com.stephen
tail -2 ~/Library/Logs/offsite-ship.log ~/Library/Logs/paperless-mail.log
docker ps --format '{{.Names}}\t{{.Status}}'
brew services list | grep -E "ollama|tailscale"
mount | grep 192.168.1.42
```
Run a job now: `launchctl kickstart -k gui/$(id -u)/com.stephen.offsite-ship`

## Gotchas
- **Keychain is locked to SSH sessions** — `security` exits **36** ("user
  interaction is not allowed"); exit **44** means "item not found". Hence the
  Bridge password file. Unlock for a session with
  `security unlock-keychain ~/Library/Keychains/login.keychain-db`
- **One SMB login per server** on macOS — a second share mounted "as another
  user" silently reuses the first login. One account per machine
- **Leftover `/Volumes/<share>` folders** push mounts to `…-1`; scripts check for
  a real mount
- **Network volumes need privacy permission** for background jobs — hence
  Full Disk Access for bash
- **zsh treats `#` literally** without `interactivecomments` — comments get
  passed as arguments
- **Docker Desktop NATs inbound traffic**, so the secondary Pi-hole logs every
  client as Docker's gateway. DNS works; use the Pi for per-client logs
- **Port 8080 is the UniFi inform port** — Pi-hole's web UI is on 8081
- **`launchctl setenv` doesn't survive a reboot** — put env vars in the launch
  agent's plist
- **Reserve inside the infrastructure range** — `.43` (inside the DHCP pool) was
  already leased to another device, so the Mini's first reservation silently failed
- **Fresh macOS installs can be slow to see DFS Wi-Fi channels** — Ethernet for setup
- **Bridge's first sync** must finish before labels appear over IMAP

## Not backed up (yet)
Everything important here is either re-creatable or a copy of something
elsewhere — **except**: the two scripts, the plists, `~/pihole/docker-compose.yml`
and the UniFi controller's autobackups.
- [ ] Copy scripts + plists + compose into this vault (or the GitHub repo)
- [ ] Copy UniFi autobackups (`~/unifi/data/backup/autobackup`) to the NAS

## To do
- [x] Fill in SSD size and macOS version
- [ ] HDMI dummy plug (proper resolution for Screen Sharing)
- [ ] HA: Ping sensor for `.15` + DNS check of the secondary Pi-hole in `systems_issues`
- [x] Tailscale: confirm second subnet router (`--advertise-routes=192.168.1.0/24`) for failover with the Pi
- [ ] Optional: NUT client so it shuts down with the NAS on a long outage
- [ ] Optional: Ollama on the LAN (`OLLAMA_HOST=0.0.0.0` in the plist), Open WebUI
- [ ] Candidates: Speedtest Tracker, Uptime Kuma

## Related
[[Home Infrastructure]] · [[Network & IPs]] · [[Switch & UniFi controller]] ·
[[Pi (Pi-hole & Unbound)]] · [[Off-site backup (Proton Drive)]] · [[Paperless]] ·
[[Tailscale]] · [[UPS & power]]

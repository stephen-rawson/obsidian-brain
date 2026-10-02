---
type: note
updated: 2026-10-02
aliases:
  - npm
  - Nginx Proxy Manager
  - reverse proxy
---

# Reverse proxy & certificates

**Nginx Proxy Manager** on the NAS, in **its own Compose project**, on its own
LAN address via macvlan. Every `*.home.stephenrawson.uk` name goes through it.

- **Admin:** http://192.168.1.250:81 · `npm.home.stephenrawson.uk`
- Credentials → Bitwarden: "Nginx Proxy Manager"; Cloudflare API token → Bitwarden
- Config and certs: `/volume1/docker/npm/{data,letsencrypt}` — in Hyper Backup and
  the weekly off-site archive

## Deployment
| | |
|---|---|
| Project | `/volume1/docker/projects/npm/compose.yaml` (separate from the media stack since 2 Oct 2026) |
| Image | `jc21/nginx-proxy-manager:latest` — [ ] pin a version |
| LAN | `192.168.1.250` on external macvlan network **`npm_lan`** |
| Bridge | `172.20.1.40` on `synobridge` (reaches containers by IP/name) |
| MAC | pinned `02:42:c0:a8:01:fa` (works because `lan` is listed first) |

`npm_lan` was created outside Compose so no project owns it:
```bash
sudo docker network create -d macvlan \
  --subnet=192.168.1.0/24 --gateway=192.168.1.1 \
  --ip-range=192.168.1.250/32 -o parent=eth0 npm_lan
```
`--ip-range …/32` means Docker can only ever use `.250`.

## Why macvlan
DSM owns ports 80/443 on the NAS, so NPM needs its own IP (`.250`, outside the
DHCP pool). It is also on `synobridge` so it can reach containers directly.

## Certificates
Wildcard `*.home.stephenrawson.uk` from Let's Encrypt via **DNS challenge**
(Cloudflare API token, scoped to DNS edit on the zone). ECDSA key. Auto-renews;
HA alerts at < 14 days. DNS challenge is required: no public A records, no
inbound ports, and only DNS challenges can issue wildcards. Only the wildcard
appears in CT logs, not each hostname.

## Proxy hosts
| Host | Upstream | Notes |
|---|---|---|
| jellyfin | `jellyfin:8096` (172.20.1.23) | |
| sonarr / radarr | `.21:8989` / `.22:7878` | |
| prowlarr | `gluetun:9696` | inside the VPN namespace |
| qbit | `gluetun:8080` | inside the VPN namespace |
| nas | **https** `172.20.0.1:5001` | not `192.168.1.42` (macvlan) |
| router | **https** `192.168.1.1:8001` | SRM returns 400 on http |
| pihole | Pi `:8080` | |
| ha | `192.168.1.119:8123` | HA trusts `.250` and `172.20.1.40` as proxies |
| couchdb | `172.20.1.41:5984` | Obsidian LiveSync |
| paperless | `172.20.1.42:8000` | `client_max_body_size 32M` |
| switch (+ unifi) | **https** `192.168.1.15:8443` | UniFi controller on the Mac Mini; websockets on |
| zigbee | http `192.168.1.11:80` | SMLIGHT web UI only |
| npm | `127.0.0.1:81` | |

All: Force SSL, HTTP/2, HSTS, websockets where needed. Admin-type hosts sit
behind the **lan-only** access list (192.168.1.0/24 + 100.64.0.0/10 tailnet).

## Name resolution
Wildcard `address=/home.stephenrawson.uk/192.168.1.250` — set in **two places**:
- Pi (primary Pi-hole): `misc.dnsmasq_lines`
- Mac Mini (secondary): `FTLCONF_misc_dnsmasq_lines` in `~/pihole/compose`
  (Pi-hole's API refuses this setting, so nebula-sync can't copy it)

A new service therefore needs **only** a proxy host. If NPM's address ever
changes, update **both** Pi-holes.

## Working on NPM
Restarting NPM takes down **every** `*.home` name — including the DSM page you
might be using to restart it. For NPM work use **`https://192.168.1.42:5001`**
directly, or SSH:
```bash
cd /volume1/docker/projects/npm
sudo docker compose up -d          # apply changes
sudo docker logs npm --tail 20
```
Restarting the media stack no longer affects NPM.

## Gotchas
- **Upstream NAS**: `172.20.0.1:5001`, not `192.168.1.42` — macvlan can't reach
  its own host (that was the 502). Same reason the NAS can't use `.250` itself.
- **Upstream router**: scheme **https** on 8001, or SRM returns 400.
- **Browsing to the proxy by IP** returns a 7-byte TLS alert and closes: no SNI,
  no matching host. Correct behaviour.
- **UniFi:** proxy only the UI (8443). The switch's **inform** traffic (8080)
  goes direct to the Mini — never point the inform host at the proxy name. If
  login fails through the proxy, add `proxy_set_header Origin "";` in Advanced.
- **Zigbee:** proxy only the web UI. TCP **6638** (Zigbee2MQTT ↔ coordinator) is
  raw TCP and stays direct.
- **The macvlan network used to live inside the media-stack project**
  (`media-stack_lan`), so rebuilding the media stack restarted NPM. Now external.

## Rebuild from scratch
1. Restore `/volume1/docker/npm` (Hyper Backup or off-site archive)
2. Create `npm_lan` (command above); `synobridge` must exist
3. `docker compose up -d` in `projects/npm`
4. Check a proxy host and the certificate expiry

## Related
[[Network & IPs]] · [[Docker media stack]] · [[Pi (Pi-hole & Unbound)]] ·
[[Mac Mini]] · [[Switch & UniFi controller]] · [[Zigbee]]

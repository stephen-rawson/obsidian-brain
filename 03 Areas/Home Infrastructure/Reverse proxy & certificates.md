---
type: note
updated: 2026-09-25
aliases:
  - npm
---

# Reverse proxy & certificates

**Nginx Proxy Manager** on the NAS, on its own LAN address via macvlan.

- **Admin:** http://192.168.1.250:81 · `npm.home.stephenrawson.uk`
- Credentials → Bitwarden: "Nginx Proxy Manager"
- Config and certs: `/volume1/docker/npm/{data,letsencrypt}` (in Hyper Backup)

## Why macvlan
DSM owns ports 80/443 on the NAS, so NPM needs its own IP (192.168.1.250,
outside the DHCP pool). It is also attached to `synobridge` (172.20.1.40) so it
can reach containers by name.

## Certificates
Wildcard `*.home.stephenrawson.uk` from Let's Encrypt via **DNS challenge**
(Cloudflare API token → Bitwarden). ECDSA key. Auto-renews.
DNS challenge is required: no public A records, no inbound ports, and only DNS
challenges can issue wildcards. Only the wildcard appears in CT logs, not each
hostname.

## Proxy hosts
jellyfin · sonarr · radarr · prowlarr (→ gluetun:9696) · qbit (→ gluetun:8080) ·
nas · pihole · router · ha · npm. All: Force SSL, HTTP/2, websockets where needed.
Admin-type hosts sit behind an access list (LAN + tailnet ranges).

## Name resolution
Pi-hole wildcard: `address=/home.stephenrawson.uk/192.168.1.250`. Adding a new
service therefore needs **only** a proxy host — no DNS change.

## Gotchas
- **Upstream NAS**: use `172.20.0.1:5001`, not 192.168.1.42 — macvlan can't reach
  its own host (that was the 502).
- **Upstream router**: scheme **https** on 8001, or SRM returns 400.
- Browsing to the proxy **by IP** returns a 7-byte TLS alert and closes: no SNI,
  so no matching host. That is correct behaviour, not a fault.

## Credentials
- Cloudflare API token --> bitwarden
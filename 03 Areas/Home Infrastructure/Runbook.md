---
type: note
updated: 2026-09-25
---

# Runbook — symptom first

## No internet anywhere
1. Wired machine: `ping 8.8.8.8`. Works → DNS problem (next section).
2. Synology Network Center → Status: WAN connected? IP 192.168.70.x?
3. du box lights: PON solid, LOS off. Red LOS = fibre fault → call du.
4. Power-cycle du box, wait 3 min, then the router.

## Names don't resolve (internet works by IP)
1. `nslookup google.com 192.168.1.31` — no answer → Pi-hole is down.
2. On the Pi: `systemctl status pihole-FTL unbound`.
3. `dig pi-hole.net @127.0.0.1 -p 5335` tests Unbound directly.
4. Remember the laptop VPN captures DNS — disconnect it before debugging.

## A site or app misbehaves
Pi-hole → Query Log → filter by the device IP → look for red entries at the time
of failure → allow the specific domain. Don't disable whole lists.

## Remote access broken
See the checks in [[Tailscale]].

## A proxied name returns 502 / 400
- 502 to the NAS: upstream must be `172.20.0.1`, not 192.168.1.42.
- 400 from the router: scheme must be https on 8001.
- 400 from HA: trusted proxies missing (Settings → System → Network).

## NAS degraded / drive failed
1. **Back up before rebuilding** — the remaining drive is the only copy.
2. Storage Manager → pool status; note which bay (DSM numbering, not `/dev/sata*`).
3. Replace like-for-like (4 TB CMR), Repair, leave it alone for ~6–10 h.
4. Afterwards: data scrubbing, and check notifications actually fire.

## Torrents slow or not connecting
1. `docker exec gluetun wget -qO- https://ifconfig.me/ip` — must not be the home IP.
2. Forwarded port in qBittorrent matches `docker exec gluetun cat /tmp/gluetun/forwarded_port`.
3. After any gluetun restart, restart qbittorrent (and prowlarr).

## Power event
Check: NAS improper-shutdown warnings, HA "restarted" push, Pi uptime.
Root cause for the Sept 2026 failures was an unprotected power strip — the UPS
fixes this once installed.

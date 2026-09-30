---
type: note
updated: 2026-09-25
aliases:
  - pi
---

# Raspberry Pi — Pi-hole, Unbound, Tailscale subnet router

- **Address:** 192.168.1.31 · UI `http://192.168.1.31:8080/admin` ·
  `pihole.home.stephenrawson.uk`
- **Login:** app password → Bitwarden: "Pi-hole"
- SSH by key (`ssh pi`); passwordless sudo is the Raspberry Pi OS default.

## Roles
1. **DNS + ad blocking** for the whole LAN (handed out by the router's DHCP).
2. **Recursive resolver** via Unbound on 127.0.0.1#5335 — no third-party DNS
   provider sees all lookups; DNSSEC validated locally.
3. **Local DNS records**, incl. wildcard `address=/home.stephenrawson.uk/192.168.1.250`.
4. **Tailscale subnet router** for 192.168.1.0/24 — see [[Tailscale]].

## Settings that matter
- **Web UI port is 8080**, not 80.
- `dns.listeningMode = ALL` — required so the tailnet can use Pi-hole. Safe:
  nothing is exposed to the internet.
- Blocklists: **Hagezi Multi Pro + Hagezi TIF (Adblock format)**. Old Firebog
  lists disabled. Adblock-syntax entries cover subdomains, so the domain count
  looks low compared with hosts-format lists — that's expected.
- Upstream: only `127.0.0.1#5335`. Never point it at the router.
- `unbound-resolvconf.service` disabled (it hijacks resolv.conf).

## Backups
Weekly cron `/usr/local/bin/pihole-backup.sh` → NAS `backups/pihole/`:
Teleporter zip, `pihole.toml`, Unbound conf, plus a monthly `dd` image of the
boot media. Retention 8 weeks / ~3 images.

## Gotchas
- Pi-hole's own blocklists can break things; check the query log filtered by the
  client's IP before blaming the network (e.g. `securemetrics.apple.com`).
- iCloud Private Relay domains are blocked by design.

## Update
```
update manually
sudo /usr/local/bin/pihole-backup.sh     # backup first
pihole -up
sudo apt update && sudo apt full-upgrade
sudo reboot
```
## Links
- [Video](https://www.youtube.com/watch?v=cE21YjuaB6o&ab_channel=CrosstalkSolutions)
- [Unbound](https://docs.pi-hole.net/guides/dns/unbound/)
---
type: area
updated: 2026-09-25
---
# Home Infrastructure

Hub note for the flat's network, servers and self-hosted services.
**Secrets live in Bitwarden, never here.**

## Components
- [[Network & IPs]] — addressing plan, reservations, Wi-Fi
- [[Router (Synology RT6600ax)]] — SRM config, DHCP, wireless
- [[du fibre line]] — ISP account, ONT, double NAT
- [[NAS (Synology DS224+)]] — storage, RAID, DSM services
- [[Pi (Pi-hole & Unbound)]] — DNS, blocking, subnet router
- [[Home Assistant]] — automations, alerts, dashboards
- [[Docker media stack]] — gluetun, qBittorrent, *arrs, Jellyfin
- [[Reverse proxy & certificates]] — NPM, wildcard cert, proxy hosts
- [[Tailscale]] — remote access
- [[Backups]] — what is backed up, where, and how to restore
- [[Runbook]] — symptom-first troubleshooting
- [[Hardware & purchases]] — kit owned, warranties, pending

## Design principles
- One SSID, one LAN; everything behind the Synology router, not the du box.
- Anything reachable remotely goes via Tailscale. **No port forwards, ever.**
- Names, not IPs: `*.home.stephenrawson.uk` via NPM.
- Config in bind mounts under `/volume1/docker/<app>`, backed up by Hyper Backup.
- Credentials in Bitwarden; recovery keys also in Proton Drive.

## Open items
- [ ] Cabinet build, then UPS install and NUT integration
- [ ] Rewire wall drops
- [ ] Off-site [[Backups]] leg (Backblaze B2 or Synology C2)
- [ ] [[du fibre line]]: bridge mode or DMZ to remove double NAT
- [ ] Migrate SRM support on [[Home Assistant]]

## Notes from daily log
```dataview
LIST
FROM "01 Daily"
WHERE contains(file.outlinks, this.file.link)
SORT file.name DESC
LIMIT 10
```

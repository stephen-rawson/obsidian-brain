---
type: area
updated: 2026-09-25
---
# Home Infrastructure

Hub note for the flat's network, servers and self-hosted services.
**Secrets live in Bitwarden, never here.**

## Components
```dataviewjs
const folder = dv.current().file.folder + "/" + dv.current().file.name; dv.list(dv.pages(`"${folder}"`) .sort(p => p.file.name) .map(p => p.file.link));
```

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
LIST FROM [[]] AND "01 Daily" SORT file.name DESC LIMIT 10
```

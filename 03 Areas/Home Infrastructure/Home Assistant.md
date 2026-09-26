---
type: note
updated: 2026-09-25
---

# Home Assistant

- **Address:** 192.168.1.119:8123 · `ha.home.stephenrawson.uk`
- Runs on its own hardware (deliberately **not** on the NAS — blast radius).
- Backups: daily, kept ~7, written to the NAS share `ha-backups` via the
  Synology DSM integration. **Encryption key → Bitwarden.**

## Integrations
- Synology DSM (NAS health), Synology SRM (device tracking, legacy platform),
- Pi-hole, Sonarr, Radarr, qBittorrent, Jellyfin, Ping (×5), System Monitor,
- REST sensors for gluetun (VPN IP, country, forwarded port).

## Structure
- **Dashboard** `dashboard-home`: Controls, Indicators (air + systems gauges),
  Lights; subviews `air-quality` and `systems`.
- **Template sensors**: `sensor.systems_issues` evaluates every health rule and
  exposes the list; `systems_health` (gauge), `systems_issues_text`,
  `binary_sensor.systems_ok`, `updates_pending`, purifier filter timers.
- **Presence** runs on `zone.home` (count of people), fed by Companion app
  trackers for both phones, so it is not tied to one person or the router.
- **Notifications** go to `notify.mobile_app_sr_mobile_app`.

## Alerting
- Systems problem (5 min) + 4-hourly reminder + all-clear · NAS storage critical
- (bypasses Do Not Disturb) · HA restarted · backup failed · weekly updates digest ·
- Sonarr/Radarr imports via webhook · purifier filters due · daily 08:00 summary.

## Gotchas
- From HA **2026.8** the `http:` block moved to Settings → System → Network.
  Trusted proxies for NPM: 192.168.1.250 and 172.20.1.40. Without it the proxy
  returns 400.
- Manual "Run" on an automation skips the trigger, so templates using
  `trigger.to_state` error — that is not a fault.

## References
- webhook IDs: sonarr-awdgf2w34t, radarr-84wertrg
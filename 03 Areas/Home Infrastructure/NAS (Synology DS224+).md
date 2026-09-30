---
type: note
updated: 2026-09-25
aliases:
  - nas
---

# NAS — Synology DS224+ ("Rayphen_Storage")

- **Address:** https://192.168.1.42:5001 · `nas.home.stephenrawson.uk`
- **Users:** personal (non-admin) for daily file access, admin for management,
  plus service accounts: `ha-user` (Home Assistant), `pi-backup` (Pi-hole backups).
  Credentials → Bitwarden.

## Storage
- **RAID 1**, 2 × 4 TB. Drive 1 failed Sept 2026 (WD Red Plus, dropped out after a
  power event; undetected for ~5 weeks). Replaced with a **Seagate IronWolf 4 TB
  (ST4000VN006)** and rebuilt. Old WD to be tested and RMA'd (3-yr warranty).
- Deliberately mixed brands so both drives don't age identically.
- Volume 1, Btrfs, ~400 GB used of 3.6 TB.

## Scheduled maintenance
| Task | Cadence |
|---|---|
| Data scrubbing | quarterly |
| SMART quick test | monthly |
| SMART extended test | quarterly |
| Hyper Backup → USB SSD | weekly + monthly integrity check |
| DSM config export | monthly |

## Key settings
- Notifications on (email + DS finder push) — this is what was missing when the
  array sat degraded for five weeks.
- Auto-restart after power loss: on. Fan: quiet mode.
- Bind mounts for all containers under `/volume1/docker/<app>`.
- Shares: `backups`, `ha-backups`, `docker`, `data` (media + torrents).

## Gotchas
- Container Manager does **not** create host folders for bind mounts — `mkdir`
  and `chown` first, or the container fails to start.
- DSM's own web server owns ports 80/443, which is why NPM runs on macvlan.
- No `unzip` on DSM; use `python3 -m zipfile -e`.

## Links
- [Software](https://global.synologydownload.com/download/Document/Hardware/HIG/DiskStation/24-year/DS224%2B/enu/DS224p_HIG_enu.pdf)
- [Hardware](https://global.synologydownload.com/download/Document/Software/UserGuide/Os/DSM/7.2/enu/Syno_UsersGuide_NAServer_7_2_enu.pdf)
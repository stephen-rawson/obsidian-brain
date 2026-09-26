---
type: note
updated: 2026-09-26
---

# Off-site backup — Proton Drive

Proton Drive has no NAS-side integration (Hyper Backup doesn't support it, and
there's no WebDAV), so the copy runs **through the laptop**, which stays in the
flat and already syncs Proton Drive.

## Script
`C:\Users\steph\Scripts\001 Jobs\nas-to-proton.ps1`, Task Scheduler,
**daily 02:00 + at log on**, interactive logon type (the account uses a Windows
PIN, so "run whether logged on or not" isn't available — and Proton Drive only
syncs while logged in anyway).

**Destination:** `Proton Drive\stephenrawson\My files\05 Reference\
08 Home Network and Technology\999 Backups`

| What | From | Frequency |
|---|---|---|
| Newest Bitwarden export | `\\192.168.1.42\bw-backups` | daily |
| Newest router `.dss` | `\\192.168.1.42\router-backups` | daily |
| Newest HA backup `.tar` | `\\192.168.1.42\ha-backups\backups` | daily |
| Pi-hole configs (mirror, `.img.gz` excluded) | `\\192.168.1.42\pi-backups` | daily |
| Encrypted 7z of the whole `docker` share | `\\192.168.1.42\docker` | Sundays, 4 kept |

- The docker archive uses **7-Zip with `-mhe=on`** (contents and filenames
  encrypted); passphrase → Bitwarden. Caches, logs, MediaCover and Jellyfin
  metadata are excluded as regenerable.
- It is an **unclean copy**: containers keep running, so SQLite databases could
  be captured mid-write. The Hyper Backup copy on the USB SSD is the
  authoritative one for those.
- `_last-run.txt` is written on every run as a heartbeat.

## Scope
Covers configuration, credentials and personal data — everything needed to
rebuild. **Does not cover** `data` (media and torrents), which is re-obtainable.
A cloud leg of Hyper Backup (Backblaze B2 or Synology C2, configs and homes only,
client-side encrypted) remains the proper long-term answer.

## To do
- [ ] HA alert if `_last-run.txt` is older than 3 days
- [ ] Verify the task runs overnight with the laptop locked
- [ ] Test-restore the docker 7z to a scratch folder

## Related
[[Backups]] · [[Bitwarden backup]] · [[NAS (Synology DS224+)]]

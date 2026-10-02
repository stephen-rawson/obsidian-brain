---
type: note
updated: 2026-10-02
aliases: [off-site backup, Proton backup, offsite, offsite-staging]
---

# Off-site backup — Proton Drive

Proton Drive has no NAS-side integration (no Hyper Backup target, no WebDAV),
so the copy goes **NAS → Mac Mini → Proton Drive**. Since 2 Oct 2026 the laptop
is no longer in the chain.

## Design: the NAS prepares, the Mini ships
| Time | Where | What |
|---|---|---|
| **01:30** | NAS, as **root** | Builds the off-site set in the `offsite-staging` share |
| **03:00** (+ at login) | Mac Mini | Mounts the share read-only, copies it into Proton Drive, pings HA |

Why split it: root on the NAS can read everything (Paperless's database dir
included), the 7z is built locally rather than over the network, and the Mini
only ever gets **read-only** access to one share.

## What is shipped
| What | Source | Frequency | Kept |
|---|---|---|---|
| Newest Bitwarden export (encrypted JSON, password-protected) | `bw-backups` | daily | 14 d staging / 90 d Proton |
| Newest router `.dss` | `router-backups` | daily (export is **manual**) | 14 d / 90 d |
| Newest HA backup `.tar` | `ha-backups/backups` | daily | 14 d / 90 d |
| Pi-hole configs (`.img.gz` excluded) | `pi-backups` | daily mirror | — |
| Paperless export (originals + manifest) | `docker/paperless/export` | daily mirror | — |
| **Encrypted 7z of the `docker` share** | `docker` | **Sundays** | 4 staging / 35 d Proton |

- 7z with `-mhe=on` (contents and filenames encrypted). Caches, logs, MediaCover,
  Jellyfin metadata, Paperless media/thumbnails excluded. If DSM lacks `7z` the
  script falls back to `tar | openssl enc -aes-256-cbc -pbkdf2`.
- **Unclean copy:** containers keep running, so SQLite databases may be caught
  mid-write. Hyper Backup on the USB SSD is authoritative for those.

## NAS side
- Script: `/root/offsite/nas-offsite-stage.sh` (root-only, mode 700)
- Passphrase: `/root/offsite/pass` (mode 600) — same as Bitwarden "NAS 7zip Password"
- DSM Task Scheduler: **"Offsite staging"**, user **root**, daily 01:30,
  email on abnormal termination
- Output: `/volume1/offsite-staging/` → `latest/`, `pi-backups/`,
  `paperless-export/`, `archives/`, `_staged.txt` (marker), `_stage.log`
- Copies with `cp --preserve=timestamps` and ends with `chmod -R go+rX` so the
  read-only account can read everything (share-level permissions still restrict
  who can open the share)
- **`/root` is not in any backup** — a copy of the script lives in this vault

## Mac Mini side
- Script: `~/bin/offsite-ship.sh`; launchd `com.stephen.offsite-ship`
  (03:00 daily + RunAtLoad)
- Mounts `smb://mini@192.168.1.42/offsite-staging` (password in Keychain);
  checks for a **real mount**, not just a folder in `/Volumes`
- Refuses to ship if `_staged.txt` is missing
- `rsync -rt` into `~/Library/CloudStorage/ProtonDrive-*/NAS Backups/`,
  **no `--delete`** — damage on the NAS side never propagates to Proton
- Retention on the Proton side: archives > 35 days, other files > 90 days
- Heartbeat: POST to HA webhook `proton-backup-aaerg23356` →
  `input_datetime.proton_backup_last_run` (alert if > 3 days old)
- Log: `~/Library/Logs/offsite-ship.log`
- Needs **Full Disk Access for `/bin/bash`** (writing into the Proton folder)
- Proton Drive app: launch at login; the Mini auto-logs-in

## Restore
1. Download the file from Proton Drive (web or app) → `NAS Backups/...`
2. Docker archive: `7z x docker-YYYYMMDD.7z` → passphrase from Bitwarden.
   Fallback format: `openssl enc -d -aes-256-cbc -pbkdf2 -in docker-…tar.gz.enc | tar xz`
3. Bitwarden export: import into Bitwarden with the export password
4. HA `.tar`: upload in HA onboarding / Settings → System → Backups
5. Router `.dss`: SRM → Backup & Restore → Restore

## Scope
Covers configuration, credentials and personal data — everything needed to
rebuild. **Does not cover** `data` (media and torrents, re-obtainable) or
future photos. A Hyper Backup cloud leg (Backblaze B2 / Synology C2,
client-side encrypted) remains the long-term answer, and is required before
Immich.

## Verified
- 2 Oct 2026: staging run clean (`7z exit 0`, ~90 s); archive tested with `7z t`
- 2 Oct 2026: Mini manual run, launchd run and post-reboot run all `ship done`

## Gotchas
- **Permission denied / rsync exit 23** — `cp -p` copied the Bitwarden export's
  `600` mode into staging. Fixed with `--preserve=timestamps` + `chmod -R go+rX`
- **A Mac keeps one SMB login per server.** Mounting `paperless-consume` as a
  second user silently reused the `offsite` login. One account per machine:
  **`mini`** (read-only on `offsite-staging`, read/write on `paperless-consume`)
- **Leftover folders in `/Volumes`** make macOS mount at `…-1`; scripts must test
  for a real mount (`mount | grep " on $MNT ("`), not `-d`
- **`scp` to DSM needs `-O`** when its SFTP service is off
- **`grep` in `/root` needs `sudo`** — the files are root-only by design
- **Router backup is manual** — the staged `.dss` went stale for a week. Export
  after any router change

## To do
- [ ] **~9 Oct:** disable the Lenovo's old "NAS to Proton Drive" task
- [ ] Delete the old laptop destination folder in Proton once the new one has a month of history
- [ ] Delete the retired `offsite` DSM user (disabled 2 Oct)
- [ ] Download a Sunday archive from Proton and test-restore it on the MacBook
- [ ] Monthly reminder: export the router `.dss`
- [ ] Hyper Backup cloud leg (B2/C2)

## Related
[[Backups]] · [[Bitwarden backup]] · [[NAS (Synology DS224+)]] · [[Mac Mini]] ·
[[Paperless]]

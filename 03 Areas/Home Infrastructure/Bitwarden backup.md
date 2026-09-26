---
type: note
updated: 2026-09-26
---

# Bitwarden backup

Bitwarden's cloud is the primary vault; this is the escape hatch. **Vaultwarden
was considered and rejected**: a password manager is the one service where the
provider's uptime and security team beat self-hosting, and putting the vault on
the NAS creates a circular dependency (the keys needed to restore the NAS would
live on the NAS).

## Automated export
- **Script:** `/volume1/docker/bw/bw-backup.sh` on the NAS, run by **DSM Task
  Scheduler**, weekly Sunday 04:00, as root.
- **Tooling:** official Bitwarden CLI at `/volume1/docker/bw/bw`, logging in with
  an **API key** (client id/secret) so 2FA isn't prompted.
- **Secrets:** `/volume1/docker/bw/.env`, root-only, mode 600 — contains the API
  key, master password and export passphrase. This is the highest-value file in
  the estate; keep it out of any cloud backup leg.
- **Output:** `bw-backups` share, `bitwarden-YYYYMMDD.json`, encrypted with a
  separate export passphrase, 90-day retention.

## Copies
1. NAS `bw-backups` share
2. USB SSD via Hyper Backup
3. Proton Drive via the nightly laptop script — see [[Backups]]

## Restore
Import into any Bitwarden account: Tools → Import → "Bitwarden (json)" → enter
the export passphrase. **Tested once** into a throwaway account. Retest yearly.

## Not covered
File attachments and Sends are not included in exports. Emergency access for
Raya is configured in the web vault and is the most important recovery path.

## Related
[[Backups]] · [[NAS (Synology DS224+)]]

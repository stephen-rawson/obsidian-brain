---
type: note
updated: 2026-10-02
aliases: [Paperless-ngx, documents, document archive]
---

# Paperless

Document archive for the household. OCRs everything on ingest, so scanned
paper becomes full-text searchable. **The point is retrieval, not storage** —
if you would never search for it, it does not belong here.

- **Address:** `https://paperless.home.stephenrawson.uk` via [[Reverse proxy & certificates]]
- **Login and API token** → Bitwarden ("Paperless", "Paperless API token")
- **Containers:** `paperless`, `paperless-db` (Postgres), `paperless-redis`,
  in the `tools` project on the NAS. Config under `/volume1/docker/paperless`
- **Runs as** `dockerlimited` / `dockergroup` (PUID 1027) inside the container
- **Version:** v3.2.1 (matters — see Gotchas)

## What goes in, and what does not
**In:** tenancy and property, banking, insurance, vehicle, identity, medical,
purchases and warranties, utilities, travel.

**Out, deliberately:**
- Anything belonging to [[Oliver Wyman]] or a client — corporate kit only.
  (Personal mail that arrives at the work address can be forwarded in.)
- IRS / US tax correspondence
- Credentials, recovery keys, key material — those live in Bitwarden
- Receipts too trivial to ever search for

## Getting documents in
| Route | How |
|---|---|
| **Laptop** | `P:` mapped to `\\192.168.1.42\paperless-consume`; drop files in |
| **Subfolder = tag** | `P:\banking\`, `P:\property\`, `P:\health\` etc. `PAPERLESS_CONSUMER_RECURSIVE` and `..._SUBDIRS_AS_TAGS` are on |
| **Phone** | Swift Paperless (iOS, third party), same URL. Works over [[Tailscale]] |
| **Email** | Send or forward to **`paperless.happiness330@passmail.net`** (Proton Pass alias). Filed within 15 min, tagged `email` — see below |

Files are consumed within seconds and **deleted from the consume folder**.

## Email intake
```
alias ──Proton filter──▶ label "📝 Paperless" (+ mark read, archive)
      ──Proton Mail Bridge on the Mac Mini (IMAP, localhost)──▶
      paperless-mail.py every 15 min ──▶ consume share /email ──▶ Paperless
```
- **Read-only:** the script opens the label with IMAP `EXAMINE`; it never
  moves, flags or marks mail. Filed messages are remembered by Message-ID in
  `~/Library/Application Support/paperless-mail/filed.json` on the Mini.
- **Naming, in order:** attachment already named in the convention → kept;
  else the **subject** in the convention (`Fwd:`/`Re:` ignored) → used;
  else the attachment's own name (Paperless reads date and content itself).
- **Files:** PDFs, plus JPEG/PNG/TIFF over 50 KB (keeps out logos).
- **Tag:** everything lands in `email/` → tag **email**.
- Details: [[Mac Mini]] (script, schedule, password file, logs).

Typical use: forward a bill to the alias with the subject
`20261001 - DEWA - Electricity bill September`.

## Filename convention
```
YYYYMMDD - Correspondent - Property Subject.ext
20260101 - Allianz - Insurance certificate.pdf
20260430 - Citibank - Monthly statement.pdf
20260721 - HM Land Registry - Broad Oaks Title register.pdf
```
- **Correspondent = who produced it**, not who it is about
- **Property first in the subject** where it applies (Broad Oaks / JQ Rise),
  since the `property` tag does not distinguish the two
- Date is the **document's own date**, not the day it was filed

## What the post-consume script does
`/volume1/docker/paperless/scripts/post-consume.py` runs after every ingest and
parses the original filename to set:
- **title** — correspondent and date stripped out
- **created date** — from the filename, overriding any date guessed from content
- **correspondent** — created automatically if it does not already exist
- **ATTENTION tag** (`#FF6D00`) — applied when a correspondent had to be created,
  so new or mistyped senders surface for review

Non-matching filenames are left untouched; the script can never fail a
consumption. Test it without Paperless:
```
docker exec paperless python3 /usr/src/paperless/scripts/post-consume.py \
  --test "20260101 - Allianz - Insurance certificate.pdf"
```
Editing the script needs no rebuild; only Compose changes do.

## Taxonomy
**Tags:** banking, car, tax, travel, identity, purchase, home, property, job,
health, tenancy, **email** (provenance, set automatically), plus **ATTENTION**
as a review queue. Default to **one subject tag per document**.

**Document types:** Certificate, Contract, Invoice, Notice, Receipt, Statement,
Warranty.

**Colours** are chosen for deuteranopia: blue/yellow axis and lightness, never
red-versus-green.

## Consume share access
| Account | Rights | Used by |
|---|---|---|
| `dockerlimited` / `dockergroup` | read/write (owner) | the Paperless container |
| `mini` | read/write | Mac Mini email intake |
| Administrators | full | management |
| `everyone` | read | [ ] **remove** — nobody else should read documents in transit |

The folder uses DSM ACLs (`synoacltool -get` shows entries, not "Linux mode").

## Backups
- `/volume1/docker/paperless` rides in Hyper Backup → USB SSD, and in the weekly
  encrypted archive off-site — see [[Off-site backup (Proton Drive)]]
- **Monthly DSM task** runs `docker exec paperless document_exporter ../export`,
  writing originals plus a manifest that is restorable **without Paperless**.
  The export folder is also mirrored off-site daily. That is the copy that
  matters in ten years.

## Gotchas
- **`PAPERLESS_FILENAME_PARSE_TRANSFORMS` does not exist in v3.x.** It is read
  from the environment and silently ignored. The post-consume script replaces it
- **Multi-line YAML mangles JSON env vars.** A folded `>-` block delivered a bare
  `[` to the container. Use single-line, single-quoted values
- **The classifier overrides folder tags.** Fix: **explicit matching rules** on
  correspondents, types and tags rather than leaving everything on Automatic
- **Container Manager does not create host folders** for bind mounts; `mkdir`
  and `chown` first. Postgres needs `999:999` and mode `700` on its data dir
- **DSM ACLs override `chmod`.** A consume folder showing `d---------` is fixed
  in Control Panel → Shared Folder → Permissions, not from the shell
- **Synology indexing** creates `@eaDir` folders inside the consume share.
  Harmless, but exclude the share from Indexing Service
- **OCR is slow on the DS224+** (30–60 s per page). Import in batches of ~20.
  Move Paperless to the Mac Mini if this becomes annoying
- **Duplicates are detected by checksum**, so re-dropping an identical file is
  harmless; near-identical copies are not caught
- **Never write temp files named `._*` to an SMB share from a Mac** — that prefix
  is AppleDouble metadata and gets refused. The email script uses `*.part`
  (Paperless ignores unsupported types) and renames when complete
- **A Mac uses one SMB login per server.** Mounting a second share as a different
  user silently reuses the first login. One account per machine (`mini`)

## To do
- [ ] Remove `everyone: read` from `paperless-consume`
- [ ] Re-map `P:` with a standard (non-admin) DSM user rather than `stephen-nas`
- [ ] Delete the retired `paperless-drop` user once the email intake has run cleanly
- [ ] Clean up the `Test` correspondent and test document from 2 Oct

## Related
[[Home Infrastructure]] · [[Backups]] · [[Reverse proxy & certificates]] ·
[[Tailscale]] · [[Mac Mini]]

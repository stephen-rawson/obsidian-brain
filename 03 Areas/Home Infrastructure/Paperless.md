---
type: note
updated: 2026-09-28
---

# Paperless

Document archive for the household. OCRs everything on ingest, so scanned
paper becomes full-text searchable. **The point is retrieval, not storage** —
if you would never search for it, it does not belong here.

- **Address:** `https://paperless.home.stephenrawson.uk` via [[Reverse proxy & certificates]]
- **Login and API token** → Bitwarden ("Paperless", "Paperless API token")
- **Containers:** `paperless`, `paperless-db` (Postgres), `paperless-redis`,
  in the `tools` project on the NAS. Config under `/volume1/docker/paperless`
- **Version:** v3.2.1 (matters — see Gotchas)

## What goes in, and what does not
**In:** tenancy and property, banking, insurance, vehicle, identity, medical,
purchases and warranties, utilities, travel.

**Out, deliberately:**
- Anything belonging to [[Oliver Wyman]] or a client — corporate kit only
- IRS / US tax correspondence
- Credentials, recovery keys, key material — those live in Bitwarden
- Receipts too trivial to ever search for

## Getting documents in
| Route | How |
|---|---|
| **Laptop** | `P:` mapped to `\\192.168.1.42\paperless-consume`; drop files in |
| **Subfolder = tag** | `P:\banking\`, `P:\property\`, `P:\health\` etc. `PAPERLESS_CONSUMER_RECURSIVE` and `..._SUBDIRS_AS_TAGS` are on |
| **Phone** | Swift Paperless (iOS, third party — there is no official app), pointed at the same URL. Works over [[Tailscale]] |
| **Email** | Not set up. Proton has no plain IMAP (needs Bridge on an always-on desktop); revisit if the Mac Mini becomes a server |

Files are consumed within seconds and **deleted from the consume folder**.

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
health, tenancy, plus **ATTENTION** as a review queue.
Default to **one tag per document** — a document tagged three ways filters into
everything, which is the same as filtering into nothing.

**Document types:** Certificate, Contract, Invoice, Notice, Receipt, Statement,
Warranty.

**Colours** are chosen for deuteranopia: blue/yellow axis and lightness, never
red-versus-green.

## Backups
- `/volume1/docker/paperless` rides in Hyper Backup → USB SSD, and in the weekly
  encrypted archive to Proton Drive — see [[Off-site backup (Proton Drive)]]
- **Monthly DSM task** runs `docker exec paperless document_exporter ../export`,
  writing originals plus a manifest that is restorable **without Paperless**.
  That is the copy that matters in ten years.

## Gotchas
- **`PAPERLESS_FILENAME_PARSE_TRANSFORMS` does not exist in v3.x.** It is read
  from the environment and silently ignored. The post-consume script replaces it
- **Multi-line YAML mangles JSON env vars.** A folded `>-` block delivered a bare
  `[` to the container. Use single-line, single-quoted values
- **The classifier overrides folder tags.** After 20 Citibank statements it
  assigned Citibank + Statement + banking to an Allianz insurance document. Fix:
  set **explicit matching rules** on correspondents, types and tags rather than
  leaving everything on Automatic
- **Container Manager does not create host folders** for bind mounts; `mkdir`
  and `chown` first. Postgres needs `999:999` and mode `700` on its data dir
- **DSM ACLs override `chmod`.** A consume folder showing `d---------` is fixed
  in Control Panel → Shared Folder → Permissions, not from the shell
- **Synology indexing** creates `@eaDir` folders inside the consume share.
  Harmless, but exclude the share from Indexing Service to stop it
- **OCR is slow on the DS224+** (30–60 s per page). Import in batches of ~20.
  Move Paperless to the Mac Mini if this becomes annoying
- **Duplicates are detected by checksum**, so re-dropping an identical file is
  harmless; near-identical copies are not caught

## Related
[[Home Infrastructure]] · [[Backups]] · [[Reverse proxy & certificates]] · [[Tailscale]]

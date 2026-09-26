---
type: note
updated: 2026-09-26
---

# Obsidian & LiveSync

The notes layer. Vault **`brain`**, local on the laptop at
`C:\Users\steph\Obsidian\brain`, replicated to CouchDB on the NAS and backed up
to a private GitHub repo. Two independent copies of every note.

## Sync: Self-hosted LiveSync → CouchDB
- **Plugin:** Self-hosted LiveSync (community), on laptop and iPhone.
- **Server:** `couchdb` container on the NAS, `synobridge` 172.20.1.41,
  data in `/volume1/docker/couchdb` (inside Hyper Backup).
- **Reached at:** `https://couchdb.home.stephenrawson.uk` via [[Reverse proxy & certificates]];
  Fauxton admin UI at `/_utils`. Works on 5G through [[Tailscale]].
- **Database:** `obsidian` · **CouchDB user:** `obsidian` → Bitwarden
- **End-to-end encryption:** on, plus property encryption. Passphrase → Bitwarden
  ("Obsidian LiveSync passphrase"). The NAS holds ciphertext only.
- **Setup URI** (encodes server + encryption settings, for adding a device) →
  Bitwarden. Re-copy it after any encryption or connection change.

## Backup: Git
- Private GitHub repo `obsidian-brain`, HTTPS remote, Credential Manager auth.
- Obsidian **Git** plugin (listed as "Git" in the browser, not "Obsidian Git"):
  auto-commit every 30 min, auto-push, pull on startup.
- `.gitignore` excludes `.obsidian/workspace*.json`, `.trash/`, and
  **`.obsidian/plugins/obsidian-livesync/data.json`** (holds CouchDB credentials).
- GitHub **push protection** rejects commits containing secrets. It has fired once
  already — notes carry pointers to Bitwarden, never values.

Notes don't nest; **folders store, links connect**. A hub note (e.g.
[[Home Infrastructure]]) links its children, and backlinks do the reverse.

## Setting up a new device
1. Install Obsidian, create an **empty** vault named `brain`.
2. Community plugins → Self-hosted LiveSync → install, enable.
3. **Open setup URI** (from Bitwarden) → URI passphrase → E2E passphrase.
4. Choose **Fetch from remote**. Never "overwrite remote" from an empty vault.
5. **Turn LiveSync on** in Sync Settings — see the gotcha below.

## Gotchas (learned the hard way)
- **LiveSync must be explicitly enabled on each device.** Setting Sync Mode to
  "LiveSync" in the dropdown is not enough; the toggle/preset has to be applied,
  or changes are captured locally and never replicate. This was the cause of a
  long "nothing syncs" session.
- **"Configure" vs "Configure And Change Remote":** the first sets local intent,
  the second rebuilds the remote to match. Encryption changes need the second.
- **Compatibility review gate:** after a plugin data-version change, replication
  is paused until acknowledged **on each device**. It fails silently otherwise.
- **`(plain)` in the log** refers to the local database, not the wire. Check
  `E2EEAlgorithm is v2` in the rule list to confirm encryption is active.
- **Duplicate saved connections** appear if the wizard is run twice; only the one
  marked **(Active)** is used. Delete the stale one.
- **CouchDB needs single-node setup once**, or it loops on
  "the _users database does not exist": POST to `/_cluster_setup` with
  `enable_single_node`, or use Fauxton's setup wizard.
- **CORS settings** in CouchDB are required for Obsidian (desktop and mobile);
  the plugin's config check fixes them.

## Related
[[Home Infrastructure]] · [[Reverse proxy & certificates]] · [[Backups]] · [[Tailscale]]

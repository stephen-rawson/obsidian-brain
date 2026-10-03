---
type: note
updated: 2026-10-03
aliases:
  - claude access
  - filesystem mcp
---

# Claude Filesystem access

Claude edits this vault directly through the Filesystem MCP server in Claude
Desktop on the Lenovo.

| | |
|---|---|
| Connector | Claude Desktop – Filesystem MCP (Lenovo) |
| Scope | `C:\Users\steph\Obsidian\brain` only (whole vault incl. `.obsidian`, `.git`). Proton Drive folder no longer exposed |
| Permissions | List, read, write, edit, move; **no delete** |
| Approvals | Each call prompts in Claude Desktop; unanswered prompts time out after 4 min |
| Verified | 2026-10-03 (write test in `00 Inbox`, manually deleted; first live edits same day) |

## Gotchas
- **Writes propagate instantly** via LiveSync → CouchDB → all devices, then to
  GitHub on the next backup. A bad write is a multi-device event.
- **`write_file` overwrites silently.** Git history is the only rollback;
  LiveSync faithfully syncs mistakes.
- **Scope includes plugin config** (`.obsidian`): Claude could alter LiveSync
  settings.
- **Write tools only appear in a new chat** after the permission is toggled.
- **A tool call that "hangs" is usually an unanswered approval prompt** in
  Claude Desktop, not lost access.

## To do
- [ ] Confirm Git backup commit frequency is short enough to act as rollback
- [ ] Decide whether to narrow scope to content folders only (excludes `.obsidian`, `.git`)

## Related
[[Obsidian & LiveSync]] · [[Home Infrastructure]] · [[Runbook]]

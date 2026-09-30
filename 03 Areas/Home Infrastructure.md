---
type: area
updated: 2026-09-25
aliases:
  - infra
---
# Home Infrastructure

Hub note for the flat's network, servers and self-hosted services.
**Secrets live in Bitwarden, never here.**

- One SSID, one LAN; everything behind the Synology router, not the du box.
- Anything reachable remotely goes via Tailscale. **No port forwards, ever.**
- Names, not IPs: `*.home.stephenrawson.uk` via NPM.
- Config in bind mounts under `/volume1/docker/<app>`, backed up by Hyper Backup.
- Credentials in Bitwarden; recovery keys also in Proton Drive.
## Links
```dataviewjs
const me = dv.current().file;
const folder = me.folder + "/" + me.name;
const items = dv.pages(`"${folder}"`)
    .where(p => p.file.path !== me.path)
    .sort(p => p.file.name);

if (items.length) {
    dv.list(items.map(p => p.file.link));
} else {
    dv.paragraph("*No notes in this area yet.*");
}
```
## Linked Tasks
```dataviewjs
const me = dv.current().file.name;
const tasks = dv.pages()
    .file.tasks
    .where(t => !t.completed && t.text.includes(`[[${me}]]`));

if (tasks.length) {
    dv.taskList(tasks, false);
} else {
    dv.paragraph("*No open tasks.*");
}
```
## Recent mentions
```dataviewjs
const me = dv.current().file.link;
const hits = dv.pages('"01 Daily"')
    .where(p => p.file.outlinks.some(l => l.path === me.path))
    .sort(p => p.file.name, 'desc')
    .limit(10);

if (hits.length) {
    dv.list(hits.map(p => `${p.file.link} — ${p.file.day ? p.file.day.toFormat("ccc d LLL") : ""}`));
} else {
    dv.paragraph("*No daily notes reference this yet.*");
}
```
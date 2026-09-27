---
type: area
---
## Overview
OCRs every document you feed it, tags and files it, and gives you full-text search over everything. Your tenancy contract, Ejari, the IRS notices, warranty receipts for the NAS drive, scanned notebook inserts: one search box.

```
  paperless-redis:
    image: redis:7
    container_name: paperless-redis
    networks: [synobridge]
    volumes:
      - /volume1/docker/paperless/redis:/data
    restart: unless-stopped

  paperless-db:
    image: postgres:16
    container_name: paperless-db
    networks: [synobridge]
    environment:
      POSTGRES_DB: paperless
      POSTGRES_USER: paperless
      POSTGRES_PASSWORD: Massive-Unvarying7-Mongrel
    volumes:
      - /volume1/docker/paperless/db:/var/lib/postgresql/data
    restart: unless-stopped

  paperless:
    image: ghcr.io/paperless-ngx/paperless-ngx:latest
    container_name: paperless
    depends_on: [paperless-db, paperless-redis]
    networks:
      synobridge:
        ipv4_address: 172.20.1.42
    environment:
      PAPERLESS_REDIS: redis://paperless-redis:6379
      PAPERLESS_DBHOST: paperless-db
      PAPERLESS_DBUSER: paperless
      PAPERLESS_DBPASS: Massive-Unvarying7-Mongrel
      PAPERLESS_SECRET_KEY: 4S!WN!A4Zu^%6Ikr
      PAPERLESS_TIME_ZONE: Asia/Dubai
      PAPERLESS_OCR_LANGUAGE: eng
      PAPERLESS_URL: https://paperless.home.stephenrawson.uk
      PAPERLESS_FILENAME_FORMAT: "{correspondent}/{document_type}/{created_year}-{created_month}-{created_day} {title}"
      USERMAP_UID: "1027"
      USERMAP_GID: "65536"
    volumes:
      - /volume1/docker/paperless/data:/usr/src/paperless/data
      - /volume1/docker/paperless/media:/usr/src/paperless/media
      - /volume1/docker/paperless/export:/usr/src/paperless/export
      - /volume1/paperless-consume:/usr/src/paperless/consume
    logging:
      driver: json-file
      options: { max-size: "10m", max-file: "3" }
    restart: unless-stopped
```

## Open Tasks
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
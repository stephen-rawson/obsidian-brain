---
type: note
updated: 2026-09-25
---

# Network & IPs

## Topology
```
du ONT (192.168.70.1)  ← fibre, ISP-owned, Wi-Fi disabled
   └─ Synology RT6600ax  WAN 192.168.70.2 (double NAT)
        LAN 192.168.1.0/24 · SSID "Rayphen"
          ├─ NAS, Pi, HA, Hue, laptops, phones
          └─ (todo) UniFi switch → in-wall drops
```
Wall drops (TV x2, D03/T03, bedroom x2) terminate on the patch panel in the du
enclosure and are **currently fed by the du box**, i.e. on 192.168.70.x, not the
LAN. Fixing this is part of the cabinet rewire.

## Addressing plan (192.168.1.0/24)
| Range | Use | Assigned by |
|---|---|---|
| .1 | Router | fixed |
| .2–.19 | reserved for core infra | DHCP reservation |
| .20–.199 | clients (phones, laptops, TV, IoT) | DHCP pool |
| .200–.254 | self-assigned / macvlan | static |

## Fixed addresses
| Device | IP | MAC | Method |
|---|---|---|---|
| Router | 192.168.1.1 | — | fixed |
| NAS | 192.168.1.42 | 90:09:d0:54:2c:55 | reservation |
| Pi-hole | 192.168.1.31 | d8:3a:dd:66:69:a5 | reservation |
| Home Assistant | 192.168.1.119 | 20:f8:3b:02:90:e0 | reservation |
| NPM (macvlan) | 192.168.1.250 | — | static, outside pool |
| du ONT | 192.168.70.1 | — | ISP |

## Docker network `synobridge` (172.20.0.0/16)
Containers with static IPs at 172.20.1.x: gluetun .10, flaresolverr .20,
sonarr .21, radarr .22, jellyfin .23, NPM .40, couchdb .41 (planned).
Docker's allocator starts at 172.20.0.2, so 172.20.1.x never collides.

## Wi-Fi
- SSID **Rayphen**, WPA2/WPA3 mixed, Smart Connect on. Guest net on 192.168.2.0/24.
- 5 GHz on **channel 136 (DFS)** — quiet, confirmed clean.
- Measured Sept 2026: desk −45, kitchen −50, guest bath −55, bedroom −57,
  main bathroom −82 (only weak spot; not worth an AP).
- Bedroom wall drop stays available for a wired AP if that changes.

## Gotchas
- **macvlan**: the NAS cannot reach 192.168.1.250 (its own macvlan child). Anything
  on the NAS that must reach NPM uses the bridge gateway 172.20.0.1 instead.
- Never advertise the LAN subnet from the NAS in Tailscale — see [[Tailscale]].

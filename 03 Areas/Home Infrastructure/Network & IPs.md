---
type: note
updated: 2026-09-30
aliases:
  - IP plan
  - addressing
  - DHCP reservations
  - LAN
---
# Network & IPs

## Topology
```
du ONT (192.168.70.1)            fibre, ISP-owned, Wi-Fi disabled
   └─ Synology RT6600ax          WAN 192.168.70.2 (double NAT)
        LAN 192.168.1.0/24 · SSID "Rayphen"
          └─ UniFi USW-Pro-24-PoE (port 1 uplink)
               ├─ NAS, Pi, Home Assistant, Mac Mini, Hue Bridge
               └─ (todo) patch panel → in-wall drops
```
Everything above sits on the **APC Smart-UPS** via the PDU — see [[UPS & power]].

In-wall drops (TV x2, D03/T03, bedroom x2) terminate on the patch panel in the
du enclosure and are **still fed by the du box**, i.e. on 192.168.70.x, not the
LAN. Moving them onto the switch is part of the cabinet rewire.

## Addressing plan (192.168.1.0/24)
| Range     | Use                                   | Assigned by          |
| --------- | ------------------------------------- | -------------------- |
| .1        | Router                                | fixed                |
| .2–.19    | Core infrastructure                   | DHCP reservation     |
| .20–.199  | Clients + grandfathered infra (below) | DHCP pool            |
| .200-219  | Semi-fixed infrastructure (e.g., TV)  | DHCP reservation     |
| .220–.254 | Self-assigned / macvlan               | static, outside pool |

**Lesson:** a reservation inside the pool only works if nothing already holds
that address. The Mac Mini was first reserved at .43, which another device had
leased, so it silently stayed dynamic. New infrastructure goes in **.2–.19**.

## Fixed addresses
| Name               | IP           | MAC               | Method      | Notes                                      |
| ------------------ | ------------ | ----------------- | ----------- | ------------------------------------------ |
| router             | .1           | 90:09:d0:40:2a:46 | fixed       | Synology RT6600ax                          |
| switch             | .10          | 70:a7:41:f6:ed:d3 | reservation | UniFi USW-Pro-24-PoE                       |
| zigbee             | .11          | 68:25:dd:2e:33:2b | reservation | SLZB-MR1                                   |
| mini               | .15          | 14:98:77:83:ae:5f | reservation | Mac Mini M1 8 GB; UniFi controller, Ollama |
| pihole             | .31          | d8:3a:dd:66:69:a5 | reservation | *grandfathered in pool*                    |
| nas                | .42          | 90:09:d0:54:2c:55 | reservation | *grandfathered in pool*; eth0 / LAN 1      |
| hue                | .66          | ec:b5:fa:94:3b:36 | reservation | *grandfathered in pool*; 10/100 port only  |
| homeassistant      | .119         | 20:f8:3b:02:90:e0 | reservation | *grandfathered in pool*                    |
| airgradient-living | .141         | d8:3b:da:1d:59:28 | reservation | *grandfathered in pool*                    |
| airgradient-master | .150         | d8:3b:da:1f:7a:34 | reservation | *grandfathered in pool*                    |
| tv                 | .201         | 00:c3:f4:a9:bf:89 | reservation |                                            |
| appletv            | .202         | c4:f7:c1:35:05:04 | reservation |                                            |
| playstation        | .203         | 5c:84:3c:a6:a6:b2 | reservation |                                            |
| npm                | .250         | 02:42:c0:a8:01:fa | static      | NPM macvlan on the NAS                     |
| du ONT             | 192.168.70.1 | —                 | ISP         |                                            |

**Grandfathered:** Pi-hole, NAS, HA, Hue and the AirGradients predate the plan
and are referenced in many configs (router DNS, Tailscale nameserver, HA
integrations, gluetun firewall, NPM upstreams). They stay where they are.

## Switch port map (UniFi)
| Port | Device         | Speed       | PoE |
| ---- | -------------- | ----------- | --- |
| 1    | Router uplink  | GbE         | off |
| 2    | NAS            | GbE         | off |
| 3    | Pi-hole        | GbE         | off |
| 4    | Home Assistant | GbE         | off |
| 5    | Mac Mini       | GbE         | off |
| 6    | Hue Bridge     | FE (normal) | off |
| 7    | TV (D02)       | GbE         | off |
| 8    | Zigbee         | FE          | on  |
| 9–24 | spare          | —           | off |

## Name resolution
- **Pi-hole local DNS:** `<name>.lan` for every row above, plus `unifi → .15`
  (the switch looks for a host literally called `unifi` to find its controller).
  Pi-hole auto-creates reverse records, so its query log shows names, not IPs.
- **Wildcard:** `*.home.stephenrawson.uk → .250` (NPM) — see [[Reverse proxy & certificates]].
- **Pi scans:** `lanscan` alias on the Pi runs arp-scan with
  `/etc/arp-scan/mac-vendor.txt`, which labels known MACs by name.

## Docker network `synobridge` (172.20.0.0/16)
Static container IPs at 172.20.1.x: gluetun .10, flaresolverr .20, sonarr .21,
radarr .22, jellyfin .23, recyclarr (dynamic), NPM .40, couchdb .41,
paperless .42. Docker's allocator starts at 172.20.0.2, so 172.20.1.x never
collides. Bridge gateway 172.20.0.1 = the NAS itself.

## Wi-Fi
- SSID **Rayphen**, WPA2/WPA3, Smart Connect on. Guest net 192.168.2.0/24.
- 5 GHz on **channel 136 (DFS)** — quiet. A freshly wiped Mac can take a minute
  to see DFS networks.
- Survey (Sept 2026): desk −45, kitchen −50, guest bath −55, bedroom −57,
  main bathroom −82 (only weak spot; not worth an AP).

## Open items
- [ ] Identify unknown clients at **.37** (bc:03:58:c2:8c:33) and **.43**
      (bc:6e:e2:ef:e0:ac) — router Device List or Pi-hole → Network
- [ ] Check the laptop isn't routing LAN traffic via Tailscale at home:
      `tracert -d -h 3 192.168.1.42` — a 100.x first hop means it is
- [ ] Rewire patch panel from du box to switch
- [ ] Known-MAC allowlist alert (arp-scan → HA) for new devices

## Gotchas
- **macvlan:** the NAS cannot reach .250 (its own macvlan child). NAS-side
  services reaching NPM use 172.20.0.1; NPM reaching DSM uses 172.20.0.1:5001.
- **Never advertise the LAN route from the NAS** in Tailscale — same reason.
  The Pi is the subnet router. See [[Tailscale]].
- **DHCP client list is incomplete after a router reboot** — it only shows
  leases issued since. Devices keep old addresses until renewal (up to 24 h).
- **Old UniFi firmware** (5.76, 2021) only offered `ssh-rsa`, `ssh-dss` and
  `hmac-sha1`; updated to 7.5.15, so modern SSH works now.

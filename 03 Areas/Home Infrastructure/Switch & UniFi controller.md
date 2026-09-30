---
type: note
updated: 2026-09-30
aliases: [switch, UniFi, USW-Pro-24-PoE, UniFi controller, unifi]
---

# Switch & UniFi controller

Core LAN switch. Everything wired in the flat goes through it; it hangs off the
router's LAN port and sits on the UPS.

## Switch
| | |
|---|---|
| Model | UniFi **USW-Pro-24-PoE** (US variant) |
| Address | `192.168.1.10` · `switch.lan` · reserved by MAC |
| MAC | `70:a7:41:f6:ed:d3` |
| Firmware | **7.5.15** (updated from 5.76.5, Oct 2021 build) |
| Power | 100–240 V, C14 inlet, earthed IEC lead to the UPS/PDU |
| Device SSH | credentials set by the controller → Bitwarden ("UniFi device SSH") |

### Port map
| Port | Device | Speed | PoE |
|---|---|---|---|
| 1 | Router uplink | GbE | off |
| 2 | NAS | GbE | off |
| 3 | Pi-hole | GbE | off |
| 4 | Home Assistant | GbE | off |
| 5 | Mac Mini | GbE | off |
| 6 | Hue Bridge | FE | off |
| 7–24 | spare | — | off |

- **PoE is off everywhere.** Nothing in the flat is PoE-powered. Enable per port
  only when something needs it (e.g. a camera). PoE only energises after
  detecting a compatible device, so leaving it on is not dangerous, just untidy
  and marginally warmer.
- **FE on the Hue Bridge is normal** — the bridge has a 10/100 port.
- Future: patch-panel drops (TV D02, Apple TV T02, D03/T03, bedroom ×2) move onto
  ports 7+ during the cabinet rewire. Name each port as it is patched.

## Controller (UniFi Network)
| | |
|---|---|
| Host | Mac Mini, Docker container `unifi` |
| URL | `https://192.168.1.15:8443` |
| Account | local admin → Bitwarden ("UniFi controller") |
| Inform host | overridden to `unifi` (Pi-hole: `unifi → 192.168.1.15`) |
| Remote access | off — use [[Tailscale]] |

- **Third-party gateway setup:** the Synology router is the gateway. In the
  controller, the Default network is `192.168.1.0/24`, gateway `.1`, **DHCP
  server off**. It must never hand out addresses alongside the router.
- **The switch keeps switching if the controller is down.** It only loses
  manageability. A dead Mini is not an outage.

## Backups
- Controller autobackup: weekly, keep 8.
- Location on the Mini: `~/unifi/data/backup/autobackup`
- [ ] Copy autobackups to the NAS on a schedule (the Mini is not in Hyper Backup)
- A restore needs a controller of the **same or newer version**.

## Monitoring
- [ ] HA: Ping sensor for `.10` and a line in `sensor.systems_issues`
- [ ] HA: UniFi Network integration, using a **dedicated** local admin for HA
- Temperature and fan level visible in the controller's device panel

## Fan
- The switch's touchscreen exposes a fan-speed control; the controller shows
  fan level and temperature.
- Levers for noise: current firmware, PoE off, side clearance for venting, and
  keeping the UPS from sitting against it. Judge noise after 20–30 minutes of
  uptime, not at boot.

## How it was set up (for next time)
1. Unadopted UniFi devices take DHCP, then look for a controller at
   `http://unifi:8080/inform`. Status via SSH `info`.
2. Factory SSH credentials `ubnt` / `ubnt`. Old firmware needed:
   ```
   ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa \
       -o MACs=+hmac-sha1 -o KexAlgorithms=+diffie-hellman-group14-sha1 ubnt@<ip>
   ```
   Not needed on 7.x firmware.
3. Point it at the controller: Pi-hole `unifi` record, or `set-inform
   http://192.168.1.15:8080/inform` over SSH.
4. Adopt in the controller → update firmware → set device SSH credentials →
   override inform host → name ports → PoE off.
5. **Factory reset:** hold the front reset pin ~10 s until LEDs flash. Needed
   only if the defaults are rejected (previously adopted elsewhere).

## Gotchas
- **Adopting while the controller host is on a temporary IP** bakes that IP into
  the switch's inform URL. Always override the inform host to a name.
- A **travel adapter** on the switch's original US cord broke the earth path and
  caused a tingle on the metal case. Replaced with an earthed C13 lead. No travel
  adapters anywhere in the cabinet.

## Related
[[Network & IPs]] · [[Home Infrastructure]] · [[UPS & power]] · [[Tailscale]]

---
type: note
updated: 2026-09-30
aliases: [UPS, APC, power, NUT, PDU, power cut]
---

# UPS & power

**Why this exists:** in Aug 2026 a faulty shared power strip cut the NAS twice
without a clean shutdown. Drive 1 dropped out of the RAID and the array ran
degraded for ~5 weeks unnoticed. The UPS removes the cause; the monitoring
below removes the "unnoticed".

## Hardware
| | |
|---|---|
| UPS | **APC Smart-UPS SMC1500IC** — 1500 VA / 900 W, pure sine wave, USB |
| Outlets | IEC **C13** (needs C14 plugs or a C14-input PDU) |
| PDU | DKURVE 6-way UK, **C14 input**, 10 A, 3 m lead, on a battery-backed outlet |
| Installed | Sept 2026 |
| Battery | replace every ~3–5 years → reminder **2029** |

## What is on battery
| Device | Connection | Notes |
|---|---|---|
| Router | PDU | |
| Switch | UPS C13 direct | earthed C13–C14 lead |
| NAS | PDU | USB data cable to the UPS |
| Pi-hole | PDU | |
| Home Assistant | PDU | |
| Mac Mini | PDU | |
| du ONT | — | [ ] confirm it is on battery; without it the LAN stays up with no internet |

- **Load ~15%** expected (≈135 W of 900 W). Record the real figure from the UPS
  display: ______ %, estimated runtime ______ min.
- **Never on the UPS:** laser printers, heaters, anything with a motor (inrush).
- **No travel adapters or daisy-chained strips** in the power chain. One broke
  the switch's earth path and put a tingle on its metal case.

## Monitoring (NUT)
The NAS owns the USB link and runs a NUT server; everything else listens.

**DSM → Control Panel → Hardware & Power → UPS**
- UPS support on, type **USB UPS**
- Time before Safe Mode: **10 minutes** (rides out short flickers)
- Network UPS server **on**, permitted IPs: `.31` Pi, `.119` HA, `.15` Mini
- NUT identity is Synology's fixed default: UPS `ups`, user `monuser`,
  password `secret`. Not changeable in DSM; the permitted-IP list is the control.

**Pi** — `nut-client` in `netclient` mode
```
/etc/nut/upsmon.conf:
MONITOR ups@192.168.1.42 1 monuser secret secondary
```
Shuts down cleanly when the NAS broadcasts forced shutdown. Check:
`upsc ups@192.168.1.42 ups.status` → `OL` online, `OB` on battery, `LB` low.

**Home Assistant** — NUT integration, host `192.168.1.42:3493`.
Sensors: status, battery charge, runtime, load, input voltage.
- **On battery** (30 s) → critical push + torrents to slow mode
- **Mains restored** (1 min) → push
- **Battery < 30% while on battery** → push, then `hassio.host_shutdown`
- `sensor.systems_issues` includes "UPS on battery"

**Mac Mini** — not a NUT client yet; it holds nothing that corrupts on an abrupt
stop. [ ] Optional: `brew install nut`, same config as the Pi.

## What happens in a power cut
1. UPS takes the load instantly; HA pushes "Power cut".
2. Mains back within 10 min → nothing else happens; HA pushes "restored".
3. Still out at 10 min → NAS enters Safe Mode and signals clients → Pi shuts down.
4. Battery < 30% → HA shuts its own host down.
5. Router, switch, ONT and Mini run until the battery is exhausted.
6. Mains returns → NAS, Mini and HA power on automatically (auto-restart after
   power failure is enabled on each); the Pi boots whenever it gets power.
7. After recovery: restart `qbittorrent` and `prowlarr` (gluetun coupling), or
   run `rebuild.sh`.

## Manual shutdown and power-up (for maintenance)
**Down, in dependency order:** Docker stack → Home Assistant (Settings → System
→ Shut down) → Pi (`sudo shutdown -h now`) → Mac Mini → NAS (DSM → Shut down)
→ router → ONT.

**Up, reverse order, pausing between each:** ONT (wait for PON solid) → router
(2 min) → switch → Pi (check DNS) → NAS (3–5 min) → Home Assistant → Mini.

## Tests
- [x] Battery connected, UPS charged
- [ ] UPS self-test (front panel)
- [ ] **Pull-plug test:** unplug the UPS from the wall for 2 min; expect the HA
      push, `OB` status, nothing drops; reconnect, expect "restored"
- [ ] **Full shutdown test** (quiet evening): leave it unplugged past 10 min;
      confirm Pi and NAS shut down in order and come back cleanly
- [ ] Socket tester on the wall socket the UPS uses

## Gotchas
- **NAS shutdown can stall** at the final stage: DSM unreachable, power LED lit.
  Wait until the drive LEDs are solid and idle before forcing it off; then
  expect an improper-shutdown notice and run a data scrub.
- **Tingle on a metal case = broken earth**, not static. Unplug, isolate device
  by device, check for adapters and two-pin leads.
- The **Hue Bridge and the in-wall drops** are not on the UPS path yet; the
  drops move onto the switch during the cabinet rewire.

## Related
[[Network & IPs]] · [[NAS (Synology DS224+)]] · [[Home Assistant]] ·
[[Switch & UniFi controller]] · [[Runbook]]

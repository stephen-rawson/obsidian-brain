---
type: note
updated: 2026-10-02
aliases: [Zigbee2MQTT, Z2M, SMLIGHT, coordinator, Aqara buttons, buttons, MQTT]
---

# Zigbee

Our own Zigbee network for buttons and switches, separate from the Hue
Bridge's (Hue bulbs stay on Hue). Home Assistant joins the two.

## The chain
```
Aqara buttons ──Zigbee ch 25──▶ SMLIGHT coordinator (192.168.1.11, PoE)
                                   │ TCP 6638
                                   ▼
                       Zigbee2MQTT add-on (HA)
                                   │ MQTT
                                   ▼
                       Mosquitto add-on ──▶ HA MQTT integration ──▶ automations
```

## Coordinator
|                |                                                                                |
| -------------- | ------------------------------------------------------------------------------ |
| Device         | SMLIGHT SLZB-06 family (Ethernet + PoE)                                        |
| Address        | `192.168.1.11` · `zigbee.lan` · reserved by MAC                                |
| Web UI         | `https://zigbee.home.stephenrawson.uk` (NPM, lan-only)                         |
| Login          | web authentication on → Bitwarden ("SMLIGHT coordinator")                      |
| Radio chip     | EFR32 → `ember`                                                                |
| Radio firmware | **Coordinator** role                                                           |
| Zigbee socket  | TCP **6638** — direct only, **never proxied**                                  |
| Power          | PoE from the UniFi switch, port 8 (PoE on for this port only)                  |
| Placement      | central, antenna vertical, out of the closed cabinet, away from router and NAS |

## Channel plan
| System              | Channel                             |
| ------------------- | ----------------------------------- |
| Zigbee2MQTT         | **25**                              |
| Hue Bridge (Zigbee) | 11                                  |
| Wi-Fi 2.4 GHz       | **6** to clear both Zigbee networks |

Changing the Zigbee channel later means re-pairing every device.

## Home Assistant side
- **Mosquitto broker** add-on: Start on boot + Watchdog.
- **MQTT integration** configured from the Mosquitto discovery card. The broker
  log should show a persistent client as `homeassistant`.
- **Zigbee2MQTT** add-on (repo `github.com/zigbee2mqtt/hassio-zigbee2mqtt`):
  ```yaml
  serial:
    port: tcp://192.168.1.11:6638
    adapter: zstack            # or ember
  advanced:
    channel: 25
    last_seen: ISO_8601
  availability:
    enabled: true
  ```
  Home Assistant integration enabled in its settings.
- **ZHA: not used.** Its discovery card for the SLZB is ignored — ZHA and
  Zigbee2MQTT cannot share the radio.

## Devices
| Friendly name | Model | Room |
|---|---|---|
| `aqara_master_bedside_mini` | Aqara Wireless Mini Switch T1 | Master, bedside |
| `aqara_master_wall_mini` | Aqara Wireless Mini Switch T1 | Master, wall by door |
| `aqara_living_desk_mini` | Aqara Wireless Mini Switch T1 | Living, desk |
| `aqara_living_wall_rocker` | Aqara Wireless Remote Switch H1 (double) | Living, wall |

H1 settings in Zigbee2MQTT: **`operation_mode: event`** (or HA never sees the
presses) and **`click_mode: multi`** (or it only reports single presses).

## What the buttons do
**Rule:** living buttons only touch living lights; bedroom buttons only touch
bedroom lights, **except Sleep**, which turns everything off. Normal mode changes
the house mode and sensor displays, not lights.

| Button | Single | Double | Triple | Hold |
|---|---|---|---|---|
| Bedside + bedroom wall | Toggle bedroom | Bedroom Rest | Normal mode | **Sleep** |
| Desk | Toggle living | Living Rest | Living Relax | Normal mode |

| Rocker | Left | Right | Both |
|---|---|---|---|
| Single | Rest ↔ off | Relax ↔ off | Normal mode |
| Double | Bright ↔ off | **Hosting** (placeholder) | — |
| Hold | Living off | Living off | Living off |

"↔ off": if living lights are off → that scene; if on → off.

## How the automations are built
- **MQTT triggers** on `zigbee2mqtt/<friendly name>`, reading
  `payload_json.action` — independent of how HA names entities. A condition
  ignores messages without an action (battery/linkquality updates share the topic).
- Bedside and bedroom-wall share **one** automation (two triggers), so they
  can't drift apart.
- **No `enable_automations` gate** on buttons. The Sleep and Normal scripts are
  ungated; the gate lives on the scheduled automations instead.
- **Battery digest:** Saturday 10:05 push if any Aqara battery is below 20%,
  found by device class so names don't matter.

## Pairing a new device
1. Zigbee2MQTT → **Permit join (All)** (≈4 min window).
2. Next to the coordinator, hold the button ~5 s until the LED blinks.
3. **Tap it every second** until "Successfully interviewed" — Aqara devices sleep
   mid-interview otherwise.
4. Rename at once (Settings → Friendly name, tick *update HA entity ID*).
5. Press every gesture and note the `action` names it sends.

## Backups
The **network key** lives in `/config/zigbee2mqtt/configuration.yaml`. Lose it and
every device must be re-paired. It is inside HA backups (add-on data) → NAS →
Proton. [ ] Confirm the HA backup lists the Zigbee2MQTT add-on.
**Never uninstall the Zigbee2MQTT add-on casually.** Mosquitto and the MQTT
integration are safe to remove and re-add.

## Gotchas (from setup day)
- **Flashed router firmware by mistake.** Coordinator is the role that forms the
  network; router extends someone else's; Matter/Thread is a different protocol.
- **Online coordinator flash failed with `http_code -1`** — a reboot of the
  SMLIGHT fixed it (low memory after the previous flash).
- **Mosquitto running ≠ HA connected.** Installing the add-on doesn't set up the
  MQTT integration. Diagnose from the Mosquitto log: Zigbee2MQTT connects as
  `addons`; HA must appear as `homeassistant`. The 2-minute connect/disconnect
  from 172.30.32.2 is just the Supervisor health check.
- **Reinstalling Mosquitto** regenerates credentials; Zigbee2MQTT briefly logs
  "not authorised" and then reconnects.
- **Only `update.<name>` appeared in HA** after renaming — hence MQTT triggers
  rather than event entities.
- **Aqara and routers:** fine while everything talks straight to the coordinator.
  If a far button drops, add an Aqara or IKEA mains plug as a router — not a
  random Tuya one.

## Open items
- [ ] Record the radio chip and the switch port above
- [ ] Move Wi-Fi 2.4 GHz to channel 6
- [ ] HA **SMLIGHT** integration (monitoring only) + `systems_issues` line for the
      Zigbee2MQTT bridge going offline
- [ ] Hosting scene: Hue "Hosting" scene, Frame Art Mode test, Sonos Beam for
      music (after the trip)
- [ ] Movie mode from the Apple TV's play state

## Related
[[Home Assistant]] · [[Switch & UniFi controller]] · [[Network & IPs]] ·
[[Reverse proxy & certificates]] · [[Hue & lighting]]

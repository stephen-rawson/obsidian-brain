---
type: note
updated: 2026-10-02
aliases: [Hue, lighting, lights, scenes, home mode, modes, Play bars]
---

# Hue & lighting

All bulbs and strips are **Philips Hue** on the **Hue Bridge's own Zigbee
network**. Home Assistant drives them through the Hue integration; buttons come
in separately via [[Zigbee]].

## Hue Bridge
| | |
|---|---|
| Address | `192.168.1.66` · `hue.lan` · reserved by MAC |
| MAC | `ec:b5:fa:94:3b:36` |
| Switch | UniFi port 6, PoE off, **FE** (the bridge has a 10/100 port — normal) |
| Zigbee | channel **11** (our Zigbee2MQTT network is on 25) |
| Power | [ ] confirm whether it is on the UPS |

## Lights
| Entity | What | Room |
|---|---|---|
| `light.hue_living_main` | Living room main | Living |
| `light.hue_living_lightstrip_kitchen` | Kitchen strip | Living |
| `light.hue_living_lightstrip_tv` | TV strip | Living |
| `light.hue_living_play_left` / `_right` | Hue Play bars (shared PSU) | Living |
| `light.hue_master_main` | Bedroom main | Master |
| `light.hue_master_lightstrip` | Bed strip | Master |

**Zones** (group entities): `light.hue_living_zone`, `light.hue_master_zone`,
`light.hue_home_zone` (everything). A zone reads *on* if any light in it is on;
`light.toggle` on a zone turns everything off if anything is on.

## Scenes (defined in the Hue app)
| Zone | Scenes |
|---|---|
| Home | `hue_home_bright` · `hue_home_relax` · `hue_home_rest` |
| Living | `hue_living_bright` · `hue_living_relax` · `hue_living_rest` · [ ] `hue_living_hosting` |
| Master | `hue_master_bright` · `hue_master_relax` · `hue_master_rest` · `hue_master_chinatown` |

## Labels
- **`all_lights`** — every light that Sleep and Away should turn off. Includes
  the Play bars. **Any new light must get this label.**
- **`air_sensor_lights`** — not lights: the display/LED brightness `number`
  entities on the two AirGradient sensors, dimmed by the modes.

## Home modes (`input_select.home_mode`)
| Mode | Script | Lights | Sensor displays | Gated? |
|---|---|---|---|---|
| Normal | `set_home_mode_normal` | none | 100% | **no** |
| Wind Down | `set_home_mode_wind_down` | Living Rest + Master Chinatown, 30 min fade | 5% | yes |
| Sleep | `set_home_mode_sleep` | **all off** (10 s fade) | off | **no** |
| Away | `set_home_mode_away` | all off; torrents unthrottled | off | yes |
| Hosting | `living_hosting` (toggle) | Living Hosting scene | — | no |

"Gated" = the script only touches lights when `input_boolean.enable_automations`
is on. Sleep and Normal are ungated so buttons always work; the schedule that
calls them checks the switch itself. Away stays gated because the presence
automation calls it without a gate of its own.

**`enable_automations` means "pause the schedule"** — not "disable buttons".

## Schedule (automations)
| When | Does | Conditions |
|---|---|---|
| 07:00 | Normal mode | switch on, someone home |
| Sunset − 30 min | Home Relax, 30 min fade | switch on, someone home, mode Normal |
| 21:00 | Wind Down mode | switch on, someone home |
| 23:30 | Sleep mode | switch on, someone home |
| Everyone out 30 min | Away mode | — |
| Someone returns (mode Away) | Normal mode | — |

[ ] Add **"mode is not Hosting"** to the 21:00, 23:30 and sunset automations, or
the schedule will dim and then kill the lights on guests.

## Buttons
See [[Zigbee]]. Rule: living buttons only touch living lights; bedroom buttons only
touch bedroom lights, except Sleep.

## Dashboard
Home view → **Controls** (mode buttons, amber when active) and **Lights** (zone
tiles with brightness sliders, strip tiles, Bright / Relax / Rest / All off).
Tile colours follow the actual light colour.

## Adding a new light (checklist)
1. Hue app: add it, name it, assign to a room; check it's in the right zones.
2. **Edit every existing scene** in that room/zone and save — Hue scenes do not
   include lights added after the scene was created.
3. HA: rename the entity (`light.hue_<room>_<what>`), set the area.
4. HA: add the **`all_lights`** label.
5. Test: a scene button, then Sleep — the new light should follow both.

## Gotchas
- **Scenes don't pick up new lights** (step 2 above). The most common reason a
  light "ignores" the evening routine.
- **The Hue Bridge has its own Zigbee network.** Hue bulbs don't relay for our
  Zigbee2MQTT buttons, and vice versa.
- **Wind Down was too abrupt** at a 30-second transition; now 30 minutes.
- **Two scheduled routines can fight:** the sunset fade and Wind Down both touch
  the living room in the evening.

## Ideas / next
- [ ] **Hosting:** Hue "Hosting" scene; script toggles mode, lights, Frame Art
      Mode and (with a Sonos Beam) a playlist
- [ ] **Movie mode:** Apple TV playing after sunset → dim living; paused → Relax
- [ ] **Morning ramp:** bedroom fade-up before the alarm
- Optional: Hue Entertainment area for the Play bars (true screen sync needs a
  Hue Sync Box)

## Related
[[Zigbee]] · [[Home Assistant]] · [[Network & IPs]] · [[UPS & power]]

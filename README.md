# Aqara W100 ↔ Climate Entity Sync — Home Assistant Blueprint

A Home Assistant blueprint that provides **bidirectional synchronization** between an
Aqara W100 (used as a virtual thermostat) and any real `climate` entity (heat pump,
boiler, AC, etc.).

> **Note:** This is a trimmed-down fork. The original project also shipped a
> Zigbee2MQTT external converter (`w100.js`). That converter is **no longer needed**:
> its changes were merged upstream into `zigbee-herdsman-converters`
> ([PR #10787](https://github.com/Koenkk/zigbee-herdsman-converters/pull/10787)) and
> ship natively in **Zigbee2MQTT ≥ 2.7.0**. This repository now contains **only the
> blueprint**.

---

## What it does

Once linked, the W100 and your real thermostat stay in sync in both directions:

- **HVAC mode** — `off` / `heat` / `cool` / `auto` (with smart mapping, see below)
- **Target temperature** — kept identical on both sides
- **Fan speed** (`fan_mode`) — optional, mapped between the W100 (`auto`/`low`/`medium`/`high`)
  and your thermostat's own fan steps
- **External temperature & humidity** — optionally published to the W100 display from
  any HA sensor

Changes made on the W100 panel propagate to the real thermostat, and changes made in
Home Assistant are reflected back on the W100.

---

## Mode mapping (important)

The W100 firmware only supports **four** HVAC modes: `off`, `heat`, `cool`, `auto`.
Many real thermostats (e.g. Daikin) expose extra modes such as `dry`
(dehumidification) and `fan_only`. This blueprint handles them so they don't conflict:

| Real thermostat mode | W100 behavior |
| --- | --- |
| `off` / `heat` / `cool` | Passed through 1:1 |
| `dry` (dehumidification) | **Mapped to the W100 `auto` slot**, bidirectionally. Pressing **Auto** on the W100 puts the real thermostat into `dry`. |
| `auto` (native) | Not used by this mapping — the `auto` slot is repurposed for `dry`. |
| `fan_only` | **Ignored by the W100**: you can still enable it in Home Assistant, the W100 simply keeps its current mode (no error, no sync loop). It cannot be triggered *from* the W100. |

### Why `dry ↔ auto`?

Since the W100 has no `dry` mode and only four slots, the `auto` slot is reused to
drive dehumidification. The trade-off is that the W100 can no longer command the real
thermostat's *native* `auto` mode. If you need a different trade-off, see
[Customizing the mapping](#customizing-the-mapping).

---

## Requirements

- **Zigbee2MQTT ≥ 2.7.0** (native Aqara W100 / `Aqara TH-S04D` support — no external
  converter required).
- The W100 paired and exposed in Home Assistant with its `climate`, `number`
  (external temperature/humidity) and `switch` (`thermostat_mode`) entities.
- A real `climate` entity to sync with.

---

## Installation

1. In Home Assistant, go to **Settings → Automations & Scenes → Blueprints → Import
   Blueprint**.
2. Import [`w100-blueprint.yaml`](w100-blueprint.yaml) (upload it, or paste the raw URL
   of this file from your repo).
3. Create a new automation from the blueprint and fill in the inputs (below).
4. Save and enable.

---

## Blueprint inputs

| Input | Description |
| --- | --- |
| **Main Thermostat** | Your real `climate` entity to sync with. |
| **Aqara W100 Climate Controller** | The W100 `climate` entity. |
| **Heating / Cooling / Auto / Fan Mode Support** | Toggle which modes your real system supports. *(Note: "Fan Mode Support" controls **fan-speed** sync, not a `fan_only` HVAC mode.)* |
| **Synchronization Delay** | Delay between sync commands, in seconds (default 2). Avoids command flooding. |
| **Enable External Data Publishing** | Push external temperature/humidity to the W100 display. |
| **External Temperature / Humidity Sensor** | The HA sensors to publish (optional). |

---

## Customizing the mapping

The mode mapping is hardcoded in [`w100-blueprint.yaml`](w100-blueprint.yaml):

- **`dry ↔ auto`** mapping lives in the HVAC-mode sync conditions/actions of both sync
  directions (search for `'dry'` / `'auto'`).
- **Ignored main-only modes** are listed in the `w100_ignored_main_modes` variable
  near the top of the file (currently `fan_only`). Add other modes there to have the
  W100 ignore them too.

---

## Credits

Based on the original
[Aqara W100 external converter + blueprint project](https://github.com/KipK/aqara-w100-z2m-converter)
by KipK. This fork keeps only the blueprint and adapts the HVAC-mode handling
(`dry`/`fan_only`) for thermostats that expose more modes than the W100 supports.

## Disclaimer

- Community-driven, not an official Aqara product.
- Use at your own risk; review the blueprint before deploying in critical environments.

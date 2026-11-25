# Better Thermostat – restart-safe window ventilation

A Home Assistant **automation blueprint** that switches a Better Thermostat to a configurable preset when a window/door stays open, then restores its previous preset and (for `none`) exact target temperature when it closes.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmisch2%2Fhomeassistant-blueprints%2Fblob%2Fmain%2Fbetter_thermostat_window_presets%2Fbetter_thermostat_window_presets.yaml)

## Features

- Original preset and setpoint restored (including manually adjusted targets in `none`).
- Per-room `input_text` snapshot survives Home Assistant restarts.
- Periodic reconciliation every 10 minutes recovers after automation reloads and missed `for:` triggers.
- Configurable open and close delays (defaults: 2 minutes / 30 seconds).
- Snapshot is never overwritten during a ventilation period.
- No integration-specific scripts, no dynamic scenes, no copy/paste between rooms.

## Installation

1. Click **Import Blueprint** above, or manually copy `better_thermostat_window_presets.yaml` to `config/blueprints/automation/<your-username>/better_thermostat_window_presets.yaml` on the Home Assistant host.
2. For **each room**, create a Text helper (`input_text`). Use minimum length `0`, maximum `255`, **no fixed initial value**. Set its *current value* to `idle` **once**, via Developer Tools → Actions → `input_text.set_value`, or through the helper control. Do not set a fixed `initial` value; it would reset the snapshot on restart. A newly created helper whose state is already empty also works.
3. Enable the `away` preset (or your alternative) for the Better Thermostat. Disable Better Thermostat's own window detection for the same sensor.
4. Go to Settings → Automations & scenes → Blueprints; reload blueprints if needed. Create an automation from the blueprint for each room.
5. Disable existing window-control automations for that room to prevent conflicts.

### Example room

- Window sensor: `binary_sensor.pokojicek_blokator_vytapeni`
- Thermostat: `climate.topeni_pracovna_sw`
- Snapshot helper: `input_text.pracovna_topeni_pred_vetranim`
- Ventilation preset: `away`
- Open delay: 2 minutes; close delay: 30 seconds.

## Behavior

| Previous state | While open | After closing |
| --- | --- | --- |
| `none`, 25 °C | `away` | `none`, 25 °C |
| `sleep`, 18 °C | `away` | `sleep` at its configured temperature |
| Already `away` | Unchanged | Unchanged |
| Thermostat `off` | Unchanged | Unchanged |

If the helper is `unknown`, `unavailable`, or contains malformed JSON, the blueprint will **not overwrite it**. Fix the helper manually; don't clear a possibly valuable snapshot without checking.

## Important limitations

- This targets a **heating-only** Better Thermostat (`hvac_mode: heat`). It does not snapshot dual-target `heat_cool` thermostats.
- When ventilation is active, the selected preset is enforced again during periodic reconciliation. Other schedules trying to change the preset can conflict. On close, the *pre-ventilation* state wins.
- `for:` waits are lost during reload/restart; the sensor must have been stable for the configured duration again after restart. Reconciliation checks once per minute, so recovery may be delayed accordingly.
- The helper state is preserved across normal restarts, but no mechanism can guarantee recovery after storage loss, an unclean shutdown before the state is persisted, deleted helpers, or invalid snapshot data.
- A very short sensor bounce during restoration could still create a race; check the automation trace if needed.
- If restoring a preset other than `none`, Better Thermostat applies that preset's **currently configured** target temperature. The snapshot of the old target is intentionally not forced onto an active preset, because a direct temperature change could cancel the preset.
- Depending on Better Thermostat availability/latency, a restore may time out after 15 seconds. The snapshot is retained, so the next reconciliation can retry.

## Publish to GitHub

Use the blueprint file as the published artifact. Once the file has a public GitHub URL, it can be imported into Home Assistant with **Blueprints → Import blueprint**.

## References

- https://www.home-assistant.io/docs/blueprint/schema/
- https://www.home-assistant.io/integrations/input_text/
- https://better-thermostat.org/setup/presets/

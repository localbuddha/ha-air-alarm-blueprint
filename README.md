# Ukraine Air Alarm — Home Assistant blueprint

Air-raid indicator automation for Home Assistant: an indicator light in the colour of the
current alert level, plus voice announcements through TTS.

- **Yellow** = drone threat, **red** = ballistic threat. Red wins over yellow.
- The light stays on for the whole alert and turns off `all_clear_delay` minutes after the
  all clear (with a short fade).
- When an alert ends and only a lower level remains, only the colour changes — no repeated
  announcement.
- On Home Assistant start, if an alert is already active, the speaker reports **how long it
  has been running** (`Повітряна тривога триває N години та M хвилини`) and the light shows
  the current level colour. With no active alert at startup, nothing happens.
- After the all clear the light shows the "idle" colour, then turns off.

## Requirements

- Home Assistant **2026.6** or newer (the `note:` keys and `color_rgb` selector).
- The **Ukraine Alarm** integration, which provides:
  - an aggregate binary sensor that is on while any alert of the category is active;
  - one binary sensor per level (`...(red)`, `...(yellow)`).
- Any color-changeable light entity — a smart bulb, an LED strip (WLED), or a screen —
  plus a `tts` entity and any `media_player` that supports TTS (a Google speaker or other).

## Install

1. Home Assistant → **Settings → Automations & Scenes → Blueprints** → **⋮ → Import blueprint**.
2. Paste this URL:
   `https://raw.githubusercontent.com/localbuddha/ha-air-alarm-blueprint/main/air_alarm_indicator.yaml`
3. Pick your sensors and devices, then **Create**.

If you copy the YAML by hand instead, save it as
`config/blueprints/automation/<your-name>/air_alarm_indicator.yaml` — `!input` is a blueprint
tag and does not work inside a normal automation.

## Inputs

| Input | Type | Default |
|---|---|---|
| `alarm_sensor` | binary_sensor entity | required |
| `red_sensor` | binary_sensor entity | required |
| `yellow_sensor` | binary_sensor entity | required |
| `indicator_light` | target (light / device / area) | required |
| `tts_entity` | tts entity | required |
| `speaker` | media_player entity | required |
| `brightness_pct` | number, % | 25 |
| `color_red` | RGB | `[255, 0, 0]` |
| `color_yellow` | RGB | `[255, 187, 0]` |
| `color_idle` | RGB | `[18, 211, 83]` |
| `all_clear_delay` | minutes | 5 |
| `startup_wait` | minutes, how long to wait at boot for the sensor to have a value | 2 |
| `language` | text | `uk` |
| `msg_red`, `msg_yellow`, `msg_all_clear`, `msg_startup` | text | Ukrainian strings |

Mode is `parallel` (max 5) so a level change during a pending all-clear wait still runs.

## Notes

- **Colour on WLED:** if your strip is running an effect (sound-reactive, palette, etc.), the
  effect overrides the colour — set it to `Solid` for this to behave predictably.
- **Startup duration** is read from `last_changed` of the level sensor, so it survives a
  restart only when Home Assistant restores the sensor state. If the integration re-creates
  the entity instead, the count restarts from boot.
- **Number agreement** in the startup sentence follows Ukrainian rules (1 годину, 2–4 години,
  5+ годин). Changing `msg_startup` changes the prefix only; the word forms stay Ukrainian.

## License

MIT — see `LICENSE`.

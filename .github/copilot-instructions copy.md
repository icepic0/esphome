## Quick orientation

This repository contains ESPHome device configurations (YAML) for an M5Stack Atom Echo voice assistant device. The main configs live under `code/` (for example `code/m5stack-atom-echo-bbc74c.yaml`). Use these files as the single source of truth for device pin mappings, audio pipeline, and wake-word logic.

## Big-picture architecture

- Firmware: ESPHome YAML describes a single ESP32-based device (board: `m5stack-atom`) using the `esp-idf` framework. The device uses I2S for microphone + speaker, and the `voice_assistant` and `micro_wake_word` components for wake-word + STT/TTS flows.
- Integrations: `api:` exposes an ESPhome API for Home Assistant; `ota:` enables over-the-air updates; `wifi:` + `captive_portal:` support initial setup. The device can run wake-word either on-device or delegated to Home Assistant (see the `select` entity `wake_word_engine_location`).
- Key flows: announcements stop the wake-word (see `on_announcement`), scripts `start_wake_word` / `stop_wake_word` coordinate starting/stopping based on `wake_word_engine_location`, and `voice_assistant.on_*` handlers toggle LEDs and control timers.

## Project-specific conventions and patterns

- File location: device YAMLs live in `code/` and are treated as build inputs (e.g. `m5stack-atom-echo-latest.yaml`).
- Substitutions: Each YAML config uses a `substitutions:` top block for `name` and `friendly_name`. Prefer modifying substitutions rather than changing names in multiple places.
- `name: None` usage: Several components use `name: None` to avoid creating a Home Assistant entity name when the entity is intentionally hidden. Keep that pattern if you want internal-only components.
- `disabled_by_default: true`: Used for optional devices (LED, buttons). Tests and deploys assume these flags are deliberate — don't remove unless you intend to expose the entity.
- Scripts & control flow: Reuse `script:` entries like `start_wake_word`, `stop_wake_word`, and `reset_led`. These centralize logic and are invoked from multiple event handlers.

## Integration points & external dependencies

- Home Assistant: The device expects to be paired to HA via `api:`. Some behavior (wake-word location) depends on HA state (`In Home Assistant` vs `On device`).
- External media: Example uses a remote audio file hosted on GitHub raw URLs (see `media_player.files.timer_finished_wave_file`). If you change the file, update the URL or add an asset to a local HTTP server.

## Common tasks & commands (examples)

- Install/run ESPhome (Python/pip):
  - pip install 'esphome>=2025.5.0'
- Validate & compile a YAML:
  - esphome compile code/m5stack-atom-echo-bbc74c.yaml
- Build & upload (serial/OTA):
  - esphome run code/m5stack-atom-echo-bbc74c.yaml
  - esphome upload code/m5stack-atom-echo-bbc74c.yaml --device <ip_or_serial>
- View logs / debug:
  - esphome logs code/m5stack-atom-echo-bbc74c.yaml

Notes: The YAML sets `min_version: 2025.5.0` under `esphome:`. Use an ESPhome CLI or Docker image that satisfies this version.

## Files to inspect for common edits

- `code/m5stack-atom-echo-bbc74c.yaml` — primary example; change pins, I2S settings, mic/speaker config, and wake-word models here.
- Other variants in `orignal_code/` (e.g. `-ww.yaml`, `-wake.yaml`) show alternative configurations and are useful references when adding features.

## Helpful examples to follow

- To change the wake-word engine location default, edit the `select` entry `wake_word_engine_location` (options: `In Home Assistant`, `On device`). The scripts `start_wake_word` / `stop_wake_word` already implement the conditional logic to switch modes.
- To add another wake-word model, add it under `micro_wake_word.models:` — the YAML already contains examples like `okay_nabu`, `hey_mycroft`.

## What not to change casually

- The `substitutions:` block names, `min_version`, and `esp32.board` value: changing these can break builds or change how entities appear in Home Assistant.
- The audio sample rate / I2S pin assignments — these are tuned for the M5Stack Atom hardware.

If any section is unclear or you want the instructions to include build CI steps, tests, or a dashboard recipe, tell me which area to expand and I will iterate.

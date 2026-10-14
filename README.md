# ReSpeaker XVF3800 ESPHome Configs

[![Compile check](https://github.com/jercoates/respeaker-xvf3800-esphome-2026/actions/workflows/compile-check.yml/badge.svg)](https://github.com/jercoates/respeaker-xvf3800-esphome-2026/actions/workflows/compile-check.yml)

Working ESPHome YAML configurations for the **Seeed Studio ReSpeaker XVF3800 4-Mic Array** (with pre-soldered XIAO ESP32-S3) as a Home Assistant voice satellite.

They are based on the excellent work by [formatBCE](https://github.com/formatBCE/Respeaker-XVF3800-ESPHome-integration). This repo keeps one config for current ESPHome and the older configs for people who have not updated, along with a step-by-step setup guide.

---

## Which config to use

Pick the file that matches your **ESPHome** version. What matters is the ESPHome version you compile with, not the Home Assistant version.

| File | ESPHome | What it is |
|---|---|---|
| [`respeaker-xvf3800-shareable.yaml`](respeaker-xvf3800-shareable.yaml) | **2026.9.0 or later** | Current. A short device file that pulls everything else from formatBCE's remote packages, plus a wake word threshold slider. |
| [`respeaker-xvf3800-2026.3-shareable.yaml`](respeaker-xvf3800-2026.3-shareable.yaml) | 2026.3.x | The original single-file config with the 2026.3.x fixes. Tested on ESPHome 2026.3.3. It does not build on ESPHome 2026.9. |

Older configs stay in this repo for anyone on an older ESPHome. The sections further down (from [Installation](#installation) on) were written for the 2026.3 file.

### The current config

- **Setup is the same.** The XMOS firmware (Step 1), `secrets.yaml` (Step 2), power (Step 3) and the installation steps below all apply. Copy the current file instead of the 2026.3 one, and set `name` and `friendly_name` under `substitutions`.
- **No workarounds needed.** formatBCE's upstream now compiles on current ESPHome without the changes listed under [What Was Changed From the Original](#what-was-changed-from-the-original), so the current file does not carry them.
- **Upstream version.** `respeaker_ref` selects it. `main` follows upstream; a full commit SHA freezes the YAML, the C++ components and the XMOS firmware together.
- **New from upstream since March.** A **Beam lock** switch (holds the mic beam on the direction of the wake word until the command ends, off by default), an **LED Ring** light entity, and **DSP route** / gain controls.
- **Changing things.** To change anything that lives in the packages, override it from your device file with `!extend` or `!remove`. See [formatBCE's README](https://github.com/formatBCE/Respeaker-XVF3800-ESPHome-integration#change-the-configuration).

### Wake word threshold slider

The current config adds one thing that is not in formatBCE's packages: a **Wake word threshold** slider on the device page in Home Assistant. It sets how sure the wake word model must be before it triggers.

| Slider | Effect |
|---|---|
| 0 (default) | Slider is off. The **Wake word sensitivity** dropdown is in charge. |
| 1–99 | Overrides the dropdown for all four wake words. **Lower** triggers more easily and gives more false triggers. **Higher** is stricter. |

For reference, the dropdown uses about 85–99 for "Slightly sensitive" and 56–93 for "Very sensitive", depending on the wake word. If you change the dropdown while the slider is above 0, the dropdown wins until you move the slider again or the device restarts.

### Compile check

The badge at the top shows whether the current config still compiles against the latest ESPHome release. The check runs every Monday and on every change to the config. If it is red, a recent ESPHome or upstream change has broken the build.

---

## Hardware

- [Seeed Studio ReSpeaker XVF3800 4-Mic Array with XIAO ESP32-S3](https://www.seeedstudio.com/ReSpeaker-XVF3800-4-Mic-Array-With-XIAO-ESP32S3-p-6489.html)
- XMOS XVF3800 audio processor with 4-mic circular array
- Hardware AEC, beamforming, DoA, noise suppression, AGC
- 360° far-field voice capture up to 5 meters
- JST speaker connector (default) or 3.5mm headphone jack (optional, see below)

---

## Prerequisites

### Step 1 — Flash XMOS Firmware

Before flashing ESPHome, the XVF3800 XMOS chip must be running formatBCE's I2C firmware (`v1.0.7`). The stock Seeed firmware will not work with this ESPHome integration.

**Windows:**

1. Hold the **Mute** button on the board, then plug USB into the **XVF3800 USB-C port** (the one near the 3.5mm jack, NOT the XIAO port) while still holding Mute. The red LED will blink — this is Safe Mode.
2. Open Device Manager. The device will appear under **Other Devices**. Use [Zadig](https://zadig.akeo.ie) to install the **WinUSB** driver on it.
3. Download [dfu-util 0.11](https://dfu-util.sourceforge.net/releases/) and add it to your PATH.
4. Download the firmware binary from this repo: [`application_xvf3800_inthost-lr48-sqr-i2c-v1.0.7-release.bin`](https://github.com/formatBCE/Respeaker-XVF3800-ESPHome-integration/raw/refs/heads/main/application_xvf3800_inthost-lr48-sqr-i2c-v1.0.7-release.bin)
5. Run:
```
dfu-util -R -e -a 1 -D path\to\application_xvf3800_inthost-lr48-sqr-i2c-v1.0.7-release.bin
```

**Linux / macOS:**
```bash
sudo dfu-util -R -e -a 1 -D application_xvf3800_inthost-lr48-sqr-i2c-v1.0.7-release.bin
```

A successful flash shows `Download 100%` and `Resetting USB to switch back to Run-Time mode`.

> **Note:** The config includes an auto-update block that will re-flash the XMOS chip OTA if a different firmware version is detected. You can comment this out in the `respeaker_xvf3800:` section if you prefer to manage firmware manually.

### Step 2 — secrets.yaml

This config uses ESPHome secrets. Make sure your `/config/esphome/secrets.yaml` contains:

```yaml
wifi_ssid: "YourNetworkName"
wifi_password: "YourWiFiPassword"
ota_password: "YourOTAPassword"
api_key: "YourAPIEncryptionKey"
```

The API key and OTA password are generated when you create a new device in ESPHome. If you already created the device, copy the values from the auto-generated YAML.

### Step 3 — Power

The XVF3800 draws more current than the XIAO USB port alone can reliably supply. For stable operation, connect a dedicated USB-C power cable to the **main board power port** in addition to whatever USB connection you use for flashing or debugging.

---

## Installation

1. Copy `respeaker-xvf3800-2026.3-shareable.yaml` into your `/config/esphome/` directory
2. Rename it to whatever you like (e.g. `living-room-jarvis.yaml`)
3. Update `name` and `friendly_name` at the top of the file for each device
4. Make sure your `secrets.yaml` is populated (see above)
5. First flash must be done via USB serial to the **XIAO ESP32-S3 port** — use ESPHome Web at [web.esphome.io](https://web.esphome.io) or the ESPHome dashboard Manual Download option
6. Subsequent updates can be done OTA wirelessly

---

## Wake Words

The config includes four wake word models compiled in simultaneously:

| Wake Word | Default Sensitivity |
|---|---|
| Hey Jarvis | Slightly sensitive (0.97) |
| Okay Nabu | Slightly sensitive (0.85) |
| Kenobi | Default |
| Hey Mycroft | Slightly sensitive (0.99) |
| Stop *(response cancel)* | Internal only — 0.50 |

**Important:** The active wake word is selected in Home Assistant, not in the YAML. After adding the device to HA, go to **Settings → Voice Assistants**, select your assistant, and choose the wake word you want under the satellite's configuration. All four models are listening simultaneously — HA controls which one triggers the pipeline.

Wake word sensitivity can be adjusted at runtime from the HA device page using the **Wake word sensitivity** select entity.

---

## 3.5mm Headphone Jack Output (Optional)

By default, audio is output through the **JST speaker connector**. If you want to use the **3.5mm headphone jack** instead, uncomment the `priority: 600` block in the `on_boot` section of the YAML:

```yaml
  on_boot:
    # - priority: 600
    #   then:
    #     - lambda: |-
    #         i2c::I2CDevice dev;
    #         ...
```

This works by writing directly to the AIC3104 codec registers over I2C at boot, before the audio pipeline initializes. Do not uncomment this if you are using the JST connector — it will redirect audio away from it.

---

## XMOS Firmware Auto-Update

The `respeaker_xvf3800:` component includes a `firmware:` block that automatically checks and updates the XMOS chip to v1.0.7 on boot if a different version is detected. The LED will pulse during the update.

If you manually flashed v1.0.7 and want to disable the auto-update check, comment out the entire `firmware:` block in the `respeaker_xvf3800:` section.

---

## Experimental Dependencies

This config depends on components that are **not yet in mainline ESPHome**:

| Component | Source | Purpose |
|---|---|---|
| `i2s_audio` | formatBCE/esphome@respeaker_microphone | XVF3800 I2S support |
| `respeaker_xvf3800`, `aic3104` | formatBCE/Respeaker-XVF3800-ESPHome-integration@main | Board driver |
| `sendspin`, `media_source`, `const` | kahrendt/esphome@7a6cf5c | Music Assistant streaming |
| `http_request` | esphome/esphome@ff8ce89 | Streaming audio fixes |

These git refs were chosen by formatBCE to work with ESPHome 2026.3.x. If a future ESPHome update breaks compilation, check [formatBCE's repo](https://github.com/formatBCE/Respeaker-XVF3800-ESPHome-integration) for updated refs.

---

## What Was Changed From the Original

The original formatBCE configuration does not compile on ESPHome 2026.3.x. The following changes were required:

| # | Change | Reason |
|---|---|---|
| 1 | `sensor.template.publish` replaced with `lambda` calling `publish_state()` | Action removed in 2026.3.x |
| 2 | `!lambda` tags in `script.execute` parameter blocks replaced with C++ `execute()` calls | Stricter 2026.3.x YAML parser rejects `!lambda` as a tag in parameter position |
| 3 | `!lambda` tags in `homeassistant.event` data blocks replaced with global string buffers | Same YAML parser issue |
| 4 | Three string globals added (`wake_word_buf`, `tts_uri_buf`, `stt_text_buf`) | Required for the `homeassistant.event` dynamic data workaround |
| 5 | 3.5mm jack routing via `i2c::I2CDevice` added (optional, commented out) | Documents the headphone jack option with correct 2026.x I2C API |

---

## Known Issues / Notes

- **I2S timeout warnings** (`ESP_ERR_TIMEOUT`) may appear briefly at boot before the XVF3800 is fully initialized. These are normal and self-resolve within a few seconds.
- **Power requirements**: The XVF3800 needs more current than the XIAO's USB port alone provides. Use the dedicated power port on the main board for stable operation.
- **formatBCE's external component** (`respeaker_microphone` branch) has not been officially updated for ESPHome 2026.3.x but currently works with the fixes in this repo. If ESPHome makes further breaking changes, check formatBCE's repo first.
- **verify_ssl: false** is set in the `http_request` section. This is required for TTS to work on most local HA installs that use self-signed certificates. If your HA uses a valid public certificate, you can remove this line.

---

## Credits

- [formatBCE](https://github.com/formatBCE) — original ReSpeaker XVF3800 ESPHome integration
- [kahrendt](https://github.com/kahrendt) — sendspin, media_source, and micro wake word components
- [Seeed Studio](https://www.seeedstudio.com) — ReSpeaker XVF3800 hardware and reference configuration
- [ESPHome](https://esphome.io) — firmware framework

---

## License

This configuration is based on formatBCE's open source work. Please refer to their repository for licensing terms. Modifications documented in this repo are shared freely for community use.

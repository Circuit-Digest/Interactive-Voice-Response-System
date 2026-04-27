# 📞 Interactive Voice Response System (IVRS) Using ESP32 GeoLinker

[![ESP32-S3](https://img.shields.io/badge/ESP32-S3-blue?style=flat-square)](https://www.espressif.com/)
[![SIM868](https://img.shields.io/badge/GSM-SIM868-green?style=flat-square)]()
[![LittleFS](https://img.shields.io/badge/Storage-LittleFS-orange?style=flat-square)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)

Control electrical devices remotely using just a **phone call** — no internet, no app, no cloud required. This project implements a fully standalone **Interactive Voice Response System (IVRS)** using the [GeoLinker](https://circuitdigest.com/) board (ESP32-S3 + SIM868), operating entirely over the **GSM cellular network**.

---

## 📖 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Components Required](#components-required)
- [What is GeoLinker?](#what-is-geolinker)
- [Block Diagram](#block-diagram)
- [Circuit Diagram & Explanation](#circuit-diagram--explanation)
- [How It Works](#how-it-works)
- [DTMF Key Mapping](#dtmf-key-mapping)
- [Code Explanation](#code-explanation)
- [Expanding the System](#expanding-the-system)
- [Arduino IDE Configuration](#arduino-ide-configuration)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)
- [Related Projects](#related-projects)

---

## Overview

Most remote control systems require a stable internet connection, a mobile app, or a cloud platform. This project solves that limitation by enabling device control through a **normal GSM phone call**.

When a user calls the system:
1. The call is **automatically answered**
2. A **welcome voice message** is played
3. A **voice menu** guides the user through available options
4. The user presses keypad buttons (**DTMF tones**) to control devices
5. The ESP32 responds with **confirmation audio** after each command

> 💡 **Before starting:** Activate your SIM card by following the [How to activate Airtel M2M IoT SimCard with GeoLinker ESP32](https://circuitdigest.com/) tutorial.

---

## Features

- ✅ Works **without internet** — GSM only
- ✅ Auto-answers incoming calls
- ✅ DTMF-based keypad control
- ✅ Pre-recorded voice prompts (LittleFS)
- ✅ Audio played via sigma-delta modulation (GPIO38)
- ✅ Scalable — easily add more outputs
- ✅ Timeout and auto-disconnect on inactivity
- ✅ Confirmation audio feedback after every command

---

## Components Required

| S.No | Component       | Quantity | Purpose |
|------|-----------------|----------|---------|
| 1    | MCP602 Op-Amp  | 1        | Amplifies and filters the audio signal |
| 2    | 47kΩ Resistor   | 1        | Voltage divider to reduce audio signal before MIC input |
| 3    | 2.2kΩ Resistor  | 1        | Works with 47kΩ to scale down microphone input signal |
| 4    | 1nF Capacitor   | 3        | Low-pass filter to remove high-frequency noise |
| 5    | 1µF Capacitor   | 2        | AC coupling and audio smoothing |
| 6    | 100kΩ Resistor  | 2        | Voltage divider for DC biasing |
| 7    | 10kΩ Resistor   | 2        | Filter and feedback network of the op-amp |
| 8    | 0.1µF Capacitor | 1        | Noise filtering and circuit stabilization |
| 9    | GeoLinker Board | 1        | ESP32-S3 main controller for the IVRS system |

---

## What is GeoLinker?

**GeoLinker GL868** is an all-in-one **ESP32 GPS + GSM IoT Development Board** designed for GPS tracking, GSM communication, and cloud connectivity on a single compact PCB.

### Key Features:
- ⚡ **ESP32-S3** — Dual-core 240MHz MCU with WiFi + BLE
- 🛰️ **Built-in GPS (GNSS)** — Supports GPS, GLONASS, and BeiDou
- 📡 **GSM/GPRS (SIM868)** — HTTP, SMS, and voice calls
- 🔋 **LiPo Charging** + power management built-in
- 📳 **Motion Detection** — Accelerometer with wake-on-motion
- 💡 **Onboard RGB + Status LEDs**
- 🔌 **USB-C** for programming and charging

---

## Block Diagram

```
[ User (Mobile Phone) ]
         |
     GSM Network
         |
  [ SIM868 GSM Module ]  ←──── Detects RING, reads DTMF tones
         |  (UART)
  [ ESP32-S3 GeoLinker ]  ────→ [ Audio Filter Circuit ] ──→ SIM868 MIC Input
         |
    [ GPIO Pins ]
         |
  [ Relays / Devices ]
```

> The ESP32-S3 acts as the **brain**, the SIM868 handles **call communication**, and the audio filter circuit ensures **clean voice playback** through the GSM call.

---

## Circuit Diagram & Explanation

The audio conditioning circuit converts the ESP32's **digital audio output (GPIO38)** into a clean analog signal suitable for the SIM868's microphone input.

### Signal Flow:

1. **C1 (1µF)** — Removes DC component from GPIO38 output
2. **R3 & R4 (100kΩ each)** — Create a 1.65V bias voltage from 3.3V (reference for op-amp)
3. **R2 + MCP6002 + C2, C3, C4** — Sallen-Key low-pass filter removes PWM switching noise
4. **C6** — Blocks unwanted DC before forwarding the signal
5. **R5 (47kΩ) + R6 (2.2kΩ)** — Voltage divider reduces signal to microphone level
6. **C5** — Stabilizes and reduces noise at the MIC input
7. Final clean signal → **MIC+ and MIC−** of SIM868

---

## How It Works

1. **User dials** the SIM card number associated with the SIM868 module
2. SIM868 sends `RING` notification to ESP32 via UART
3. ESP32 **auto-answers** after a predefined number of rings
4. DTMF detection is enabled in SIM868
5. ESP32 **plays welcome audio** followed by the **menu prompt** (stored in LittleFS)
6. User presses a key → SIM868 detects the DTMF tone → ESP32 processes the command
7. ESP32 **controls GPIO pins** (relays/devices) accordingly
8. **Confirmation audio** is played (e.g., *"Output 1 turned ON"*)
9. If no input is received within **30 seconds**, the call is auto-terminated
10. User can press `#` to end the call manually (a goodbye message plays)
11. System **resets to idle** and waits for the next call

---

## DTMF Key Mapping

| Key | Action |
|-----|--------|
| `1` | Turn **ON** Output 1 (GPIO4 HIGH) |
| `2` | Turn **OFF** Output 1 (GPIO4 LOW) |
| `3` | Turn **ON** Output 2 (GPIO5 HIGH) |
| `4` | Turn **OFF** Output 2 (GPIO5 LOW) |
| `*` | Repeat menu |
| `#` | End call |

---

## Code Explanation

The firmware is written for **ESP32-S3** using the Arduino framework. Key modules:

### 1. Libraries & Defines
```cpp
#include <Arduino.h>
#include <LittleFS.h>
#include <GL868_ESP32.h>

#define PIN_AUDIO_OUT 38
#define SIM_SERIAL (GeoLinker.modem.getSerial())
#define ANSWER_AFTER_RINGS 1
#define DTMF_WAIT_MS 30000
```
Defines hardware pins, UART communication, and system timing.

### 2. Output & Audio Mapping
```cpp
#define OUT_COUNT 2
static const uint8_t OUT_PINS[OUT_COUNT] = {4, 5};
#define AUD_WELCOME "/audio/welcome.raw"
#define AUD_MENU    "/audio/menu.raw"
```
Maps relay GPIO pins and links each output to its confirmation audio file.

### 3. Audio Playback (Sigma-Delta Modulation)
```cpp
void IRAM_ATTR audio_isr() {
  if (g_audioPlaying) {
    sigmaDeltaWrite(PIN_AUDIO_OUT, g_audioBuf[g_audioPos++]);
  }
}
void audio_init() {
  sigmaDeltaAttach(PIN_AUDIO_OUT, SD_CARRIER_HZ);
  timerAttachInterrupt(g_audioTimer, &audio_isr);
}
```
Plays 8kHz, 8-bit PCM audio files from LittleFS using a **timer ISR** and sigma-delta modulation on GPIO38.

### 4. IVRS State Machine
```cpp
void ivrs_run() {
  switch (g_ivrsState) {
    case IVRS_PLAYING_WELCOME:
      audio_play_file(AUD_WELCOME);
      g_ivrsState = IVRS_PLAYING_MENU;
      break;
  }
}
void ivrs_handle_dtmf(char key) {
  if (key == '1') digitalWrite(OUT_PINS[0], HIGH);
}
```
Manages call flow: welcome → menu → DTMF processing → confirmation → idle.

### 5. GSM Communication & Main Loop
```cpp
void handle_urc(const String &urc) {
  if (urc.indexOf("RING") >= 0) do_answer();
}
void setup() {
  GeoLinker.modem.begin();
  audio_init();
  sim_configure();
}
void loop() {
  if (SIM_SERIAL.available()) handle_urc(sim_read());
  ivrs_run();
}
```
Detects `RING`, answers the call, and continuously runs the IVRS state machine.

---

## Expanding the System

To add more outputs (e.g., Output 3):

1. Add GPIO to `OUT_PINS[]` array:
   ```cpp
   static const uint8_t OUT_PINS[] = {4, 5, 6};
   ```
2. Add corresponding name in `OUT_NAMES[]`
3. Upload new audio files: `output3_on.raw`, `output3_off.raw` to LittleFS
4. Map new DTMF keys:
   - `5` → Turn ON Output 3
   - `6` → Turn OFF Output 3
5. Update the IVRS menu audio to inform users of the new options

> The array-based, state-machine architecture makes this expansion straightforward with **no major code restructuring** needed.

---

## Arduino IDE Configuration

| Setting | Value |
|---------|-------|
| Board | ESP32-S3 |
| CPU Frequency | 240MHz |
| Flash Mode | QIO 80MHz |
| Flash Size | 4MB |
| Partition Scheme | SPIFFS / LittleFS |
| PSRAM | QSPI PSRAM Enabled |
| Upload Speed | 921600 bps |
| USB Mode | UART0 / Hardware CDC |

### Required Setup:
1. Install the **GeoLinker library** from Library Manager
2. Install the **ESP32 LittleFS Uploader** tool
3. Place audio files (`.raw` — 8kHz, 8-bit PCM) inside the `/data/audio/` folder
4. Use the LittleFS uploader to flash audio files to ESP32 memory
5. Flash the main `.ino` sketch

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| SIM868 not responding to AT commands | Check TX/RX pin connections and baud rate. Verify PWRKEY sequence during startup. |
| No audio heard during call | Confirm `.raw` audio files are in LittleFS (8kHz, 8-bit PCM). Check low-pass filter wiring and MIC bias circuit. |
| Audio is distorted or noisy | Verify Sallen-Key filter component values and connections. Check resistor divider to avoid overdriving MIC. Ensure good grounding. |
| System restarts or becomes unstable | Use a stable power supply with sufficient current capacity. Add decoupling capacitors. Avoid powering from weak USB sources. |

---

## FAQ

<details>
<summary><strong>1. What is the main purpose of this project?</strong></summary>
To control electrical devices remotely using a phone call, with voice prompts and keypad input — no internet needed.
</details>

<details>
<summary><strong>2. Why GSM instead of Wi-Fi or IoT platforms?</strong></summary>
GSM works where internet is unavailable or unreliable, providing stable communication using basic cellular networks.
</details>

<details>
<summary><strong>3. What is IVRS?</strong></summary>
Interactive Voice Response System — it plays recorded messages and responds to user keypad inputs via DTMF tones during a call.
</details>

<details>
<summary><strong>4. What are DTMF tones?</strong></summary>
Sounds generated when pressing phone keypad keys. Each key produces a unique frequency combination detected by the SIM868 module.
</details>

<details>
<summary><strong>5. Can this work without internet?</strong></summary>
Yes — it operates entirely over the GSM network with no internet dependency.
</details>

<details>
<summary><strong>6. What devices can be controlled?</strong></summary>
Any electrical device: lights, fans, motors, appliances — typically via relays connected to ESP32 GPIO pins.
</details>

<details>
<summary><strong>7. How is audio played to the user?</strong></summary>
Audio files are stored in ESP32 LittleFS memory and played using sigma-delta modulation, filtered, and fed into the SIM868 MIC input.
</details>

<details>
<summary><strong>8. Can more outputs be added?</strong></summary>
Yes — update the GPIO array, add new DTMF key mappings, upload new audio files, and update the menu prompt.
</details>

<details>
<summary><strong>9. Where can this be used in real life?</strong></summary>
Home automation, agriculture (motor/pump control), industrial automation, security systems, and remote monitoring.
</details>

---

## Related Projects

- 🔗 [Low-Cost IVR Device using Raspberry Pi](https://circuitdigest.com/)
- 🔗 [Arduino-based Voice Controlled LEDs](https://circuitdigest.com/microcontroller-projects/arduino-based-voice-controlled-leds)
- 🔗 [Offline Voice Recognition Module Alternatives](https://circuitdigest.com/tutorial/offline-voice-recognition-module-alternatives-to-vc-02)
- 🔗 [Voice Controlled Home Automation using Arduino](https://circuitdigest.com/microcontroller-projects/voice-controlled-home-automation-using-arduino)

---

## License

This project is open-source and available under the [MIT License](LICENSE).

---

> Made with ❤️ by [Circuit Digest](https://circuitdigest.com/)

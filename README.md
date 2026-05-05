<div align="center">

<h1>ESP-01 Custom PCB Design</h1>

<p><strong>A custom two-layer PCB redesign of the ESP-01 (ESP8266) Wi-Fi module</strong><br/>
Onboard AMS1117-3.3 regulator · Extended GPIO4 & GPIO5 · Breadboard-friendly single-row headers</p>

<p>
  <img src="https://img.shields.io/badge/Status-Design%20Complete%2C%20Not%20Fabricated-orange?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-KiCad-blue?style=flat-square"/>
  <img src="https://img.shields.io/badge/Layers-2--Layer%20PCB-lightgrey?style=flat-square"/>
  <img src="https://img.shields.io/badge/Input-5V%20%2F%203.3V%20Dual-green?style=flat-square"/>
</p>

</div>

---

## Overview

The official ESP-01 is one of the most widely used Wi-Fi modules in hobby electronics — cheap, capable, and carrying the full ESP8266 SoC. But it has two well-known frustrations:

1. **Only 4 GPIO pins** are broken out (GPIO0, GPIO2, TX, RX), even though the ESP8266 has more usable I/O available. GPIO4 and GPIO5 are documented, electrically sound, and confirmed usable — they just never made it to the standard ESP-01 pinout.

2. **3.3V only** — but most hobbyist power sources are 5V. This forces an external regulator every single time.

This design solves both problems, built from scratch in KiCad starting from the official ESP-01 schematic and the ESP8266 Technical Reference Manual.

---

## Features

| Feature | Official ESP-01 | This Design |
|---|---|---|
| GPIO Pins | 4 (GPIO0, GPIO2, TX, RX) | **6** (+ GPIO4, GPIO5) |
| Header Layout | 2×4 dual-row (not breadboard-compatible) | **2× single-row 1×6** (J1 + J2) |
| Power Input | 3.3V only | **5V via AMS1117 OR 3.3V direct** |
| Onboard Regulator | None | **AMS1117-3.3 LDO, 800mA** |
| Total Header Pins | 8 | **12** |
| I2C Support | Limited | **GPIO4 + GPIO5 = dedicated I2C pins** |
| PCB | Original AI-Thinker layout | **KiCad from scratch, two-layer** |

---

## System Block Diagram

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────────────────────────┐
│  5V INPUT   │────▶│  AMS1117-3.3 │────▶│                                     │
│     PAD     │     │  LDO Reg.    │     │           ESP8266 SoC               │
└─────────────┘     └──────────────┘     │                                     │
                                    ┌───▶│  GPIO0, GPIO2, TX (GPIO1), RX (GPIO3│──▶ J2 Header
┌─────────────┐                     │    │  GPIO4 ✦, GPIO5 ✦  (new)            │──▶ J1 Header
│  3.3V INPUT │─────────────────────┘    │  CH_PD, RST, boot resistors         │
│   PAD       │  (bypasses regulator)    └────────────────────┬────────────────┘
└─────────────┘                                               │
   USE ONLY                                                   ▼
   ONE AT A TIME                                         Wi-Fi 802.11 b/g/n
                                                         2.4 GHz PCB Antenna
```

> ✦ GPIO4 and GPIO5 are not present on the official ESP-01 pinout.

---

## Power Paths

### Path A — 5V Input (via AMS1117)
Apply 5V to the `5V_IN` pad. The AMS1117-3.3 LDO regulates it down to a stable 3.3V output with onboard bypass capacitors (10µF + 100nF on both input and output). Use with USB power banks, 5V adapters, or Arduino 5V pins.

### Path B — 3.3V Direct Input
Apply 3.3V to the `3V3_IN` pad. This bypasses the AMS1117 entirely and feeds the 3.3V rail directly. Use with STM32, ESP32, or any already-regulated 3.3V supply.

> ⚠️ **Never connect both input pads simultaneously.**

---

## Pin Reference

### J2 Header — Left Side (Standard Pins)

| Pin | Signal | Notes |
|-----|--------|-------|
| 1 | GND | Common ground |
| 2 | 3V3_IN | 3.3V direct input pad (bypass regulator) |
| 3 | GPIO0 | Pull-up 10kΩ — HIGH = normal boot mode |
| 4 | GPIO2 | Pull-up 10kΩ — HIGH required at boot |
| 5 | TX (GPIO1) | UART TX — serial output |
| 6 | RX (GPIO3) | UART RX — serial input |

### J1 Header — Right Side (Extended Pins)

| Pin | Signal | Notes |
|-----|--------|-------|
| 1 | CH_PD | Pull-up 10kΩ — must be HIGH to operate |
| 2 | RST | Pull-up 10kΩ — active LOW reset |
| 3 | **GPIO4** ✦ | New breakout — not on official ESP-01 |
| 4 | **GPIO5** ✦ | New breakout — not on official ESP-01 |
| 5 | 5V_IN | 5V power input pad (to AMS1117) |
| 6 | GND | Common ground |

---

## Boot Strapping

The ESP8266 samples these pins at power-on to select boot mode. **For normal SPI Flash boot (running user firmware):**

| Pin | Level | Resistor on this PCB |
|-----|-------|----------------------|
| GPIO0 | HIGH | 10kΩ pull-up to 3.3V |
| GPIO2 | HIGH | 10kΩ pull-up to 3.3V |
| GPIO15 | LOW | 10kΩ pull-down to GND |

These resistors are soldered permanently on the PCB — no external wiring needed.

---

## Bill of Materials

| Reference | Qty | Value | Footprint |
|-----------|-----|-------|-----------|
| C1 | 1 | 3.0 pF | C_0402 |
| C2 | 1 | 2.4 pF | C_0402 |
| C3, C4 | 2 | 6.8 pF | C_0402 |
| C5 | 1 | TBD (NC) | C_0402 |
| C6 | 1 | 0.1 µF | C_0402 |
| C7, C10, C11 | 3 | 10 µF | C_0402 |
| C8 | 1 | 1 µF (NC) | C_0402 |
| C9 | 1 | 1 µF | C_0402 |
| D1, D2 | 2 | LED | LED_0603 |
| J1, J2 | 2 | Conn_01x06 | PinHeader_1x06_P2.54mm |
| L1 | 1 | 2.2 nH | L_0402 |
| L2 | 1 | 4.3 nH | L_0402 |
| R1 | 1 | 200 Ω | R_0603 |
| R2 | 1 | 499 Ω | R_0603 |
| R3 | 1 | 12 kΩ | R_0603 |
| R4, R7 | 2 | 10 kΩ | R_0603 |
| R5, R6 | 2 | 1 kΩ | R_0603 |
| U1 | 1 | ESP8266EX | QFN-32 |
| U2 | 1 | W25Q16JVSS | SOIC-8 |
| U3 | 1 | AMS1117-3.3 | SOT-223-3 |
| Y1 | 1 | 26 MHz Crystal | Crystal_SMD_3225-4Pin |

---

## Fabrication Notes

These Gerber files are ready to be submitted to any standard PCB manufacturer (JLCPCB, PCBWay, OSH Park, etc.).

**Recommended PCB specs:**
- Layers: **2**
- Thickness: **1.6mm**
- Copper weight: **1 oz**
- Surface finish: HASL or ENIG
- Solder mask: any colour
- Min trace/space: 0.1mm / 0.1mm

> ⚠️ **This design has not yet been physically fabricated or tested.** Review the schematic and layout carefully before ordering. GPIO4 and GPIO5 availability is confirmed from the ESP8266EX datasheet but should be verified against your specific module batch.

---

## Design Notes

- Designed from scratch in **KiCad** — not a copy of the original AI-Thinker layout.
- **Two-layer stackup:** signal routing on F_Cu, ground/power plane on B_Cu.
- AMS1117 placed close to the 5V pad to minimise input trace inductance.
- Bypass capacitors placed as close as possible to AMS1117 output and ESP8266 VCC.
- Power traces for 3.3V and GND are widened for current capacity.
- KiCad DRC run to verify all clearances, footprint assignments, and net connections.

---

## References

- [ESP8266EX Datasheet](https://www.espressif.com/sites/default/files/documentation/0a-esp8266ex_datasheet_en.pdf)
- [ESP8266 Technical Reference Manual](https://www.espressif.com/sites/default/files/documentation/esp8266-technical_reference_en.pdf)
- [AMS1117 Datasheet](http://www.advanced-monolithic.com/pdf/ds1117.pdf)
- [KiCad EDA](https://www.kicad.org/)

---

## License

Designed by **[Aazan Ali Khan](https://aazanalikhan.vercel.app)** — feel free to use, modify, and build upon it.

📄 [View Full Documentation & Project Page](https://aazanalikhan.vercel.app/projects/project_2/project2.html)

---

<div align="center">
<sub>PCB Design Only · Not Yet Fabricated · Open Source Hardware</sub>
</div>

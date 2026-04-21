# MKS TinyBee — Wiring TODO

Checklist of physical hardware changes required for the MKS Gen L → MKS TinyBee migration.
Firmware is already compiled and verified (`SUCCESS`). All items below are bench/hardware work only.

---

## 1. BLTouch

| Wire | Old connection | New connection | Status |
|------|---------------|---------------|--------|
| Servo / control (orange+brown+red) | SERVO0 | SERVO0 (GPIO2) — **no change** | ☐ |
| Signal / trigger (black+white) | Z_MIN (GPIO32) | GPIO35 header on TinyBee | ☐ |
| Z mechanical endstop | Z_STOP (GPIO22) | Z_STOP (GPIO22) — **no change** | ☐ |

> **Note:** GPIO32 conflicts with Y_STOP on TinyBee. The signal wire must be moved to GPIO35 (input-only pin, suitable for probe signal).

---

## 2. Fans

| Fan | Old pin | New header | Status |
|-----|---------|-----------|--------|
| Part cooling fan (FAN0) | D9 | FAN0 header on TinyBee | ☐ |
| E0 auto-fan (FAN1) | D8 | FAN1 header on TinyBee | ☐ |
| Controller fan | D5 | **Disabled in firmware** — disconnect or leave unconnected | ☐ |

> **Note:** `USE_CONTROLLER_FAN` is disabled. FAN1 is reserved for E0 auto-fan (virtual pin 148).

---

## 3. RGB LED — Remove / Disconnect

| Item | Action | Status |
|------|--------|--------|
| RGB LED wiring (old GPIO25 / GPIO27 / GPIO29) | Disconnect — pins are used by I2S stepper stream | ☐ |

> **Note:** NeoPixel strip (30px) on EXP1_06 (GPIO16) replaces status lighting.

---

## 4. NeoPixel Strip

| Item | Connection | Status |
|------|-----------|--------|
| NeoPixel data | EXP1 pin 6 (GPIO16) | ☐ |
| NeoPixel power / GND | 5V + GND from board | ☐ |
| Pixel count | 30 pixels configured in firmware | ☐ |

---

## 5. VREF Calibration (TMC2209 Standalone)

All drivers are in standalone mode — current is set physically via VREF potentiometer.

Formula: `VREF = I_RMS × 2.5 × R_sense` — typical R_sense = 0.11Ω → `VREF ≈ I_RMS × 0.275 × 2`

| Axis | Target (mA RMS) | Expected VREF (V) | Measured VREF (V) | Status |
|------|----------------|-------------------|-------------------|--------|
| X    | 440 mA         | ~0.770 V          |                   | ☐ |
| Y    | 440 mA         | ~0.770 V          |                   | ☐ |
| Z    | 440 mA         | ~0.770 V          |                   | ☐ |
| Z2   | 440 mA         | ~0.770 V          |                   | ☐ |
| E0   | 600 mA         | ~1.050 V          |                   | ☐ |

> **Procedure:** Power board without motors enabled. Measure VREF between the potentiometer wiper and GND. Adjust trimmer until target voltage is reached. Do one axis at a time, starting with X.

---

## 6. Flash & Functional Validation

### Flash
- [ ] `pio run -e mks_tinybee --target upload`

### Post-flash Checks

| Test | Command | Expected result | Status |
|------|---------|----------------|--------|
| Endstops | `M119` | All endstops OPEN (no trigger) | ☐ |
| BLTouch deploy | `M280 P0 S10` | Probe pin extends | ☐ |
| BLTouch stow | `M280 P0 S90` | Probe pin retracts | ☐ |
| Pin states | `M43` | No unexpected signals | ☐ |
| Short X move | `G91 ; G1 X10 F1000` | Smooth movement, correct direction | ☐ |
| Short Y move | `G91 ; G1 Y10 F1000` | Smooth movement, correct direction | ☐ |
| Short Z move | `G91 ; G1 Z5 F300` | Smooth movement, correct direction | ☐ |
| Home all | `G28` | All axes home correctly | ☐ |
| Bed leveling | `G29` | Mesh probe completes without error | ☐ |
| Display | — | Screen on, encoder responds, click works | ☐ |
| NeoPixel | `M150 R255 G0 B0` | Strip lights up red | ☐ |
| E0 fan trigger | Heat hotend to threshold | FAN1 activates automatically | ☐ |

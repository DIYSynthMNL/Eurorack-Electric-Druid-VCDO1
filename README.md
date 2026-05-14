This module is shared under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International license](https://creativecommons.org/licenses/by-nc-sa/4.0/). All credits go to Tom Wiltshire, founder of Electric Druid.

# Eurorack Electric Druid VCDO1

![Electric Druid VCDO1 assembled](photos/front.jpg)

A Eurorack wavetable oscillator module built around the [VCDO1](https://electricdruid.net/product/vcdo-wavetable-oscillator/) chip from Electric Druid. The VCDO1 is a digital wavetable + sub-oscillator chip in an 18-pin DIP, running its own DAC internally and exposing both main and sub waveform outputs.

## Features

- **16 main waveforms** with smooth morphing via the Wave Select control
- **8 sub waveforms** with 4 selectable octave offsets (+1 oct, unison, −1 oct, −2 oct)
- **V/oct Note CV** input (0–5V, 5 octaves) with V/oct gain trim and offset trim
- **Exponential FM CV** input (±5V) with zero trim
- **Wave Select CV** for main and sub waveforms (±5V each)
- **Bitcrush** amount with both panel control and CV input
- **Glide (portamento)** time with both panel control and CV input, plus a separate Glide on/off gate input
- Two simultaneous outputs: Main oscillator and Sub oscillator (10Vpp each, anti-aliased)
- Eurorack ±12V power via either 16-pin IDC or 3-pin JST connector

The module adds **CV-controllable bitcrush and glide amounts** on top of the original Electric Druid VCDO datasheet circuit.

## Inputs and outputs

| Jack | Range | Notes |
|---|---|---|
| Note CV | 0–5V | 1V/oct, scaled to chip's 0–5V range; trim pots adjust V/oct and offset |
| FM CV | ±5V (10Vpp) | Inverted, scaled, offset-shifted to 0–5V; zero trim |
| Main Wave Select CV | ±5V (10Vpp) | Inverted, scaled, offset-shifted to 0–5V |
| Sub Wave Select CV | ±5V (10Vpp) | Inverted, scaled, offset-shifted to 0–5V |
| Bitcrush CV | ±5V (10Vpp) | Mixed with front-panel Bitcrush Control via Amount attenuator |
| Glide Time CV | ±5V (10Vpp) | Mixed with front-panel Glide Time Control via Amount attenuator |
| Glide on/off CV | 0–5V (gate) | NPN-buffered, drives chip's digital glide input |
| Main Out | 10Vpp | 4-pole Bessel reconstruction filter (≈18 kHz), buffered, 1K series |
| Sub Out | 10Vpp | Identical filter chain |

## Block diagram

```
                                  ┌───────────────────┐
  Note CV ─► [V/oct + offset] ──► │                   │
   FM CV ─► [scale + zero] ─────► │                   │ Main ─► [4-pole Bessel LP] ─► Main Out
  MainSel ─► [scale + offset] ──► │                   │
  SubSel ─► [scale + offset] ───► │   VCDO1 chip      │
 Bitcrush ─► [pot + CV+amount] ─► │   (digital        │
GlideTime ─► [pot + CV+amount] ─► │    wavetable      │
Glide on/off ─► [NPN inverter] ─► │    oscillator)    │ Sub  ─► [4-pole Bessel LP] ─► Sub Out
                                  │                   │
  Pots (Course, Fine,             │                   │
   MainSel, SubSel) ────────────► │                   │
                                  └───────────────────┘

  Eurorack ±12V ─► U6 (78M05) / U7 (79M05) ─► ±5V rails
```

## Power

- Eurorack ±12V in via **J3** (3-pin JST) or **J4** (16-pin IDC) — populate one
- Local U6 (78M05) generates +5V from +12V
- Local U7 (79M05) generates −5V from −12V
- R45 / R46 (10K) are optional minimum-load resistors

> **Note on the Electric Druid datasheet:** The original VCDO1 datasheet circuit has an error in the negative 5V regulator wiring. This schematic corrects it — verify when copying from the datasheet directly.

## Calibration

The board has three trim pots that need adjustment for accurate tracking. You'll need: a multimeter, a calibrated reference tone source (a tuner app, another tuned VCO, or a 1V/oct keyboard).

**RV1 — V/oct Adjust** sets the per-octave gain of the Note CV.
**RV2 — Offset trim** sets the DC offset of the Note CV (the "zero" pitch when 0V is applied).
**RV3 — FM Zero trim** ensures no DC drift on the FM input when nothing is patched.

Procedure:

1. **Warm up.** Power the module for at least 5 minutes — the temperature affects the chip and trim accuracy.
2. **Set baseline.** Freq Course and Fine at 12 o'clock. Nothing patched.
3. **Tune the base pitch.** Apply 0V to the Note CV jack (or short the jack tip to GND). Adjust **RV2 (Offset trim)** until the main output matches a known low reference (e.g., C2 = 65.41 Hz).
4. **Tune the per-octave gain.** Apply +4V to the Note CV jack. Adjust **RV1 (V/oct)** until the main output is exactly 4 octaves higher than step 3 (e.g., C6 = 1046.5 Hz).
5. **Iterate.** Step 3–4 interact slightly; bounce between them until both are accurate.
6. **Zero the FM input.** With Note CV at 0V and nothing patched into FM, adjust **RV3 (FM Zero trim)** until probing AIN_Freq_Mod_CV (or the FM jack tip) shows ~0V DC.

If tracking drifts at the extremes (very low or very high notes), check the timing capacitor area on the VCDO1 (this is a chip-internal characteristic).

## References

Local archived copies live in [`references/`](references/) so this repo stays useful if the upstream links die.

- **VCDO1 datasheet (Tom Wiltshire / Electric Druid, 2015)** — [local copy](references/VCDO1-Datasheet-ElectricDruid-2015.pdf) · [upstream](https://electricdruid.net/wp-content/uploads/2015/07/VCDO-Datasheet.pdf) — chip pinout, example circuit, and waveform reference. Note the negative-5V-regulator wiring error called out above; this schematic corrects it.
- [Electric Druid VCDO1 product page](https://electricdruid.net/product/vcdo-wavetable-oscillator/)

## Build status

What's ready for builders today, and what's still on the TODO list:

**Production assets** (what you need to actually fabricate and assemble a final unit)

- [x] Schematic — Rev 0.1.5 ([PDF](Schematic%20PDFs/Electric%20Druid%20VCDO1_Multiboard_Schematic_0.1.5.pdf))
- [ ] PCB layout — in progress — multiboard layout in `kicad/`, not yet separated for fab
- [ ] Gerber files for fabrication — none yet
- [x] BOM — [BOMs - VCDO1 - Schematic Rev 0.1.2.pdf](BOMs/BOMs%20-%20VCDO1%20-%20Schematic%20Rev%200.1.2.pdf)
- [ ] Final front panel (SVG/PDF for fab) — none yet
- [x] License — [LICENSE](LICENSE)

**Prototype assets** (for breadboard / perfboard / 3D-printed-panel builds before final PCB)

- [x] 3D-printed prototype panel STL — [VCDO.stl](3D%20printed%20front%20panel/VCDO.stl)

**Documentation**

- [x] Photos of the assembled module — see [photos/](photos/)
- [ ] Demo video — none yet
- [ ] Build / assembly instructions — none yet

Want to help fill a gap (build photos, gerbers, an assembly guide)? Open an issue or PR.

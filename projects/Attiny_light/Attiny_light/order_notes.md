# Attiny_light rev 2 — order notes

Last updated 2026-10-31. Schematic: `Attiny_light.kicad_sch` (rev 2, ERC clean).

## Before ordering — open questions

- [ ] **Is the cell already protected?** (protected 18650 = slightly longer, ring/board at the − end; pouch LiPo = small board under the tape at the tabs)
  - Protected → don't buy/fit U3, Q3, R10, R11, C6; **bridge JP1** with solder.
  - Bare cell → fit them, leave JP1 **open**.
- [ ] **Cell chemistry**: the cell is AA-size (**14500**), measured 3.3 V when empty (2026-10-31). The label must say **3.7 V Li-ion**.
  If it says **3.2 V / IFR / LiFePO4**, do NOT use this charger: TP4054 charges to 4.2 V and LiFePO4 maxes out at 3.6 V.
- [x] **Cell capacity** → R3 (charge current = 1000 V / R3, keep ≤ 0.5 × capacity):
  - 14500 cells are typically 600–900 mAh → **3k = 333 mA** ← chosen
  - 2k = 500 mA is too much for a 14500
- [ ] **Board size**: measure the original board (or the spot it sits in) with calipers before ordering. The design is Ø19 mm; an AA tube itself is only ~15 mm inside.
- [ ] **FS8205A pinout**: the footprint follows EVVO FS8205A (SOT-23-6): 1 S1, 2 D, 3 S2, 4 G2, 5 D, 6 G1. Other "8205A" makers use different pinouts. Check the datasheet of the exact part you buy.
- [ ] **ATtiny85 package**: must be **ATtiny85-20SU** (or ATtiny85V-10SU), the **208-mil / 5.3 mm wide SOIC ("8S2")**. The normal 150-mil SOIC-8 does not fit.

## LED measurements (2026-10-31)

| LED | What it is | Measured |
|---|---|---|
| A | strip of 5 small LEDs | ~25 mA at ~3.0 V |
| B | single bigger LED | ~25–35 mA at ~3.0 V |

- Both are common anode (shared + wire to L+).
- R13 = **33 Ω** (shared; only one LED is on at a time). Expected current: ~33 mA at 4.2 V (full), ~20 mA at 3.7 V, ~12 mA at 3.4 V. Can't overdrive the LEDs even at 100 % PWM.
- Brighter option: 22 Ω gives ~47 mA at 4.2 V and ~30 mA at 3.7 V. Then the firmware must cap the PWM duty when the battery is full.
- Original modes: LED X flicker, LED X steady, LED Y steady, off. Wire either LED to L1/L2; the firmware decides which channel flickers.

## PCB order settings

| Setting | Value | Why |
|---|---|---|
| Layers | 2 | |
| Shape | round, Ø19 mm | |
| **Thickness** | **1.0 mm** (1.2 mm also OK) | USB-C shell tabs go through the board and only fit 1.0–1.2 mm. **Not the 1.6 mm default.** |
| Surface finish | ENIG preferred (HASL OK) | flat pads for 0402 parts and pogo pins |
| Min track / space | 0.15 / 0.15 mm | |
| Vias | 0.3 mm drill / 0.6 mm pad | |

Very small boards: check the fab's minimum board size; some ask for a panel.

## Bill of materials

| Ref | Qty | Value | Package | Part / notes |
|---|---|---|---|---|
| J1 | 1 | USB-C power-only, vertical | 6-pin | Same Sky **UJC-VP-3-SMT-TR** (footprint made from its datasheet) |
| U1 | 1 | TP4054 | SOT-23-5 | pins: 1 CHRG, 2 GND, 3 BAT, 4 VCC, 5 PROG |
| U2 | 1 | ATtiny85-20SU | SOIC-8 **208 mil** | see above |
| U3 | 1 | DW01A | SOT-23-6 | only for an unprotected cell |
| Q1, Q2 | 2 | AO3400A | SOT-23 | LED switches |
| Q3 | 1 | FS8205A | SOT-23-6 | only for an unprotected cell; check pinout |
| R1, R2 | 2 | 5.1 kΩ | 0402 | USB-C CC pull-downs (without them C-to-C chargers give no power) |
| R3 | 1 | 3 kΩ (or 2 kΩ) | 0402 | charge current |
| R4 | 1 | 47 Ω | 0402 | MCU supply filter |
| R5, R7 | 2 | 100 Ω | 0402 | gate resistors |
| R6, R8 | 2 | 100 kΩ | 0402 | gate pull-downs |
| R9 | 1 | 1 kΩ | 0402 | button series |
| R10 | 1 | 100 Ω | 0402 | protection only |
| R11 | 1 | 1 kΩ | 0402 | protection only |
| R12 | 1 | 10 kΩ | 0402 | reset pull-up (optional) |
| R13 | 1 | 33 Ω | 0603 | LED current |
| C1 | 1 | 4.7 µF 10 V X5R | 0603 | USB input |
| C2, C4 | 2 | 10 µF 10 V X5R | 0603 | charger output, MCU bulk |
| C3 | 1 | 100 nF | 0402 | MCU decoupling |
| C5 | 1 | 10 nF | 0402 | button filter (optional) |
| C6 | 1 | 100 nF | 0402 | protection only |
| JP1 | – | solder jumper | – | bridge only if protection is NOT fitted |
| TP1–TP8 | – | pads | – | wires: B+, B−, PB4, SW, SW_GND, L+, L1, L2 |

0402 parts are tiny and easy to lose: buy at least 10 of each. If you hand-solder, a stencil and paste make it much easier.

For programming: a USBasp (or an Arduino as ISP) plus 6 pogo pins on a 2.54 mm grid for the bottom ISP pads.

## Bring-up (one step at a time, test after each)

1. **No power:** check for shorts with a multimeter: VBUS–GND, B+–B−, VCC–GND.
2. **USB only, no battery:** expect 5 V at C1. TP4054 CHRG blinks (no battery present), so PB3 toggles.
3. **Battery on a bench supply** (3.7 V, current limit 50 mA): VCC ≈ 3.7 V, idle current small.
   If the protection is fitted and nothing has power, plug in USB once: DW01 boards sometimes start in the cut-off state.
4. **Flash fuses** (battery out, or programmer set to not power the board):
   `avrdude -c usbasp -p t85 -B 10 -U lfuse:w:0x62:m -U hfuse:w:0xDD:m -U efuse:w:0xFF:m`
   (1 MHz internal clock, BOD 2.7 V, RESET kept, SPI programming kept). Use `-B 10` because a 1 MHz chip needs a slow ISP clock.
   Don't press the button while flashing (PB2 = SCK).
5. **Firmware, one feature at a time:** button + one LED → mode cycling → PWM flicker → charge detect → low-battery cut-off → sleep.
6. **Sleep current:** measure it with USB unplugged; it should be in the µA range. Turn on the PB3 (CHRG) pull-up only while awake, because it may leak into the unpowered TP4054.

# Attiny_light rev 2 — order notes

Last updated 2026-11-03. Schematic: `Attiny_light.kicad_sch` (rev 2, ERC clean).

## Before ordering — open questions

- [x] **Battery protection**: decided 2026-11-03 to **always fit** U3, Q3, R10, R11 and C6, and leave JP1 **open**, without relying on whether the cell has its own protection. Two protections in series are harmless: about +50 mΩ and +3 µA. JP1 is only a fallback bypass.
- [ ] **Cell chemistry** (unknown, from the cheapest rechargeable light): the cell is AA-size (**14500**), 3.3 V when found empty (2026-10-31). Cheap USB-charged lights almost always use **3.7 V Li-ion**, but check before charging it to 4.2 V:
  1. **Label first:** "3.7 V", "Li-ion", "ICR"/"INR" means OK. "3.2 V", "IFR", "LiFePO4" means **don't use this charger** (TP4054 charges to 4.2 V; LiFePO4 maximum is 3.6 V).
  2. **No readable label:** test the cell on its own with a bench supply set to **3.60 V, 100 mA limit**. That's safe for both chemistries. Charge until the current drops below about 15 mA, disconnect, wait 1 hour, then measure:
     - It reached 3.6 V quickly (under about an hour) and **rests at about 3.55 V or more** → **Li-ion**, safe for this board.
     - It charged for **several hours** and **rests around 3.35–3.45 V** → **LiFePO4**, don't use it with this board.
  - Stay nearby while it charges. Don't try to "find the max voltage" with a 4.2 V charger until you know the chemistry.
- [x] **Cell capacity** → R3 (charge current = 1000 V / R3, keep ≤ 0.5 × capacity):
  - 14500 cells are typically 600–900 mAh → **3k = 333 mA** ← chosen
  - 2k = 500 mA is too much for a 14500
- [x] **Board size**: Ø19 mm confirmed (2026-11-03). The board sits where the original did; the cell itself is narrower.
- [x] **FS8205A pinout**: checked 2026-11-03 against the datasheet of the part being ordered (EVVO FS8205A, SOT23-6L, marking "8205A", `FS8205A-new.pdf`): 1 S1, 2 D, 3 S2, 4 G2, 5 D, 6 G1. It matches the board. If you ever buy an "8205A" from another maker, check again, because pinouts differ.
- [x] **ATtiny85 package**: ordering **ATtiny85-20SU**, the **208-mil / 5.3 mm wide SOIC ("8S2")**. The normal 150-mil SOIC-8 does not fit.

## LED measurements (2026-10-31)

| LED | What it is | Measured |
|---|---|---|
| A | strip of 5 small LEDs | ~25 mA at ~3.0 V |
| B | single bigger LED | ~25–35 mA at ~3.0 V |

- Both are common anode (shared + wire to L+).
- R13 = **33 Ω** (shared; only one LED is on at a time). Expected current: ~33 mA at 4.2 V (full), ~20 mA at 3.7 V, ~12 mA at 3.4 V. Can't overdrive the LEDs even at 100 % PWM.
- Brighter option: 22 Ω gives ~47 mA at 4.2 V and ~30 mA at 3.7 V. Then the firmware must cap the PWM duty when the battery is full.
- Original modes: LED X flicker, LED X steady, LED Y steady, off. Wire either LED to L1/L2; the firmware decides which channel flickers.

## PCB layout status (2026-11-03)

- Ø19 mm, 2 layers, all 36 parts placed, fully routed, GND pour on both layers.
- KiCad DRC: 0 errors, 0 unconnected, matches the schematic. 8 warnings remain: the wire pads and the PB4 test pad differ from the library on purpose, because their silk outlines were removed to make room for the labels.
- **Top (USB-C side):** USB-C, charger, ATtiny85, both LED MOSFETs, switch wire pads **SW** and **GND** at the right edge, PB4 test pad.
- **Bottom:** wire pads B+, B−, L1, L2, L+, battery protection, ISP pogo pads (square pad = pin 1, MISO).
- Before ordering:
  - [ ] Print the board 1:1 (KiCad *File → Print*, 100 %) and lay the real USB-C connector on it. Its footprint was drawn from the datasheet, and KiCad has no 3D model for it.
  - [ ] Check that the bottom pads clear the USB-C shell tabs that poke through (the two oval slots at the top edge).
  - [ ] JP1 stays open: the protection chips are always fitted.

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

## Files to send to the producer

Everything is in `fab/`, generated from the final board on 2026-11-03.

| File | Who needs it | What it is |
|---|---|---|
| **`Attiny_light_rev2_gerbers.zip`** | **upload this to the PCB maker** | Gerbers for both copper, mask, silkscreen and paste layers plus the board outline, and the Excellon drill file (17 vias and the 2 plated USB-C slots) |
| `Attiny_light_rev2_BOM.csv` | you (shopping list) | parts grouped by value; bare pads and the jumper are left out |
| `assembly_top.pdf`, `assembly_bottom.pdf` | you (soldering map) | part outlines, reference designators and pad numbers, 4× scale; the bottom view is mirrored as you see it |
| `gerbers/` | – | the same files as the zip, unpacked |

For bare boards, the zip is the only file the producer needs; set thickness, finish and so on in their order form (table above).

- **Stencil (optional):** the paste layers are already in the zip. Order a stencil with the boards if you want to use solder paste.
- **Slot check:** the USB-C needs two **plated slots** (0.5 × 1.1 mm). Most fabs accept 0.5 mm as the minimum plated slot width. If the online viewer shows round holes there instead of slots, ask them before paying.
- **If you regenerate after a change:** in KiCad *Fabrication Outputs* use Gerber layers F/B.Cu, F/B.Mask, F/B.Silkscreen, F/B.Paste and Edge.Cuts. For the drill file use Excellon, mm, and "alternate" mode for oval holes.

## Bill of materials

| Ref | Qty | Value | Package | Part / notes |
|---|---|---|---|---|
| J1 | 1 | USB-C power-only, vertical | 6-pin | Same Sky **UJC-VP-3-SMT-TR** (footprint made from its datasheet) |
| U1 | 1 | TP4054 | SOT-23-5 | pins: 1 CHRG, 2 GND, 3 BAT, 4 VCC, 5 PROG |
| U2 | 1 | ATtiny85-20SU | SOIC-8 **208 mil** | see above |
| U3 | 1 | DW01A | SOT-23-6 | battery protection (always fitted) |
| Q1, Q2 | 2 | AO3400A | SOT-23 | LED switches |
| Q3 | 1 | FS8205A | SOT-23-6 | battery protection (always fitted); check pinout |
| R1, R2 | 2 | 5.1 kΩ | 0402 | USB-C CC pull-downs (without them C-to-C chargers give no power) |
| R3 | 1 | 3 kΩ (or 2 kΩ) | 0402 | charge current |
| R4 | 1 | 47 Ω | 0402 | MCU supply filter |
| R5, R7 | 2 | 100 Ω | 0402 | gate resistors |
| R6, R8 | 2 | 100 kΩ | 0402 | gate pull-downs |
| R9 | 1 | 1 kΩ | 0402 | button series |
| R10 | 1 | 100 Ω | 0402 | battery protection |
| R11 | 1 | 1 kΩ | 0402 | battery protection |
| R12 | 1 | 10 kΩ | 0402 | reset pull-up (optional) |
| R13 | 1 | 33 Ω | 0603 | LED current |
| C1 | 1 | 4.7 µF 10 V X5R | 0603 | USB input |
| C2, C4 | 2 | 10 µF 10 V X5R | 0603 | charger output, MCU bulk |
| C3 | 1 | 100 nF | 0402 | MCU decoupling |
| C5 | 1 | 10 nF | 0402 | button filter (optional) |
| C6 | 1 | 100 nF | 0402 | battery protection |
| JP1 | – | solder jumper | – | leave **open**; bridging it bypasses the protection |
| TP1–TP8 | – | pads | – | wires: B+, B−, PB4, SW, SW_GND, L+, L1, L2 |
| J2 | – | pads | – | ISP pogo pads, no part |

0402 parts are tiny and easy to lose: buy at least 10 of each. If you hand-solder, a stencil and paste make it much easier.

## Programming the ATtiny85

**Not over USB-C.** The ATtiny85 has no USB hardware, and the USB-C on this board is a 6-pin power-only part: it has no data lines at all. A USB bootloader (Micronucleus, as on the Digispark) would need:
- a 16-pin USB-C with D+/D−;
- two Zener diodes and three resistors;
- PB3 and PB4 given over to USB, but PB3 is already the charge input;
- a 16.5 MHz clock, which is out of spec at battery voltage;
- 2 KB of the 8 KB flash.

That doesn't fit a Ø19 mm board, so firmware goes in through the 6 **ISP pogo pads** on the bottom.

**Tools to buy:**
- [ ] **USBasp programmer** (cheap clone is fine). It has a 10-pin plug, so get a **10-pin → 6-pin AVR ISP adapter** too if one isn't included. An Arduino Uno or Nano running the "ArduinoISP" sketch also works.
- [ ] **Pogo-pin adapter for a 2 × 3, 2.54 mm grid**: sold as a "2.54 mm 2x3 6P pogo pin programming clip / test clamp". Or make one: 6 pogo pins (e.g. P75-E2) pushed through 2.54 mm perfboard, wired to a 6-pin female header. Hold it on the pads for the few seconds of flashing.
- [ ] *(optional)* **ATtiny85 in DIP-8 + breadboard**, to start writing firmware before the boards arrive. Use the same wiring: LEDs on PB0/PB1, button PB2 to GND.
- [ ] *(optional)* **SOIC-8 test clip** (the cheap "25-series flash" clip fits the 208-mil package): clips straight onto U2, if the parts next to it leave room.
- Software: **avrdude** from the Arduino IDE, or PlatformIO in VS Code (installs in your home folder, nothing system-wide).

**How to flash:** take the battery out. Set the USBasp's supply jumper to **3.3 V** (5 V also works: everything on the board is rated for it). Press the pogo adapter on with the **square pad = pin 1**, and use `-B 10`. The LEDs may flicker while flashing because MOSI/MISO also drive the LED MOSFETs; that's normal.

## Bring-up (one step at a time, test after each)

1. **No power:** check for shorts with a multimeter: VBUS–GND, B+–B−, VCC–GND.
2. **USB only, no battery:** expect 5 V at C1. TP4054 CHRG blinks (no battery present), so PB3 toggles.
3. **Battery on a bench supply** (3.7 V, current limit 50 mA): VCC ≈ 3.7 V, idle current small.
   If the protection is fitted and nothing has power, plug in USB once: DW01 boards sometimes start in the cut-off state.
4. **Flash fuses** (battery out, USBasp powering the board at 3.3 V; see *Programming the ATtiny85*):
   `avrdude -c usbasp -p t85 -B 10 -U lfuse:w:0x62:m -U hfuse:w:0xDD:m -U efuse:w:0xFF:m`
   (1 MHz internal clock, BOD 2.7 V, RESET kept, SPI programming kept). Use `-B 10` because a 1 MHz chip needs a slow ISP clock.
   Don't press the button while flashing (PB2 = SCK).
5. **Firmware, one feature at a time:** button + one LED → mode cycling → PWM flicker → charge detect → low-battery cut-off → sleep.
6. **Sleep current:** measure it with USB unplugged; it should be in the µA range. Turn on the PB3 (CHRG) pull-up only while awake, because it may leak into the unpowered TP4054.

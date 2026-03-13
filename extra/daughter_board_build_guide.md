# FSAE Display System — Daughter Board (Button/LED Panel) KiCad 9.0 Build Guide

> Full schematic, wiring diagrams, exact KiCad library references, and step-by-step
> instructions for the **Daughter Board (Button/LED Panel)**.
>
> This board connects to the Main PCB via an 8-pin Molex Micro-Fit 3.0 harness.

---

## Table of Contents

1. [Bill of Materials & KiCad Library Map](#1-bill-of-materials--kicad-library-map)
2. [Full Schematic & Net List](#2-full-schematic--net-list)
3. [Step-by-Step: Building the Schematic in KiCad 9.0](#3-step-by-step-building-the-schematic-in-kicad-90)
4. [Step-by-Step: PCB Layout in KiCad 9.0](#4-step-by-step-pcb-layout-in-kicad-90)
5. [Design Rules & Manufacturing Notes](#5-design-rules--manufacturing-notes)

---

## 1. Bill of Materials & KiCad Library Map

### 1.1 Daughter Board Components

| Ref | Component | Value | KiCad Symbol Library : Symbol | KiCad Footprint Library : Footprint | Notes |
|-----|-----------|-------|-------------------------------|-------------------------------------|-------|
| SW1 | Tactile Button | 12mm | `Switch:SW_Push` | `Button_Switch_THT:SW_PUSH_6mm_H5mm` | Screen select (GPIO17) |
| SW2 | Tactile Button | 12mm | `Switch:SW_Push` | `Button_Switch_THT:SW_PUSH_6mm_H5mm` | Record (GPIO27) |
| SW3 | Tactile Button | 12mm | `Switch:SW_Push` | `Button_Switch_THT:SW_PUSH_6mm_H5mm` | Up/navigate (GPIO22) |
| LED3 | LED (Green 5mm) | 5mm | `Device:LED` | `LED_THT:LED_D5.0mm` | Power indicator |
| LED4 | LED (Blue 5mm) | 5mm | `Device:LED` | `LED_THT:LED_D5.0mm` | CAN indicator |
| LED5 | LED (Red 5mm) | 5mm | `Device:LED` | `LED_THT:LED_D5.0mm` | Record indicator |
| R5, R6, R7 | Resistor | 330Ω | `Device:R` | `Resistor_SMD:R_0805_2012Metric` | LED current limit |
| R8, R9, R10 | Resistor | 10kΩ | `Device:R` | `Resistor_SMD:R_0805_2012Metric` | Button pullup to 3.3V |
| C12, C13, C14 | Capacitor | 100nF | `Device:C` | `Capacitor_SMD:C_0805_2012Metric` | Button debounce caps |
| J6 | Daughter Board Connector | Molex Micro-Fit 3.0 8-pin | `Connector_Generic:Conn_02x04_Odd_Even` | `Connector_Molex:Molex_Micro-Fit_3.0_43045-0812_2x04_P3.00mm_Vertical` | Male PCB header; mates with J2 on main PCB. **Part:** Molex 43045-0812. **DigiKey:** WM1785-ND. For harness: Molex 43025-0800 receptacle (DigiKey WM2014-ND), 43030-0007 female crimp pins (DigiKey WM1142CT-ND; 8 per cable). |

### 1.2 Power Symbols

| Symbol | KiCad Power Library : Symbol |
|--------|------------------------------|
| +3V3 | `power:+3V3` |
| GND | `power:GND` |

---

## 2. Full Schematic & Net List

### 2.1 Connector Pinout (J6)

The daughter board receives all power and signals through a single 8-pin Molex
Micro-Fit 3.0 connector (J6) that mates with J2 on the main PCB.

```
    J6 Molex Micro-Fit 3.0 (mating connector to J2 on main PCB)

    Pin 1: LED_DB1   ← GPIO23 from Pi   → Green LED driver
    Pin 2: LED_DB2   ← GPIO24 from Pi   → Blue LED driver
    Pin 3: LED_DB3   ← GPIO5 from Pi    → Red LED driver
    Pin 4: BTN1      → GPIO17 on Pi     ← Screen button
    Pin 5: BTN2      → GPIO27 on Pi     ← Record button
    Pin 6: BTN3      → GPIO22 on Pi     ← Up button
    Pin 7: +3V3      ← 3.3V power rail
    Pin 8: GND       ← Ground
```

### 2.2 Full Schematic

```
DAUGHTER BOARD SCHEMATIC
=========================

    J6 Molex Micro-Fit 3.0 (mating connector to J2 on main PCB)
    ┌─────────────────────────────────────────────────────────────┐
    │  Pin 1: LED_DB1      Pin 2: LED_DB2                        │
    │  Pin 3: LED_DB3      Pin 4: BTN1                           │
    │  Pin 5: BTN2         Pin 6: BTN3                           │
    │  Pin 7: +3V3         Pin 8: GND                            │
    └────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┘
         │      │      │      │      │      │      │      │
     LED_DB1 LED_DB2 LED_DB3 BTN1  BTN2   BTN3  +3V3   GND
         │      │      │      │      │      │      │      │
         │      │      │      │      │      │      │      │
    ┌────┘      │      │      │      │      │      │      │
    │           │      │      │      │      │      │      │
    ▼           ▼      ▼      │      │      │      │      │
   R5(330Ω)  R6(330Ω) R7(330Ω)│     │      │      │      │
    │           │      │      │      │      │      │      │
    ▼           ▼      ▼      │      │      │      │      │
   LED3(G)   LED4(B) LED5(R)  │      │      │      │      │
   Anode      Anode   Anode   │      │      │      │      │
    │           │      │      │      │      │      │      │
   LED3       LED4   LED5    │      │      │      │      │
   Cathode    Cath.  Cath.   │      │      │      │      │
    │           │      │      │      │      │      │      │
    └───────────┴──────┴──────┼──────┼──────┼──────┼──────┘
                              │      │      │      │    (all to GND)
                              │      │      │      │
                              │      │      │   +3V3
                              │      │      │      │
                              ▼      ▼      ▼      │
                             SW1    SW2    SW3     │
                           (SCRN) (REC)  (UP)      │
                              │      │      │      │
                             GND    GND    GND     │
                                                   │
    Pullup resistors:                              │
                                                   │
    BTN1 ──── R8 (10kΩ) ──── +3V3 ◄───────────────┤
    BTN2 ──── R9 (10kΩ) ──── +3V3 ◄───────────────┤
    BTN3 ──── R10(10kΩ) ──── +3V3 ◄───────────────┘

    Debounce caps (optional but recommended):
    BTN1 ──── C12 (100nF) ──── GND
    BTN2 ──── C13 (100nF) ──── GND
    BTN3 ──── C14 (100nF) ──── GND
```

### 2.3 How the Circuits Work

**Buttons:** Each button line is pulled HIGH to +3V3 through a 10kΩ resistor.
When the button is pressed, it shorts to GND, pulling the line LOW. The Pi
reads LOW = pressed, HIGH = released. The 100nF caps filter contact bounce.

**LEDs:** The Pi GPIO pin drives HIGH (3.3V) through the 330Ω resistor into
the LED anode. Current flows through the LED to GND.
I_LED ≈ (3.3V − V_fwd) / 330Ω ≈ (3.3 − 2.0) / 330 ≈ 4mA (safe for GPIO).

### 2.4 Daughter Board Net List

| Net Name | Nodes (Ref:Pin) |
|---|---|
| LED_DB1 | J6:1, R5:1 |
| LED_DB2 | J6:2, R6:1 |
| LED_DB3 | J6:3, R7:1 |
| BTN1 | J6:4, SW1:1, R8:1, C12:1 |
| BTN2 | J6:5, SW2:1, R9:1, C13:1 |
| BTN3 | J6:6, SW3:1, R10:1, C14:1 |
| +3V3 | J6:7, R8:2, R9:2, R10:2 |
| GND | J6:8, LED3:K, LED4:K, LED5:K, SW1:2, SW2:2, SW3:2, C12:2, C13:2, C14:2 |
| LED3_A | R5:2, LED3:A |
| LED4_A | R6:2, LED4:A |
| LED5_A | R7:2, LED5:A |

---

## 3. Step-by-Step: Building the Schematic in KiCad 9.0

### 3.1 Create the Project

1. Open **KiCad 9.0**
2. **File → New Project**
3. Name: `FSAE_Display_DaughterBoard`
4. Choose your project directory
5. KiCad creates `.kicad_pro`, `.kicad_sch`, and `.kicad_pcb` files

### 3.2 Open the Schematic Editor

1. Double-click the `.kicad_sch` file
2. **File → Page Settings** → Set paper size to A4 (daughter board is simple enough)
3. Fill in the title block

### 3.3 Add Power Symbols

1. Press **P** (or **Place → Power Port**)
2. Place `+3V3` and `GND` symbols

### 3.4 Place the Connector

1. Press **A** → search `Conn_02x04_Odd_Even` → place **J6**
2. Label pins using net labels (press **L**):
   - Pin 1: `LED_DB1`
   - Pin 2: `LED_DB2`
   - Pin 3: `LED_DB3`
   - Pin 4: `BTN1`
   - Pin 5: `BTN2`
   - Pin 6: `BTN3`
   - Pin 7: connect to `+3V3` power symbol
   - Pin 8: connect to `GND` power symbol

### 3.5 Place LED Circuits

For each LED (repeat 3 times for LED3, LED4, LED5):

1. Press **A** → search `R` → place resistor (R5, R6, R7 = 330Ω)
2. Press **A** → search `LED` → place LED (LED3, LED4, LED5)
3. Wire: `LED_DBx` net label → R → LED Anode → LED Cathode → `GND`

### 3.6 Place Button Circuits

For each button (repeat 3 times for SW1, SW2, SW3):

1. Press **A** → search `SW_Push` → place switch (SW1, SW2, SW3)
2. Press **A** → search `R` → place pullup resistor (R8, R9, R10 = 10kΩ)
3. Press **A** → search `C` → place debounce cap (C12, C13, C14 = 100nF)
4. Wire:
   - `BTNx` net label → SW pin 1 AND R pullup pin 1 AND cap pin 1
   - SW pin 2 → `GND`
   - R pullup pin 2 → `+3V3`
   - Cap pin 2 → `GND`

### 3.7 Add Power Flags, Annotate & Check

**Power Flags on the Daughter Board:**

The daughter board receives +3V3 and GND through connector J6 (passive pins).
KiCad's ERC doesn't know that these nets are powered externally by the main
PCB, so you need PWR_FLAG symbols to suppress the "pin not driven" errors.

```
POWER FLAG PLACEMENT DIAGRAM (Daughter Board)
===============================================

You need exactly TWO PWR_FLAG symbols on the daughter board.

    ┌─── Net: +3V3 ─────────────────────────────────────────┐
    │                                                        │
    │         J6 Pin 7 (+3V3)                                │
    │              │                                         │
    │              ├──── R8, R9, R10 pullups                  │
    │              │                                         │
    │         ┌────┴────┐                                    │
    │         │PWR_FLAG │  ◄── Place here, on the +3V3 net   │
    │         └─────────┘      at or near J6 pin 7           │
    │                                                        │
    │  WHY: +3V3 comes from the main PCB through J6, which   │
    │  has passive-type pins. PWR_FLAG tells ERC that this    │
    │  net is externally powered.                            │
    └────────────────────────────────────────────────────────┘

    ┌─── Net: GND ──────────────────────────────────────────┐
    │                                                        │
    │         J6 Pin 8 (GND)                                 │
    │              │                                         │
    │              ├──── LED cathodes, button pins, caps      │
    │              │                                         │
    │         ┌────┴────┐                                    │
    │         │PWR_FLAG │  ◄── Place here, on the GND net    │
    │         └─────────┘      at or near J6 pin 8           │
    │                                                        │
    └────────────────────────────────────────────────────────┘
```

**PWR_FLAG summary for Daughter Board:**

| PWR_FLAG # | Attach To Net | Place Near | Reason |
|---|---|---|---|
| 1 | +3V3 | J6 pin 7 | 3.3V from main PCB enters via Molex connector (passive) |
| 2 | GND | J6 pin 8 | Ground from main PCB enters via Molex connector (passive) |

**Now annotate and run ERC:**

1. **Tools → Annotate Schematic** → click **Annotate**
2. **Inspect → Electrical Rules Check (ERC)** → click **Run ERC**
3. All ERC errors should now be resolved. If you still see "pin not driven"
   warnings, verify the PWR_FLAG symbols are properly wired (not floating).

### 3.8 Assign Footprints

1. **Tools → Assign Footprints** opens the Footprint Assignment tool with
   three panels (library list, your components, footprints in selected library).

2. Work through each component row and assign footprints. Here's the
   complete walkthrough:

```
FOOTPRINT ASSIGNMENT WALKTHROUGH (Daughter Board)
===================================================

Center Panel (your components)     →    Footprint to Assign
──────────────────────────────────────────────────────────────

  SW1, SW2, SW3 : SW_Push           →    Button_Switch_THT : SW_PUSH_6mm_H5mm
       How: Select all three SW rows (Ctrl+click). In left panel,
       expand "Button_Switch_THT". In right panel, double-click
       "SW_PUSH_6mm_H5mm". All three get assigned at once.

  LED3, LED4, LED5 : LED (5mm)      →    LED_THT : LED_D5.0mm
       How: Select all three LED rows, expand "LED_THT",
       pick "LED_D5.0mm".

  R5, R6, R7 : Resistor (330Ω)     →    Resistor_SMD : R_0805_2012Metric
  R8, R9, R10 : Resistor (10kΩ)    →    Resistor_SMD : R_0805_2012Metric
       How: Select all six resistor rows, assign R_0805 in bulk.

  C12, C13, C14 : Capacitor (100nF) →    Capacitor_SMD : C_0805_2012Metric
       How: Select all three, assign C_0805 in bulk.

  J6 : Conn_02x04 (Molex)           →    Connector_Molex : Molex_Micro-Fit_3.0_43045-0812_2x04_P3.00mm_Vertical
       How: Expand "Connector_Molex", search "Micro-Fit". Part: Molex 43045-0812 (DigiKey WM1785-ND).

  PWR_FLAG symbols                   →    (No footprint needed — virtual)
```

3. Verify no rows have empty footprint fields.
4. Click **Apply, Save Schematic & Continue** (or **OK**).

### 3.9 Push to PCB

1. **Tools → Update PCB from Schematic** (F8)

---

## 4. Step-by-Step: PCB Layout in KiCad 9.0

### 4.1 Board Outline

1. Select **Edge.Cuts** layer
2. Draw a rectangle: **60mm × 40mm** (adjust to fit your enclosure face)
3. Add four **mounting holes** (M3) at corners:
   - Place → Footprint → search `MountingHole_3.2mm_M3_Pad`
   - Position ~3mm inset from each corner

### 4.2 Component Placement

The daughter board mounts to the enclosure face with buttons and LEDs
protruding through cut-outs. The Molex connector faces the back/interior.

```
┌──────────────────────────────────────────────────────┐
│                DAUGHTER BOARD (top view)              │
│                                                      │
│    (M3)                                      (M3)    │
│                                                      │
│      ┌─────┐       ┌─────┐       ┌─────┐            │
│      │ SW1 │       │ SW2 │       │ SW3 │            │
│      │SCRN │       │ REC │       │ UP  │            │
│      └─────┘       └─────┘       └─────┘            │
│                                                      │
│       (●)           (●)           (●)                │
│       LED3          LED4          LED5               │
│       PWR           CAN           REC                │
│      (green)       (blue)        (red)               │
│                                                      │
│    (M3)     [J6 Molex 8-pin (on back)]       (M3)   │
│                                                      │
└──────────────────────────────────────────────────────┘

BACK SIDE (SMD components):
- R5, R6, R7 (330Ω)  — behind each LED
- R8, R9, R10 (10kΩ) — near each button
- C12, C13, C14 (100nF) — near each button
- J6 Molex connector — bottom edge
```

**Placement steps:**

1. **Buttons (SW1–SW3):** Place across the top half, evenly spaced. These
   protrude through the enclosure panel — ensure spacing matches your
   enclosure cut-out pattern.
2. **LEDs (LED3–LED5):** Place below each button, centered.
3. **J6 (Molex connector):** Place on the bottom edge of the board (facing
   the back/inside of the enclosure).
4. **SMD resistors and caps:** Place on the back copper layer (B.Cu) directly
   behind their associated button or LED for short trace lengths.

### 4.3 Set Design Rules

1. **File → Board Setup → Design Rules → Constraints:**
   - Minimum track width: **0.25mm**
   - Minimum clearance: **0.2mm**
2. **Net Classes:**
   - Default: 0.3mm track width (sufficient for all signals on this board)

### 4.4 Route Traces

This board is simple point-to-point wiring. Use 0.3mm traces for everything.

1. Route LED_DB1/2/3 from J6 pins → resistors → LED anodes
2. Route BTN1/2/3 from J6 pins → button pin 1, pullup resistors, debounce caps
3. Route +3V3 from J6 pin 7 → all pullup resistors (R8, R9, R10)
4. Route GND from J6 pin 8 → LED cathodes, button pin 2s, cap negatives

### 4.5 Add Ground Plane

1. Select **B.Cu** (back copper layer)
2. **Place → Add Filled Zone** → draw around board outline
3. Set net to `GND`
4. Press **B** to fill

### 4.6 Add Silkscreen Labels

1. Select **F.Silkscreen**
2. Add labels:
   - "FSAE Display — Daughter Board v1.0"
   - "SCRN", "REC", "UP" near buttons
   - "PWR", "CAN", "REC" near LEDs
   - Pin 1 indicator near J6

### 4.7 Run DRC & Generate Gerbers

1. **Inspect → Design Rules Check** → fix all errors
2. **File → Fabrication Outputs → Gerbers** → generate files
3. **Generate Drill Files** → Excellon format
4. Zip and upload to manufacturer

---

## 5. Design Rules & Manufacturing Notes

### 5.1 PCB Specifications

| Parameter | Value |
|---|---|
| Layers | 2 (F.Cu + B.Cu) |
| Board thickness | 1.6mm |
| Copper weight | 1 oz (35µm) |
| Min trace width | 0.25mm (10 mil) |
| Min clearance | 0.2mm (8 mil) |
| Min drill | 0.3mm |
| Solder mask | Green |
| Silkscreen | White |
| Surface finish | HASL |
| Board size | ~60mm × 40mm (adjust to enclosure) |

### 5.2 Power Budget (Daughter Board Only)

| Consumer | Current Draw |
|---|---|
| 3× LEDs (4mA each) | ~12 mA |
| 3× Pullup resistors (0.33mA each when pressed) | ~1 mA |
| **Total** | **~13 mA** |

All power comes from the +3V3 rail via the Molex connector. This is a
negligible load on the Pi's 3.3V regulator.

### 5.3 Enclosure Integration Notes

- **Button actuator height:** Choose buttons with actuator height that suits
  your enclosure panel thickness. For a 2mm acrylic panel, use buttons with
  ≥5mm actuator length.
- **LED bezels:** Consider using 5mm LED panel mount bezels for a clean
  appearance. These snap into a 7mm hole drilled in the enclosure face.
- **Mounting standoffs:** Use M3 × 6mm standoffs between the daughter board
  and the enclosure panel. This sets the board back from the panel surface
  by 6mm, which works well with standard button and LED heights.
- **Molex harness:** Crimp a Molex Micro-Fit 3.0 wire harness using **Molex 43025-0800** (2×4 receptacle housing, DigiKey WM2014-ND) and **Molex 43030-0007** (female crimp terminals, DigiKey WM1142CT-ND; 8 per cable). Typical harness length: 150mm–300mm depending on enclosure layout.

### 5.4 Assembly Notes

1. **Solder SMD components** (resistors, caps) on the back side first
2. **Solder through-hole components** (buttons, LEDs, J6) on the front side
3. **Test each LED:** Apply 3.3V through a 330Ω resistor to J6 pins 1/2/3
   and verify LEDs light up
4. **Test each button:** Use a multimeter in continuity mode — verify each
   button shorts its J6 pin to GND when pressed, and reads HIGH (~3.3V)
   via pullup when released
5. **Connect to main PCB** via Molex harness and verify GPIO functionality:
   ```bash
   # Quick GPIO test on Pi (using gpiod tools)
   # Read button states:
   gpioget gpiochip0 17   # BTN1 (should read 1, goes to 0 on press)
   gpioget gpiochip0 27   # BTN2
   gpioget gpiochip0 22   # BTN3

   # Toggle LEDs:
   gpioset gpiochip0 23=1  # LED3 on
   gpioset gpiochip0 24=1  # LED4 on
   gpioset gpiochip0 5=1   # LED5 on
   ```

---

## Appendix A: KiCad Library Quick Reference (Daughter Board)

**Symbol Chooser:**

| Search Term | Library:Symbol Found |
|---|---|
| `SW_Push` | Switch : SW_Push |
| `LED` | Device : LED |
| `R` | Device : R |
| `C` | Device : C |
| `Conn_02x04` | Connector_Generic : Conn_02x04_Odd_Even |
| `+3V3` | power : +3V3 |
| `GND` | power : GND |
| `PWR_FLAG` | power : PWR_FLAG |

**Footprint Chooser:**

| Search Term | Library : Footprint |
|---|---|
| `SW_PUSH_6mm` | Button_Switch_THT : SW_PUSH_6mm_H5mm |
| `LED_D5` | LED_THT : LED_D5.0mm |
| `R_0805` | Resistor_SMD : R_0805_2012Metric |
| `C_0805` | Capacitor_SMD : C_0805_2012Metric |
| `Micro-Fit_3.0` | Connector_Molex : Molex_Micro-Fit_3.0_43045-0812_2x04_P3.00mm_Vertical |
| `MountingHole_3.2mm` | MountingHole : MountingHole_3.2mm_M3_Pad |

---

## Appendix B: Wiring Harness Pinout (J2 ↔ J6)

This table shows the wire-for-wire mapping between the main PCB connector (J2)
and the daughter board connector (J6). Use this when crimping the Molex harness.

| Wire # | J2 Pin (Main PCB) | J6 Pin (Daughter) | Signal | Wire Color (Suggested) |
|---|---|---|---|---|
| 1 | Pin 1 | Pin 1 | LED_DB1 (Green LED) | Green |
| 2 | Pin 2 | Pin 2 | LED_DB2 (Blue LED) | Blue |
| 3 | Pin 3 | Pin 3 | LED_DB3 (Red LED) | Red |
| 4 | Pin 4 | Pin 4 | BTN1 (Screen) | White |
| 5 | Pin 5 | Pin 5 | BTN2 (Record) | Yellow |
| 6 | Pin 6 | Pin 6 | BTN3 (Up) | Orange |
| 7 | Pin 7 | Pin 7 | +3V3 Power | Red (thick) |
| 8 | Pin 8 | Pin 8 | GND | Black |

> **Note:** The Molex Micro-Fit 3.0 connector is keyed, so it can only be
> inserted one way. Pin 1 is indicated by a triangle/arrow on the housing.
> Double-check pin numbering matches between your crimp tool and the
> datasheet before crimping.

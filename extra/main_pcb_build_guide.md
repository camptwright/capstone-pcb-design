# FSAE Display System — Main PCB (Pi Shield) KiCad 9.0 Build Guide

> Full schematic, wiring diagrams, exact KiCad library references, and step-by-step
> instructions for the **Main PCB (Pi Shield)**.
>
> **GPS Module:** SparkFun NEO-M9N Chip Antenna (Qwiic) — GPS-15733
>
> **Display:** DigiWise HU-043WISBUAA1-B 4.3" HDMI TFT (800×480, IPS, capacitive touch)

---

## Table of Contents

1. [Bill of Materials & KiCad Library Map](#1-bill-of-materials--kicad-library-map)
2. [Full Schematic & Net List](#2-full-schematic--net-list)
3. [Custom Symbol & Footprint Creation](#3-custom-symbol--footprint-creation)
4. [Step-by-Step: Building the Schematic in KiCad 9.0](#4-step-by-step-building-the-schematic-in-kicad-90)
5. [Step-by-Step: PCB Layout in KiCad 9.0](#5-step-by-step-pcb-layout-in-kicad-90)
6. [Design Rules & Manufacturing Notes](#6-design-rules--manufacturing-notes)
7. [Raspberry Pi 40-Pin Pinout Reference](#7-raspberry-pi-40-pin-pinout-reference)

---

## 1. Bill of Materials & KiCad Library Map

Every component below maps to a **KiCad 9.0 default library symbol** and
**footprint**. Where a default symbol does not exist, the table says **CUSTOM**
and Section 3 shows how to create it.

### 1.1 Main PCB Components

| Ref | Component | Value | KiCad Symbol Library : Symbol | KiCad Footprint Library : Footprint | Notes |
|-----|-----------|-------|-------------------------------|-------------------------------------|-------|
| U1 | CAN Controller | MCP2515-I/SO | `Interface_CAN_LIN:MCP2515-xSO` | `Package_SO:SOIC-18W_7.5x11.6mm_P1.27mm` | SPI CAN controller |
| U2 | CAN Transceiver | SN65HVD230DR | `Interface_CAN_LIN:SN65HVD230` | `Package_SO:SOIC-8_3.9x4.9mm_P1.27mm` | 3.3V CAN transceiver |
| U3 | Buck Converter IC | MP1584EN-LF-Z | Download from [Ultra Librarian](https://www.ultralibrarian.com/) → `MP1584EN-LF-Z` | `Package_SO:SOIC-8-1EP_3.9x4.9mm_P1.27mm_EP2.29x3mm` | 12V→5V step-down. 8-pin SOIC + exposed pad. Use KiCad standard library. Alt: use pre-built module (see §3.2) |
| Y1 | Crystal | 8 MHz | `Device:Crystal` | `Crystal:Crystal_HC49-4H_Vertical` | For MCP2515 oscillator |
| D1 | Schottky Diode | SS34 | `Device:D_Schottky` | `Diode_SMD:D_SMA` | Reverse polarity protection on 12V input |
| D2 | Schottky Diode | SS34 | `Device:D_Schottky` | `Diode_SMD:D_SMA` | Buck converter freewheeling diode |
| F1 | Fuse | 3A | `Device:Fuse` | `Fuse:Fuseholder_Clip-5x20mm_Littelfuse_111_Inline_P20.00x5.00mm_D1.05mm_Horizontal` | Input protection. Alt: `Fuseholder_Cylinder-5x20mm_Bulgin_FX0457_Horizontal_Closed` for enclosed holder |
| L1 | Inductor | 33µH / 3.5A | `Device:L` | `Inductor_SMD:L_Bourns_SRR1260` | Buck converter output inductor. Part: Bourns SRR1260-330M (12.5×12.5mm, 33µH, 3.5A sat). DigiKey: SRR1260-330MCT-ND. |
| LED1 | LED (Green) | 3mm | `Device:LED` | `LED_THT:LED_D3.0mm` | Power status (on-board) |
| LED2 | LED (Blue) | 3mm | `Device:LED` | `LED_THT:LED_D3.0mm` | CAN status (on-board) |
| R1 | Resistor | 330Ω | `Device:R` | `Resistor_SMD:R_0805_2012Metric` | LED1 current limit |
| R2 | Resistor | 330Ω | `Device:R` | `Resistor_SMD:R_0805_2012Metric` | LED2 current limit |
| R3 | Resistor | 120Ω | `Device:R` | `Resistor_SMD:R_0805_2012Metric` | CAN bus termination |
| R4 | Resistor | 10kΩ | `Device:R` | `Resistor_SMD:R_0805_2012Metric` | MCP2515 RESET pullup |
| R_FB1 | Resistor | 100kΩ | `Device:R` | `Resistor_SMD:R_0805_2012Metric` | Buck FB upper (sets Vout=5V) |
| R_FB2 | Resistor | 19.1kΩ | `Device:R` | `Resistor_SMD:R_0805_2012Metric` | Buck FB lower (sets Vout=5V) |
| C1, C2 | Capacitor | 22pF | `Device:C` | `Capacitor_SMD:C_0805_2012Metric` | Crystal load caps |
| C3, C4, C5, C6 | Capacitor | 100nF | `Device:C` | `Capacitor_SMD:C_0805_2012Metric` | Decoupling (C3: U1 VDD, C4: U2 VCC, C5: U3 VIN bypass, C6: general 3.3V rail) |
| C7 | Capacitor | 22µF / 25V | `Device:C` | `Capacitor_SMD:C_1210_3216Metric` | Buck input cap (ceramic X5R/X7R MLCC; if unavailable in 1210 at 25V, use electrolytic + THT footprint) |
| C8 | Capacitor | 22µF / 10V | `Device:C` | `Capacitor_SMD:C_1210_3216Metric` | Buck output cap (ceramic X5R/X7R MLCC) |
| C9 | Capacitor | 100nF | `Device:C` | `Capacitor_SMD:C_0805_2012Metric` | Buck bootstrap cap (BST to SW) |
| C_COMP | Capacitor | 10nF | `Device:C` | `Capacitor_SMD:C_0805_2012Metric` | Buck COMP pin compensation cap |
| C10 | Capacitor | 10µF / 16V | `Device:C` | `Capacitor_SMD:C_0805_2012Metric` | Bulk decoupling 3.3V rail (ceramic MLCC) |
| C11 | Capacitor | 10µF / 16V | `Device:C` | `Capacitor_SMD:C_0805_2012Metric` | Bulk decoupling 5V rail (ceramic MLCC) |
| JP1 | Jumper Header | 2-pin + shunt | `Connector_Generic:Conn_01x02` | `Connector_PinHeader_2.54mm:PinHeader_1x02_P2.54mm_Vertical` | CAN termination enable. Header: Sullins PREC002SAAN-RC (DigiKey S1011EC-02-ND). Shunt: Sullins STC02SYAN (DigiKey S9001-ND). |
| J1 | 40-pin RPi Header | 2×20 Female socket | `Connector_Generic:Conn_02x20_Odd_Even` | `Connector_PinSocket_2.54mm:PinSocket_2x20_P2.54mm_Vertical` | Shield: female socket plugs onto Raspberry Pi 4 GPIO. **Part:** Sullins PPPC202LFBN-RC. **DigiKey:** S7057-ND. |
| J2 | Daughter Board Connector | Molex Micro-Fit 3.0 8-pin | `Connector_Generic:Conn_02x04_Odd_Even` | `Connector_Molex:Molex_Micro-Fit_3.0_43045-0812_2x04_P3.00mm_Vertical` | Male PCB header, 8-pin to daughter board. **Part:** Molex 43045-0812. **DigiKey:** WM1785-ND. |
| J3 | GPS Header | 1×6 Male 2.54mm | `Connector_Generic:Conn_01x06` | `Connector_PinHeader_2.54mm:PinHeader_1x06_P2.54mm_Vertical` | SparkFun NEO-M9N mount (see §3.3). Male pin header; breakout with female connector plugs onto this. **Part:** Sullins PREC006SAAN-RC. **DigiKey:** S1011EC-06-ND. |
| J4 | CAN Bus Terminal | 2-pos Screw | `Connector:Screw_Terminal_01x02` | `TerminalBlock_Phoenix:TerminalBlock_Phoenix_MKDS-1,5-2-5.08_1x02_P5.08mm_Horizontal` | CAN_H, CAN_L only (2-wire CAN). Board GND is from power input J5. **Part:** Phoenix Contact 1715721 (MKDS 1.5/2-5.08). **DigiKey:** 277-1651-ND. |
| J5 | Power Input Terminal | 2-pos Screw | `Connector:Screw_Terminal_01x02` | `TerminalBlock_Phoenix:TerminalBlock_Phoenix_MKDS-1,5-2-5.08_1x02_P5.08mm_Horizontal` | 12V, GND from car. **Part:** Phoenix Contact 1715721 (MKDS 1.5/2-5.08). **DigiKey:** 277-1651-ND. |
| J6 | (Optional) 2-pin header | — | `Connector_Generic:Conn_01x02` | `Connector_PinHeader_2.54mm:PinHeader_1x02_P2.54mm_Vertical` | If used for auxiliary connection. **Part:** Sullins PREC002SAAN-RC. **DigiKey:** S1011EC-02-ND. |
| J7 | Display Power + USB Header | 4-pos | `Connector_Generic:Conn_01x04` | `Connector_PinHeader_2.54mm:PinHeader_1x04_P2.54mm_Vertical` | Pin 1: GND, Pin 2: +5V, Pin 3: D+ (DISP_USB_DP), Pin 4: D− (DISP_USB_DM) for HU-043WISBUAA1-B display. **Part:** Sullins PREC004SAAN-RC. **DigiKey:** S1011EC-04-ND. |

### 1.2 Power Symbols (used on both boards)

| Symbol | KiCad Power Library : Symbol |
|--------|------------------------------|
| +12V | `power:+12V` |
| +5V | `power:+5V` |
| +3V3 | `power:+3V3` |
| GND | `power:GND` |

---

## 2. Full Schematic & Net List

### 2.1 Block Diagram

```
  12V IN ─► [D1 Schottky] ─► [F1 Fuse] ─► [U3 MP1584EN Buck 12V→5V] ─► +5V Rail
                                                                           │
                                                   ┌──────────────────────┤
                                                   │                      │
                                                   ▼                      ▼
                                            Raspberry Pi 4          ┌──────────────┐
                                          (via J1 40-pin header)    │ J7: Display  │
                                                   │                │ 5V Power Out │
                                                   │                │(HU-043WISBU) │
                              ┌────────────────────┼─────────┐      └──────────────┘
                              │ SPI                │ UART    │
                              ▼                    ▼         ▼
                         ┌─────────┐         ┌──────────┐  ┌──────────┐
                         │   U1    │         │  J3 GPS  │  │ J2 Daugh │
                         │ MCP2515 │         │ NEO-M9N  │  │  Board   │
                         └────┬────┘         │ Breakout │  │ 8-pin    │
                              │              └──────────┘  └──────────┘
                              ▼
                         ┌─────────┐
                         │   U2    │
                         │SN65HVD  │
                         │  230    │──── J4 CAN Bus ──── To Car
                         └─────────┘
```

### 2.2 Power Supply Section

The buck converter uses the **MP1584EN-LF-Z** from Monolithic Power Systems.
Download the symbol, footprint, and 3D model from
[Ultra Librarian](https://www.ultralibrarian.com/) (search "MP1584EN-LF-Z").

**MP1584EN-LF-Z actual pinout (from datasheet / Ultra Librarian):**

```
          ┌───────────┐
   SW  ──┤ 1       8 ├── BST
   EN  ──┤ 2       7 ├── VIN
  COMP ──┤ 3       6 ├── FREQ
   FB  ──┤ 4       5 ├── GND
          └─────┬─────┘
                │ EPAD (GND)
```

```
SCHEMATIC: Power Input & Buck Converter
=========================================

        J5 (Screw Terminal)
        Pin 1: +12V_IN
        Pin 2: GND
              │
              │    D1 (SS34 Schottky)               F1 (3A Fuse)
   +12V_IN ──┤──►│Anode ──── Cathode│──►──────── Fuse_In ── Fuse_Out ──►── +12V_FUSED
              │                                                                │
             GND                                                               │
                                                                               │
                           ┌───────────────────────────────────────────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  U3 MP1584EN │
                    │  -LF-Z       │
        +12V_FUSED─►│ 7:VIN  8:BST│──── C9 (100nF) ──── pin 1 (SW)
                    │              │
        +12V_FUSED─►│ 2:EN   1:SW │──►── L1 (33µH) ──►── +5V
                    │              │           │
                    │ 6:FREQ       │         C8 (22µF)
                    │  │           │           │
                    │  └─ VIN      │          GND
                    │ (1.5MHz)     │
                    │              │
                    │ 3:COMP 5:GND│──── GND
                    │  │      │    │
                    │ C_COMP  │  EPAD│──── GND
                    │ (10nF)  │   │  │
                    └──┼──────┼───┼──┘
                       │      │   │
                      GND    GND GND
                    │
                    │ 4:FB
                    │  │
                    └──┘
                       │
                  ┌────┴────┐
                  │         │
               R_FB1     R_FB2
              (100kΩ)   (19.1kΩ)
                  │         │
                 +5V       GND

    D2 (SS34) cathode to pin 1 (SW), anode to GND (freewheeling diode)

    C7 (22µF/25V) between +12V_FUSED and GND (input decoupling, close to VIN pin)
    C8 (22µF/10V) between +5V and GND (output decoupling)
    C11 (10µF) between +5V and GND (additional bulk)
```

**MP1584EN-LF-Z Pin Connections (8-pin SOIC + Exposed Pad):**

| MP1584EN Pin | Name | Net | Connection |
|---|---|---|---|
| 1 | SW | SW_NODE | Switch output → L1 input, D2 cathode, C9 other end |
| 2 | EN | +12V_FUSED | Tied to VIN (always enabled). For soft-start, add RC to VIN. |
| 3 | COMP | COMP_NODE | Compensation pin → C_COMP (10nF) to GND |
| 4 | FB | FB_NODE | Feedback → junction of R_FB1 (100kΩ to +5V) and R_FB2 (19.1kΩ to GND) |
| 5 | GND | GND | Ground plane |
| 6 | FREQ | +12V_FUSED | Tied to VIN for 1.5MHz switching frequency (allows smaller inductor). Tie to GND for ~500kHz. |
| 7 | VIN | +12V_FUSED | Input voltage from fuse → C7 (22µF/25V) to GND nearby |
| 8 | BST | BST_NODE | Bootstrap → C9 (100nF) between BST (pin 8) and SW (pin 1) |
| EP | EPAD | GND | Exposed thermal pad → GND with thermal vias underneath |

> **Alternative (Pre-Built Module):** If using a pre-built MP1584 buck module
> instead, skip U3/D2/L1/C5/C7/C8/C9/C_COMP/R_FB1/R_FB2 and instead place a
> 4-pin header (IN+, IN−, OUT+, OUT−). See Section 3.2.

### 2.3 Display Power Output (HU-043WISBUAA1-B)

The DigiWise HU-043WISBUAA1-B display requires a 5V supply (4.5V–5.5V) at
up to 450mA (typ. 400mA). The display connects via HDMI for video and USB
for capacitive touch. Power is provided through its CN2 connector (2-pin,
2.0mm pitch wafer).

The main PCB supplies display power from the +5V rail through a dedicated
header (J7 pins 1–2), keeping the display powered from the same buck converter as the
Pi. The display's touch interface passes through J7 pins 3–4 as USB D+/D− data
back to the Pi.

```
SCHEMATIC: Display Power & Touch Connection
=============================================

    +5V Rail ─────────────────────────────► J7 Pin 2 (+5V) ──► Display CN2 Pin 1 (5.0V)
    GND ──────────────────────────────────► J7 Pin 1 (GND) ──► Display CN2 Pin 2 (GND)

    Display CN4 (Touch Panel USB) — via J7 pins 3–4:
    Directly routed from Pi USB port OR via J7 passthrough (pins 3–4):

    J7 Pin 1: GND                 ──► Display CN4 Pin 1 (GND-EARTH)
    J7 Pin 2: +5V                 ──► Display CN4 Pin 2 (VDD_5V)
    J7 Pin 3: USB_D+ (DISP_USB_DP) ──► Display CN4 Pin 4 (D+)  ──► Pi USB D+
    J7 Pin 4: USB_D- (DISP_USB_DM) ──► Display CN4 Pin 5 (D-)  ──► Pi USB D-

    NOTE: Display CN4 pin 3 is GND (power ground), tie to GND.

    Backlight control (CN3) is optional:
    - Pin 1: GND
    - Pin 2: PWM (internal pullup to 3.3V, leave floating for full brightness)
    - Pin 3: NC
```

**Display Power Budget:**

| Parameter | Value |
|---|---|
| Supply Voltage | 5.0V (4.5–5.5V range) |
| Typical Supply Current | 400 mA |
| Maximum Supply Current | 450 mA |
| PWM Dimming Frequency | 200 Hz – 200 kHz |
| PWM High Voltage | 2V – 5V |

**Important Notes:**
- The display's HDMI input (CN1) connects directly to the Pi's HDMI port via a short HDMI cable — this is not routed through the PCB.
- The display's USB touch (CN4) can connect directly to a Pi USB port via cable, OR you can route USB D+/D- through J7 pins 3–4 on the main PCB for a cleaner harness. For simplicity, a direct USB cable from Pi to display is recommended.
- J7 pins 1–2 provide a clean 5V power tap specifically for the display, avoiding voltage drop from daisy-chaining through the Pi.

### 2.4 CAN Bus Section

```
SCHEMATIC: MCP2515 + SN65HVD230 CAN Interface
===============================================

                         +3V3
                          │
                         R4 (10kΩ)        +3V3
                          │                 │
                    ┌─────┴────────────────┼──────────────────────┐
                    │ 18:VDD    17:~RESET  │                      │
                    │                      C3 (100nF)             │
                    │ MCP2515-I/SO (U1)    │                      │
                    │                     GND                     │
                    │                                             │
      GPIO8 (CE0)──│ 16:~CS                                      │
                    │                                             │
     GPIO11 (SCLK)─│ 13:SCK            1:TXCAN ─────────┐        │
                    │                                    │        │
     GPIO10 (MOSI)─│ 14:SI             2:RXCAN ────┐    │        │
                    │                               │    │        │
      GPIO9 (MISO)─│ 15:SO                         │    │        │
                    │                               │    │        │
       GPIO25 (INT)│ 12:~INT                        │    │        │
                    │                               │    │        │
                    │ 6:OSC2 ───┐                   │    │        │
                    │           │                   │    │        │
                    │ 7:OSC1 ─┐ │                   │    │        │
                    │         │ │                   │    │        │
                    │ 8:VSS   │ │                   │    │        │
                    │  │      │ │                   │    │        │
                    └──┼──────┼─┼───────────────────┼────┼────────┘
                       │      │ │                   │    │
                      GND     │ │                   │    │
                              │ │                   │    │
                           ┌──┘ └──┐                │    │
                           │  Y1   │ (8 MHz)        │    │
                           │Crystal│                │    │
                           └──┬─┬──┘                │    │
                            C1│ │C2 (22pF each)     │    │
                              │ │                   │    │
                             GND GND                │    │
                                                    │    │
              ┌─────────────────────────────────────┘    │
              │                                          │
              │         ┌────────────────────────────────┘
              │         │
              │         │          +3V3
              │         │           │
              │         │     ┌─────┴──────┐
              │         │     │ 3:VCC      │
              │         └────►│ 1:D   7:CANH│────► J4 Pin 1 (CAN_H)
              │               │            │
              └──────────────►│ 4:R   6:CANL│────► J4 Pin 2 (CAN_L)
                              │            │
                              │ 8:Rs  2:GND│      J4 Pin 3 (GND)
                              │  │     │   │
                              │  │     │   │    C4 (100nF) between
                              └──┼─────┼───┘    VCC and GND
                                 │     │
                                GND   GND

              SN65HVD230 (U2)
              Pin 5 (Vref): Leave floating or 100nF to GND
              Pin 8 (Rs): Tie to GND for high-speed mode
```

**CAN Termination (120Ω with Jumper):**

```
    J4 Pin 1 (CAN_H) ──── R3 (120Ω) ──── J4 Pin 2 (CAN_L)
                      │                │
                      └── JP1 pin 1    JP1 pin 2 ──┘
                          (place jumper to enable termination)
```

**MCP2515 Complete Pin Table:**

| MCP2515 Pin | Name | Net | Connection |
|---|---|---|---|
| 1 | TXCAN | TXCAN | U2 pin 1 (D) |
| 2 | RXCAN | RXCAN | U2 pin 4 (R) |
| 3 | CLKOUT | — | No connect (or use for debug) |
| 4 | ~TX0BF | — | No connect (or wire to LED2 for CAN activity, see §2.6) |
| 5 | ~TX1BF | — | No connect |
| 6 | OSC2 | XTAL2 | Y1 pin 2 + C2 (22pF→GND) |
| 7 | OSC1 | XTAL1 | Y1 pin 1 + C1 (22pF→GND) |
| 8 | VSS | GND | Ground |
| 9 | ~TX2RTS | +3V3 | Tie high (inactive, has internal pullup) |
| 10 | ~TX1RTS | +3V3 | Tie high (inactive, has internal pullup) |
| 11 | ~TX0RTS | +3V3 | Tie high (inactive, has internal pullup) |
| 12 | ~INT | MCP_INT | Pi GPIO25 (J1 pin 22) |
| 13 | SCK | SPI_SCK | Pi GPIO11 (J1 pin 23) |
| 14 | SI | SPI_MOSI | Pi GPIO10 (J1 pin 19) |
| 15 | SO | SPI_MISO | Pi GPIO9 (J1 pin 21) |
| 16 | ~CS | SPI_CE0 | Pi GPIO8 (J1 pin 24) |
| 17 | ~RESET | MCP_RST | R4 (10kΩ) pullup to +3V3 |
| 18 | VDD | +3V3 | 3.3V rail + C3 (100nF→GND) |

**SN65HVD230 Complete Pin Table:**

| SN65HVD230 Pin | Name | Net | Connection |
|---|---|---|---|
| 1 | D | TXCAN | MCP2515 pin 1 |
| 2 | GND | GND | Ground |
| 3 | VCC | +3V3 | 3.3V rail + C4 (100nF→GND) |
| 4 | R | RXCAN | MCP2515 pin 2 |
| 5 | Vref | — | Float or 100nF to GND |
| 6 | CANL | CAN_L | J4 pin 2, R3 (term) |
| 7 | CANH | CAN_H | J4 pin 1, R3 (term) |
| 8 | Rs | GND | Ground (high-speed mode) |

### 2.5 GPS Module Section (SparkFun NEO-M9N, Chip Antenna — GPS-15733)

The SparkFun NEO-M9N GPS Breakout with chip antenna is mounted directly onto
the main PCB using female pin headers. The breakout board has 0.1"-spaced PTH
pins along its edges. We connect via UART for NMEA data and provide 3.3V power
from the main PCB's regulated rail.

**Key Specifications:**
- **Module:** u-blox NEO-M9N, 92-channel M9 engine
- **Constellations:** GPS, GLONASS, Galileo, BeiDou (4 concurrent)
- **Accuracy:** ~1.5m horizontal
- **Protocols:** NMEA, UBX, RTCM over UART or I2C
- **Supply Voltage:** 3.3V
- **Cold start TTFF:** ~24s | **Hot start:** ~2s (onboard rechargeable backup battery)
- **Board dimensions:** 33mm × 40.6mm (1.30" × 1.60")
- **I2C Address:** 0x42 (default)
- **Interfaces:** UART (TX/RX), I2C (SDA/SCL), SPI (requires jumper)
- **Anti-jamming:** Integrated SAW filter + LNA; detects jamming/spoofing

```
SCHEMATIC: GPS Module Connection
=================================

    SparkFun NEO-M9N Breakout (mounted on main PCB via pin headers)

    Main PCB provides these connections through J3 header:

         J3 Female Header (on main PCB, breakout plugs in)

         Pin mapping (matching SparkFun board PTH edge):
         ┌──────────────────────────────────────────┐
         │  GND  3V3  TX  RX  SDA  SCL              │
         │   │    │    │    │   │    │               │
         └───┼────┼────┼────┼───┼────┼───────────────┘
             │    │    │    │   │    │
            GND +3V3   │    │   │    │
                       │    │   │    │
                       │    │  (I2C available but unused in
                       │    │   this design — reserved for
                       │    │   future expansion)
                       │    │
                       │    └──── Pi GPIO14 / TXD (J1 pin 8)
                       │          (Pi TX → GPS RX)
                       │
                       └───────── Pi GPIO15 / RXD (J1 pin 10)
                                  (GPS TX → Pi RX)
```

**Connections used:**

| GPS Breakout Pin | Main PCB Net | RPi GPIO (J1 pin) | Direction |
|---|---|---|---|
| GND | GND | Pin 6, 9, 14, etc. | — |
| 3V3 | +3V3 | Pin 1 (3.3V) | Power to GPS |
| TX | GPS_TX | GPIO15 / RXD (pin 10) | GPS → Pi |
| RX | GPS_RX | GPIO14 / TXD (pin 8) | Pi → GPS |
| SDA | — | (GPIO2 / pin 3, reserved) | Not connected (I2C reserved) |
| SCL | — | (GPIO3 / pin 5, reserved) | Not connected (I2C reserved) |

> **Important:** The NEO-M9N operates at 3.3V logic levels natively — no level
> shifting is needed for the Raspberry Pi's 3.3V GPIO.

> **Note on I2C:** The NEO-M9N supports I2C at address 0x42 in addition to
> UART. The J3 header breaks out SDA/SCL for future use (e.g., if you want to
> free up UART for another peripheral). For this design, UART is used since
> it's the most straightforward path for NMEA data.

**Physical Mounting:** Create a rectangular keep-out zone (35mm × 43mm) on the
main PCB with mounting holes matching the SparkFun board's corner holes. The
female pin header solders to the main PCB; the breakout board's male pins plug
into it.

### 2.6 Status LEDs (On-Board)

**On-board LEDs (Main PCB):**
- LED1 (Green): Power indicator — connected directly from +3V3 through R1 (330Ω) to GND. Always on when 3.3V rail is active.
- LED2 (Blue): CAN activity — wired to MCP2515 pin 4 (~TX0BF) which can be configured via the BFPCTRL register to output a signal when a CAN frame is received.

```
    +3V3
     │
    R1 (330Ω)
     │
    LED1 (Green) Anode
     │
    LED1 Cathode
     │
    GND

    MCP2515 Pin 4 (~TX0BF)    (Configure via BFPCTRL register for RX activity)
     │
    R2 (330Ω)
     │
    LED2 (Blue) Cathode
     │
    LED2 Anode
     │
    +3V3
```

### 2.7 Daughter Board Connector

```
SCHEMATIC: 8-Pin Molex Micro-Fit 3.0 (J2)
============================================

    J2 Molex Micro-Fit 3.0  (2×4, keyed connector)

    Pin 1: LED_DB1   → Pi GPIO23 (J1 pin 16)   Green LED on daughter board
    Pin 2: LED_DB2   → Pi GPIO24 (J1 pin 18)   Blue LED on daughter board
    Pin 3: LED_DB3   → Pi GPIO5  (J1 pin 29)   Red LED on daughter board
    Pin 4: BTN1      → Pi GPIO17 (J1 pin 11)   Screen button
    Pin 5: BTN2      → Pi GPIO27 (J1 pin 13)   Record button
    Pin 6: BTN3      → Pi GPIO22 (J1 pin 15)   Up button
    Pin 7: +3V3      → +3V3 rail
    Pin 8: GND       → GND
```

**Harness (J2 ↔ daughter board J6):** To make the cable between main and daughter board, order: **Molex 43025-0800** (2×4 receptacle housing, DigiKey WM2014-ND) and **Molex 43030-0007** (female crimp terminals, DigiKey WM1142CT-ND; need 8 per cable). Crimp 8 wires into the receptacle; plug into J2 (main) and J6 (daughter).

### 2.8 Raspberry Pi 40-Pin Header (J1) — Complete Pin Assignment

| J1 Phys Pin | RPi GPIO | Net Name | Connects To |
|---|---|---|---|
| 1 | 3.3V | +3V3 | Power rail, GPS 3V3, Daughter board 3V3 |
| 2 | 5V | +5V | Buck converter output (powers Pi) |
| 3 | GPIO2 (SDA1) | — | Reserved (I2C, for future GPS I2C) |
| 4 | 5V | +5V | Buck converter output |
| 5 | GPIO3 (SCL1) | — | Reserved (I2C, for future GPS I2C) |
| 6 | GND | GND | Ground |
| 7 | GPIO4 | — | Unused |
| 8 | GPIO14 (TXD) | GPS_RX | NEO-M9N RX (Pi transmits to GPS) |
| 9 | GND | GND | Ground |
| 10 | GPIO15 (RXD) | GPS_TX | NEO-M9N TX (GPS transmits to Pi) |
| 11 | GPIO17 | BTN1 | J2 pin 4 → Daughter board Button 1 |
| 12 | GPIO18 | — | Unused |
| 13 | GPIO27 | BTN2 | J2 pin 5 → Daughter board Button 2 |
| 14 | GND | GND | Ground |
| 15 | GPIO22 | BTN3 | J2 pin 6 → Daughter board Button 3 |
| 16 | GPIO23 | LED_DB1 | J2 pin 1 → Daughter board LED1 (green) |
| 17 | 3.3V | +3V3 | Power rail |
| 18 | GPIO24 | LED_DB2 | J2 pin 2 → Daughter board LED2 (blue) |
| 19 | GPIO10 (MOSI) | SPI_MOSI | U1 MCP2515 pin 14 (SI) |
| 20 | GND | GND | Ground |
| 21 | GPIO9 (MISO) | SPI_MISO | U1 MCP2515 pin 15 (SO) |
| 22 | GPIO25 | MCP_INT | U1 MCP2515 pin 12 (~INT) |
| 23 | GPIO11 (SCLK) | SPI_SCK | U1 MCP2515 pin 13 (SCK) |
| 24 | GPIO8 (CE0) | SPI_CE0 | U1 MCP2515 pin 16 (~CS) |
| 25 | GND | GND | Ground |
| 26 | GPIO7 (CE1) | — | Unused |
| 27 | GPIO0 (ID_SD) | — | Reserved (HAT EEPROM, do not use) |
| 28 | GPIO1 (ID_SC) | — | Reserved (HAT EEPROM, do not use) |
| 29 | GPIO5 | LED_DB3 | J2 pin 3 → Daughter board LED3 (red) |
| 30 | GND | GND | Ground |
| 31 | GPIO6 | — | Unused |
| 32 | GPIO12 | — | Unused |
| 33 | GPIO13 | — | Unused |
| 34 | GND | GND | Ground |
| 35 | GPIO19 | — | Unused |
| 36 | GPIO16 | — | Unused |
| 37 | GPIO26 | — | Unused |
| 38 | GPIO20 | — | Unused |
| 39 | GND | GND | Ground |
| 40 | GPIO21 | — | Unused |

### 2.9 Complete Main PCB Net List

| Net Name | Nodes (Ref:Pin) |
|---|---|
| +12V_IN | J5:1, D1:Anode |
| +12V_FUSED | D1:Cathode, F1:1 |
| +12V_POST_FUSE | F1:2, U3:7(VIN), U3:2(EN), U3:6(FREQ), C7:+, C5:1 |
| SW_NODE | U3:1(SW), L1:1, D2:Cathode, C9:1 |
| BST_NODE | U3:8(BST), C9:2 |
| COMP_NODE | U3:3(COMP), C_COMP:1 |
| FB_NODE | U3:4(FB), R_FB1:2, R_FB2:1 |
| +5V | L1:2, C8:+, C11:+, R_FB1:1, J1:2, J1:4, J7:2 |
| +3V3 | J1:1, J1:17, U1:18, U2:3, U1:9, U1:10, U1:11, R4:1, C3:1, C4:1, C10:+, J2:7, J3:3V3, LED1_R1 junction, LED2 anode |
| GND | J5:2, C5:2, C7:−, C8:−, C10:−, C11:−, C_COMP:2, U3:5(GND), U3:EP, U1:8, U2:2, U2:8, C1:2, C2:2, C3:2, C4:2, J1:6,9,14,20,25,30,34,39, J2:8, J3:GND, J4:3, J7:1, LED1:Cathode, R_FB2:2 |
| XTAL1 | U1:7, Y1:1, C1:1 |
| XTAL2 | U1:6, Y1:2, C2:1 |
| TXCAN | U1:1, U2:1 |
| RXCAN | U1:2, U2:4 |
| CAN_H | U2:7, J4:1, R3:1 (through JP1) |
| CAN_L | U2:6, J4:2, R3:2 (through JP1) |
| MCP_RST | U1:17, R4:2 |
| MCP_INT | U1:12, J1:22 (GPIO25) |
| SPI_SCK | U1:13, J1:23 (GPIO11) |
| SPI_MOSI | U1:14, J1:19 (GPIO10) |
| SPI_MISO | U1:15, J1:21 (GPIO9) |
| SPI_CE0 | U1:16, J1:24 (GPIO8) |
| GPS_TX | J3:TX, J1:10 (GPIO15/RXD) |
| GPS_RX | J3:RX, J1:8 (GPIO14/TXD) |
| BTN1 | J2:4, J1:11 (GPIO17) |
| BTN2 | J2:5, J1:13 (GPIO27) |
| BTN3 | J2:6, J1:15 (GPIO22) |
| LED_DB1 | J2:1, J1:16 (GPIO23) |
| LED_DB2 | J2:2, J1:18 (GPIO24) |
| LED_DB3 | J2:3, J1:29 (GPIO5) |
| DISP_USB_DP | J7:3 (USB D+ passthrough to Pi USB) |
| DISP_USB_DM | J7:4 (USB D- passthrough to Pi USB) |

---

## 3. Custom Symbol & Footprint Creation

### 3.1 MP1584EN-LF-Z Symbol & Footprint (from Ultra Librarian)

The MP1584EN is **not** in KiCad 9.0's default libraries. Download the
complete symbol + footprint + 3D model from Ultra Librarian:

1. Go to [Ultra Librarian](https://www.ultralibrarian.com/) and search for
   **MP1584EN-LF-Z** (Monolithic Power Systems).
2. Select the part — you will see:
   - **Symbol:** `MP1584EN-LF_1` (8 pins + EPAD)
   - **Footprint:** `SOIC8E_MPS`
   - **3D Model:** SOIC-8 with exposed pad
3. Click **Download Now** → Select **KiCad** as the EDA tool → Download the ZIP.
4. Extract the ZIP. You will get:
   - A `.kicad_sym` file (symbol library)
   - A `.kicad_mod` file (footprint)
   - A `.step` or `.wrl` file (3D model)
5. **Import into your KiCad project:**
   - **Symbol:** Schematic Editor → **Preferences → Manage Symbol Libraries** →
     **Project Libraries** tab → click **+** (Add existing) → browse to the
     downloaded `.kicad_sym` file → OK.
   - **Footprint:** PCB Editor → **Preferences → Manage Footprint Libraries** →
     **Project Libraries** tab → click **+** → browse to the folder containing
     the `.kicad_mod` file → OK.
6. In the schematic, press **A** (Add Symbol) → search for `MP1584EN` → place it.

**Alternative:** If you prefer not to use Ultra Librarian, assign the KiCad standard
footprint `Package_SO:SOIC-8-1EP_3.9x4.9mm_P1.27mm_EP2.29x3mm` in the Assign
Footprints tool. The pinout is identical.

**Ultra Librarian Pin Map (matches datasheet):**

```
          ┌───────────┐
   SW  ──┤ 1       8 ├── BST
   EN  ──┤ 2       7 ├── VIN
  COMP ──┤ 3       6 ├── FREQ
   FB  ──┤ 4       5 ├── GND
          └─────┬─────┘
                │ EPAD (GND)
```

| Pin | Name | KiCad Pin Type |
|-----|------|----------------|
| 1 | SW | Passive |
| 2 | EN | Input |
| 3 | COMP | Passive |
| 4 | FB | Input |
| 5 | GND | Power Input |
| 6 | FREQ | Input |
| 7 | VIN | Power Input |
| 8 | BST | Passive |
| EP | EPAD | Power Input |

### 3.2 Alternative: Pre-Built MP1584 Buck Module Footprint

If using a pre-built module (the common blue/red boards on Amazon), create
a simple 4-pin through-hole footprint:

1. **Footprint Editor → File → New Footprint** in your project library
2. Name: `MP1584_Module_4pin`
3. Add 4 through-hole pads in a 2×2 grid, 2.54mm pitch:
   - Pad 1: IN+ (top-left)
   - Pad 2: IN− (top-right)
   - Pad 3: OUT+ (bottom-left)
   - Pad 4: OUT− (bottom-right)
4. Actual pad spacing varies by module — **measure your specific module**
   and adjust. Typical: ~17mm × 22mm board with 4 corner holes.
5. Add a courtyard rectangle matching the module dimensions
6. Add mounting hole pads if the module has them

In the schematic, use `Connector_Generic:Conn_01x04` as the symbol and assign
this custom footprint.

### 3.3 Custom Footprint: SparkFun NEO-M9N Breakout Mount

The SparkFun NEO-M9N GPS Breakout (chip antenna version, GPS-15733) is
33mm × 40.6mm with PTH pins along one edge. Create a footprint to mount it:

**Option A — 6-Pin Connection (Recommended):**

Use a standard 1×6 female pin header. Solder male headers to the GPS board's
GND, 3V3, TX, RX, SDA, SCL pins. Plug into the female header on the main PCB.
Add mounting holes for mechanical support.

1. **Footprint Editor → New Footprint** → Name: `SparkFun_NEO-M9N_Mount`
2. Place a 1×6 THT pad row at 2.54mm pitch for the signal connection
3. Add 4 NPTH (Non-Plated Through Hole) mounting holes at the four corners:
   - Hole diameter: 2.5mm (for M2 screws)
   - Rectangle: ~28mm × 36mm (measure from SparkFun board drawing)
4. Draw courtyard: 35mm × 43mm rectangle
5. Draw silkscreen outline showing board placement area
6. In the schematic, use symbol `Connector_Generic:Conn_01x06` and label pins:
   - Pin 1: GND
   - Pin 2: 3V3
   - Pin 3: TX (GPS transmits)
   - Pin 4: RX (GPS receives)
   - Pin 5: SDA (reserved)
   - Pin 6: SCL (reserved)

**Schematic for J3:**

| J3 Pin | Net | Connects To |
|---|---|---|
| 1 | GND | Ground plane |
| 2 | +3V3 | 3.3V power rail |
| 3 | GPS_TX | Pi GPIO15/RXD (J1 pin 10) |
| 4 | GPS_RX | Pi GPIO14/TXD (J1 pin 8) |
| 5 | — | Reserved (SDA, future I2C) |
| 6 | — | Reserved (SCL, future I2C) |

---

## 4. Step-by-Step: Building the Schematic in KiCad 9.0

### 4.1 Create the Project

1. Open **KiCad 9.0**
2. **File → New Project**
3. Name: `FSAE_Display_MainPCB`
4. Choose your project directory
5. KiCad creates `.kicad_pro`, `.kicad_sch`, and `.kicad_pcb` files

### 4.2 Open the Schematic Editor

1. Double-click the `.kicad_sch` file (or click the schematic icon in the project manager)
2. The **Schematic Editor (Eeschema)** opens with a blank sheet

### 4.3 Set Up the Sheet

1. **File → Page Settings** → Set paper size to A3 (gives more room)
2. Fill in the title block: Project name, date, revision, your name

### 4.4 Add Power Symbols

1. Press **P** (or **Place → Power Port**) to add power symbols
2. Search for and place these power symbols (one of each, in a clear area):
   - `+12V`
   - `+5V`
   - `+3V3`
   - `GND`
3. These are global power nets — every symbol with the same name connects automatically

### 4.5 Place Components — Power Supply Section

1. Press **A** (Add Symbol) → search `D_Schottky` → place **D1** (reverse polarity diode)
2. Press **A** → search `Fuse` → place **F1**
3. Press **A** → search `MP1584EN` (from your imported Ultra Librarian library) → place **U3**
4. Press **A** → search `D_Schottky` → place **D2** (freewheeling diode)
5. Press **A** → search `L` → place **L1** (inductor)
6. Press **A** → search `Screw_Terminal_01x02` → place **J5** (power input)
7. Add resistors (**R**) for R_FB1, R_FB2
8. Add capacitors (**C**) for C5, C7, C8, C9, C11, C_COMP

**Wiring the power section (MP1584EN-LF-Z 8-pin + EPAD):**

9. Press **W** to start wiring
10. Connect in this order:
    - J5 pin 1 → D1 Anode
    - D1 Cathode → F1 pin 1
    - F1 pin 2 → U3 pin 7 (VIN), U3 pin 2 (EN), U3 pin 6 (FREQ), C7 positive, C5 pin 1
    - C5 pin 2 → GND (VIN bypass — place close to pins 5 & 7)
    - U3 pin 1 (SW) → L1 pin 1, D2 Cathode, C9 pin 1
    - U3 pin 8 (BST) → C9 pin 2
    - U3 pin 3 (COMP) → C_COMP pin 1; C_COMP pin 2 → GND
    - L1 pin 2 → +5V net, C8 positive, R_FB1 pin 1
    - U3 pin 4 (FB) → R_FB1 pin 2, R_FB2 pin 1
    - R_FB2 pin 2 → GND
    - D2 Anode → GND
    - C7 negative, C8 negative → GND
    - U3 pin 5 (GND), U3 EPAD → GND

### 4.6 Place Components — Display Power + USB Output

1. Press **A** → search `Conn_01x04` → place **J7** (display power + USB header)
2. Wire J7 pin 1 to `GND`, pin 2 to `+5V`
3. Wire J7 pin 3 to net `DISP_USB_DP` (USB D+), pin 4 to net `DISP_USB_DM` (USB D−)
4. Place net labels `DISP_USB_DP` and `DISP_USB_DM` on the wires to pins 3 and 4

### 4.7 Place Components — CAN Bus Section

1. Press **A** → search `MCP2515` → select `MCP2515-xSO` → place **U1**
2. Press **A** → search `SN65HVD230` → place **U2**
3. Press **A** → search `Crystal` → place **Y1**
4. Press **A** → search `Screw_Terminal_01x02` → place **J4** (CAN terminal, 2-pos: CAN_H, CAN_L)
5. Add **R3** (120Ω), **R4** (10kΩ)
6. Add **JP1** (2-pin jumper header: `Conn_01x02`)
7. Add **C1, C2** (22pF), **C3, C4** (100nF)

**Wiring the CAN section:**

8. Wire U1 per the pin table in §2.4:
   - U1:1 (TXCAN) → U2:1 (D)
   - U1:2 (RXCAN) → U2:4 (R)
   - U1:6 (OSC2) → Y1 pin 2, C2 pin 1
   - U1:7 (OSC1) → Y1 pin 1, C1 pin 1
   - C1 pin 2, C2 pin 2 → GND
   - U1:8 (VSS) → GND
   - U1:17 (~RESET) → R4 pin 2; R4 pin 1 → +3V3
   - U1:18 (VDD) → +3V3; C3 between VDD and GND
   - U1:9 (~TX2RTS), U1:10 (~TX1RTS), U1:11 (~TX0RTS) → +3V3 (tie high, inactive)
9. Wire U2 per §2.4:
   - U2:3 (VCC) → +3V3; C4 between VCC and GND
   - U2:7 (CANH) → J4:1, one side of R3
   - U2:6 (CANL) → J4:2, other side of R3
   - R3 connects through JP1 (jumper enables/disables termination)
   - U2:8 (Rs) → GND
   - U2:2 → GND
   - J4:3 → GND

### 4.8 Place the Raspberry Pi Header

1. Press **A** → search `Conn_02x20_Odd_Even` → place **J1**
2. This is a large symbol. Position it centrally on the schematic.
3. Use **net labels** (press **L**) to connect GPIOs to their destinations:
   - Label J1 pin 19 as `SPI_MOSI`, label U1 pin 14 as `SPI_MOSI` → they connect
   - Label J1 pin 21 as `SPI_MISO`, label U1 pin 15 as `SPI_MISO`
   - Label J1 pin 23 as `SPI_SCK`, label U1 pin 13 as `SPI_SCK`
   - Label J1 pin 24 as `SPI_CE0`, label U1 pin 16 as `SPI_CE0`
   - Label J1 pin 22 as `MCP_INT`, label U1 pin 12 as `MCP_INT`

4. Power connections:
   - J1 pin 2, pin 4 → `+5V` power symbol
   - J1 pin 1, pin 17 → `+3V3` power symbol
   - J1 pins 6, 9, 14, 20, 25, 30, 34, 39 → `GND` power symbol

5. GPS UART labels:
   - J1 pin 8 (GPIO14/TXD) → label `GPS_RX`
   - J1 pin 10 (GPIO15/RXD) → label `GPS_TX`

6. Daughter board labels:
   - J1 pin 11 (GPIO17) → label `BTN1`
   - J1 pin 13 (GPIO27) → label `BTN2`
   - J1 pin 15 (GPIO22) → label `BTN3`
   - J1 pin 16 (GPIO23) → label `LED_DB1`
   - J1 pin 18 (GPIO24) → label `LED_DB2`
   - J1 pin 29 (GPIO5) → label `LED_DB3`

### 4.9 Place GPS & Daughter Board Connectors

1. Press **A** → search `Conn_01x06` → place **J3** (GPS header)
   - Label pin 1: GND, pin 2: +3V3, pin 3: `GPS_TX`, pin 4: `GPS_RX`, pin 5: SDA (NC), pin 6: SCL (NC)

2. Press **A** → search `Conn_02x04_Odd_Even` → place **J2** (daughter board)
   - Label pin 1: `LED_DB1`, pin 2: `LED_DB2`
   - Label pin 3: `LED_DB3`, pin 4: `BTN1`
   - Label pin 5: `BTN2`, pin 6: `BTN3`
   - Label pin 7: `+3V3`, pin 8: `GND`

### 4.10 Place On-Board LEDs

1. Place **LED1** (Device:LED) and **R1** (330Ω) for power indicator
2. Wire: +3V3 → R1 → LED1 Anode → LED1 Cathode → GND
3. Place **LED2** (Device:LED) and **R2** (330Ω) for CAN status
4. Wire: +3V3 → LED2 Anode → LED2 Cathode → R2 → U1 pin 4 (~TX0BF)
   (Active-low: configure via BFPCTRL register for RX buffer full indication)

### 4.11 Add Decoupling Capacitors

Decoupling capacitors suppress high-frequency noise on power rails and prevent
voltage sag during transient current draw. Every IC needs its own local
decoupling cap — placed as close to the IC's VDD/VCC pin as physically
possible (ideally within 3–5mm), with a short, direct return path to GND.

**Why this matters:** Without decoupling, the MCP2515's SPI lines can
glitch, the SN65HVD230 can produce corrupted CAN frames, and the buck
converter can oscillate. This is especially critical in a race car with
noisy 12V power and electromagnetic interference from the motor controller.

**Step-by-step placement in the schematic:**

1. Press **A** → search `C` → place a capacitor symbol near each IC. Edit
   the value field (press **E** on the symbol) to set it to `100nF`.

2. Wire each cap between the IC's power pin and a `GND` power symbol:

```
DECOUPLING CAPACITOR PLACEMENT MAP
====================================

Each cap is shown with its IC and the exact pin it decouples.
In the schematic, place the cap symbol directly adjacent to the IC
symbol it serves. In the PCB layout, place it within 3–5mm of the pin.

    ┌─────────────── U1 (MCP2515) ───────────────┐
    │                                              │
    │   Pin 18 (VDD) ─────┬───── +3V3             │
    │                      │                       │
    │                 C3 (100nF)    ◄── Place here │
    │                      │                       │
    │                     GND                      │
    └──────────────────────────────────────────────┘

    ┌─────────────── U2 (SN65HVD230) ─────────────┐
    │                                               │
    │   Pin 3 (VCC) ──────┬───── +3V3              │
    │                      │                        │
    │                 C4 (100nF)    ◄── Place here  │
    │                      │                        │
    │                     GND                       │
    └───────────────────────────────────────────────┘

    ┌─────────────── U3 (MP1584EN) ────────────────┐
    │                                               │
    │   Pin 8 (VCC) ──────┬───── (internal LDO)    │
    │                      │                        │
    │                 C5 (100nF)    ◄── Place here  │
    │                      │                        │
    │                     GND                       │
    │                                               │
    │   (Skip C5 if using pre-built buck module)    │
    └───────────────────────────────────────────────┘

    ┌─────────────── J1 (Pi Header Area) ──────────┐
    │                                               │
    │   +3V3 (from J1 pin 1/17) ──┬─────           │
    │                              │                │
    │                        C6 (100nF)  ◄── Place  │
    │                              │      near J1   │
    │                             GND               │
    │                                               │
    │   This "general" decoupling cap filters the   │
    │   3.3V rail at the point where it enters the  │
    │   shield PCB from the Pi's onboard regulator. │
    └───────────────────────────────────────────────┘
```

3. Add bulk decoupling capacitors. These are larger-value caps that handle
   slower, larger current transients. Place them near the power entry
   points of each rail, not necessarily next to any specific IC:

```
BULK DECOUPLING PLACEMENT
===========================

    +3V3 Rail Entry (near J1 pin 1)
         │
    ┌────┴────┐
    │         │
    │   C10 (10µF / 16V)    ◄── Ceramic MLCC, 0805
    │         │                   Handles 3.3V rail sag during
    │        GND                  GPS startup, MCP2515 SPI bursts
    │
    │
    +5V Rail (buck converter output, near L1 output)
         │
    ┌────┴────┐
    │         │
    │   C11 (10µF / 16V)    ◄── Ceramic MLCC, 0805
    │         │                   Handles 5V rail transients from
    │        GND                  Pi + display current draw
    │
    │   (Note: C8, the 22µF output cap, is part of the buck
    │    converter circuit and is separate from C11. Both are
    │    needed — C8 for converter stability, C11 for additional
    │    bulk filtering.)
```

**Summary of all decoupling capacitors:**

| Ref | Value | Decouples | IC Pin | Type |
|-----|-------|-----------|--------|------|
| C3 | 100nF | U1 MCP2515 VDD | Pin 18 | High-freq bypass |
| C4 | 100nF | U2 SN65HVD230 VCC | Pin 3 | High-freq bypass |
| C5 | 100nF | U3 MP1584EN VIN bypass | Near pin 7 (VIN) | High-freq bypass (place close to VIN + GND pins) |
| C6 | 100nF | +3V3 rail at J1 | Near J1 pin 1 | High-freq bypass |
| C10 | 10µF | +3V3 rail bulk | Near J1 pin 1 | Bulk decoupling |
| C11 | 10µF | +5V rail bulk | Near L1 output | Bulk decoupling |

> **PCB Layout Rule of Thumb:** In the PCB editor, the trace from the IC
> power pin to the cap pad, and from the cap's GND pad to the ground plane
> via, should form the shortest possible loop. A long, meandering trace
> between the IC and its decoupling cap defeats the purpose. Route the cap
> connection first, before other traces, to ensure the shortest path.

### 4.12 Add Power Flags & Annotate

**What are PWR_FLAGs?** KiCad's ERC (Electrical Rules Check) requires that
every power net be "driven" by at least one power source. Connectors like
screw terminals (J5) and pin headers (J1) are classified as *passive* pins,
not *power output* pins — so KiCad doesn't recognize them as sources.
Without PWR_FLAG symbols, you'll get ERC errors like:

    Error: Pin not driven (Net +12V_IN)
    Error: Pin not driven (Net GND)

PWR_FLAG tells KiCad: "Trust me, this net is powered from an external source."

**Where to place PWR_FLAG symbols:**

You need PWR_FLAGs on every power net that enters the board through a
connector (not through an IC power output). For this design, you need
exactly **three** PWR_FLAG symbols.

```
POWER FLAG PLACEMENT DIAGRAM (Main PCB)
=========================================

Place each PWR_FLAG by pressing A → search "PWR_FLAG" → place it,
then wire it directly to the net it validates.


    ┌─── Net: +12V_IN ─────────────────────────────────────────┐
    │                                                           │
    │         J5 Pin 1 (+12V_IN)                                │
    │              │                                            │
    │              ├──── D1 Anode (to rest of power circuit)    │
    │              │                                            │
    │         ┌────┴────┐                                       │
    │         │PWR_FLAG │  ◄── Place here, on the +12V_IN net   │
    │         └─────────┘      at or near J5 pin 1              │
    │                                                           │
    │  WHY: J5 is a screw terminal (passive pin). The 12V       │
    │  car battery is the actual source, but KiCad can't see    │
    │  it. PWR_FLAG tells ERC this net is externally powered.   │
    └───────────────────────────────────────────────────────────┘


    ┌─── Net: +5V ──────────────────────────────────────────────┐
    │                                                            │
    │   If using MP1584EN-LF-Z (U3) from Ultra Librarian: The   │
    │   output is taken from L1 (connected to SW pin 1), not    │
    │   from a dedicated output pin. The +5V net is defined by  │
    │   the inductor output. You will likely need a PWR_FLAG on │
    │   the +5V net (on the L1/C8/R_FB1 junction) to satisfy   │
    │   ERC, since no pin on U3 is typed "Power Output."        │
    │                                                            │
    │   If using a pre-built module (Conn_01x04): The connector │
    │   pins are passive, so you WOULD need a PWR_FLAG on the   │
    │   OUT+ pin's net.                                         │
    │                                                            │
    │         +5V rail (L1 pin 2 junction)                       │
    │              │                                             │
    │         ┌────┴────┐                                        │
    │         │PWR_FLAG │  ◄── Place on +5V net                  │
    │         └─────────┘                                        │
    └────────────────────────────────────────────────────────────┘


    ┌─── Net: +3V3 ─────────────────────────────────────────────┐
    │                                                            │
    │   +3V3 comes from the Pi's onboard regulator, entering    │
    │   our PCB through J1 pin 1 (which is a passive pin).      │
    │                                                            │
    │         J1 Pin 1 (+3V3)                                    │
    │              │                                             │
    │              ├──── (to U1, U2, GPS, daughter board, etc.)  │
    │              │                                             │
    │         ┌────┴────┐                                        │
    │         │PWR_FLAG │  ◄── Place here, on the +3V3 net      │
    │         └─────────┘      at or near J1 pin 1              │
    │                                                            │
    │  WHY: The Pi regulates 5V→3.3V internally, but from       │
    │  KiCad's perspective, J1 is just a connector with          │
    │  passive pins. PWR_FLAG validates this external source.    │
    └────────────────────────────────────────────────────────────┘


    ┌─── Net: GND ──────────────────────────────────────────────┐
    │                                                            │
    │         J5 Pin 2 (GND)                                     │
    │              │                                             │
    │              ├──── (to ground plane, all GND connections)  │
    │              │                                             │
    │         ┌────┴────┐                                        │
    │         │PWR_FLAG │  ◄── Place here, on the GND net       │
    │         └─────────┘      at or near J5 pin 2              │
    │                                                            │
    │  WHY: GND is a power net too. Without this flag, KiCad    │
    │  will warn that the GND net has no driving source.         │
    │  One PWR_FLAG on GND is sufficient for the entire board.  │
    └────────────────────────────────────────────────────────────┘
```

**PWR_FLAG summary for Main PCB:**

| PWR_FLAG # | Attach To Net | Place Near | Reason |
|---|---|---|---|
| 1 | +12V_IN | J5 pin 1 | 12V from car enters via screw terminal (passive) |
| 2 | +3V3 | J1 pin 1 | 3.3V from Pi enters via GPIO header (passive) |
| 3 | GND | J5 pin 2 | Ground reference enters via screw terminal (passive) |

> **Common mistake:** Don't place a PWR_FLAG on every single +3V3 or GND
> symbol on the schematic — you only need **one** per net, anywhere on that
> net. KiCad nets are global, so one flag covers the whole net.

> **Another common mistake:** Placing PWR_FLAG on signal nets (like SPI_MOSI)
> — don't do this. PWR_FLAG is only for power nets that KiCad can't identify
> as being driven by a power source.

**Now annotate and run ERC:**

1. **Tools → Annotate Schematic** → click **Annotate** → assigns reference
   designators (U1, R1, C1, etc.) sequentially. If you've already annotated,
   it will preserve existing designators and only assign new ones.

2. **Inspect → Electrical Rules Check (ERC)** → click **Run ERC**

3. Review the results panel. Common ERC findings and fixes:

   | ERC Message | What It Means | How To Fix |
   |---|---|---|
   | "Pin not driven" on +12V_IN | Missing PWR_FLAG | Add PWR_FLAG on +12V_IN net (see diagram above) |
   | "Pin not driven" on GND | Missing PWR_FLAG | Add PWR_FLAG on GND net |
   | "Pin not driven" on +3V3 | Missing PWR_FLAG | Add PWR_FLAG on +3V3 net |
   | "Unconnected pin" on U1 pin 3 | MCP2515 CLKOUT not wired | Place a No Connect flag: press **Q** → click pin 3 |
   | "Unconnected pin" on U1 pin 5 | MCP2515 ~TX1BF not wired | Place No Connect flag on pin 5 |
   | "Different net names on same wire" | Conflicting labels | Check for duplicate or mismatched net labels |

4. Mark all intentionally unused MCP2515 pins with No Connect flags (press
   **Q** then click each unconnected pin):
   - Pin 3: CLKOUT (unused)
   - Pin 5: ~TX1BF (unused)
   - (Pin 4: ~TX0BF is used for LED2 if wired per §2.6)

### 4.13 Assign Footprints

Every schematic symbol needs a physical footprint assigned before you can
create the PCB layout. In KiCad 9.0, you can assign footprints in two ways:
per-symbol (in the schematic) or all-at-once (via the Footprint Assignment
tool). The all-at-once method is recommended since it gives you a table view
of every component.

**Method: Footprint Assignment Tool (Recommended)**

1. In the Schematic Editor, go to **Tools → Assign Footprints** (or click the
   footprint assignment icon in the top toolbar — it looks like a chip with an
   arrow).

2. The **Footprint Assignment** window opens with three panels:
   - **Left panel:** Footprint library browser (library list)
   - **Center panel:** Your component list (every symbol in the schematic)
   - **Right panel:** Footprints within the selected library

3. For each component in the center panel, you need to assign a footprint.
   Components that already have a default footprint (from the symbol library)
   will show it — verify it's correct.

**Step-by-step assignment walkthrough:**

Work through the center panel component-by-component. For each row, either
confirm the existing footprint or assign a new one. The table below matches
the `fsae_pcb.kicad_sch` schematic component references:

```
FOOTPRINT ASSIGNMENT WALKTHROUGH (matches fsae_pcb.kicad_sch)
==============================================================

Center Panel (your components)     →    Footprint to Assign
──────────────────────────────────────────────────────────────

  U1 : MCP2515-xSO                 →    Package_SO : SOIC-18W_7.5x11.6mm_P1.27mm
       How: Expand "Package_SO", double-click "SOIC-18W_7.5x11.6mm_P1.27mm".
       (May already be assigned from symbol library.)

  U2 : SN65HVD230                  →    Package_SO : SOIC-8_3.9x4.9mm_P1.27mm
       How: Expand "Package_SO", pick "SOIC-8_3.9x4.9mm_P1.27mm".
       (May already be assigned.)

  U3 : MP1584EN-LF-Z               →    Package_SO : SOIC-8-1EP_3.9x4.9mm_P1.27mm_EP2.29x3mm
       How: Search "SOIC-8-1EP" in filter. Use EP2.29x3mm variant.
       The "-1EP" suffix means "1 Exposed Pad" for the thermal pad.

  Y1 : Crystal (8 MHz)             →    Crystal : Crystal_HC49-4H_Vertical
       How: Expand "Crystal" in left panel.

  D1, D2 : D_Schottky              →    Diode_SMD : D_SMA
       How: Expand "Diode_SMD", pick "D_SMA".

  F1 : Fuse                        →    Fuse : Fuseholder_Clip-5x20mm_Littelfuse_111_Inline_P20.00x5.00mm_D1.05mm_Horizontal
       How: Expand "Fuse", search "5x20" or "Littelfuse_111". This is a clip-style
       holder for standard 5×20mm cylindrical fuses. Alternative: search
       "Fuseholder_Cylinder-5x20mm" for enclosed holders (e.g. Bulgin FX0457).

  L1 : Inductor (33µH / 3.5A)      →    Inductor_SMD : L_Bourns_SRR1260
       How: Expand "Inductor_SMD", search "SRR1260" or "Bourns". Part: Bourns SRR1260-330M (12.5×12.5mm, 33µH, 3.5A saturation). DigiKey: SRR1260-330MCT-ND.

  LED1, LED2 : LED                 →    LED_THT : LED_D3.0mm
       How: Expand "LED_THT".

  R1, R2, R3, R4, R_FB1, R_FB2     →    Resistor_SMD : R_0805_2012Metric
       How: Select all resistor rows (Ctrl+click), expand
       "Resistor_SMD", double-click "R_0805_2012Metric".

  C1, C2 : Capacitor (22pF)        →    Capacitor_SMD : C_0805_2012Metric
       How: Crystal load caps — 0805 size.

  C3, C4, C5, C6, C9, C10, C11, C12: Capacitor (100nF / 110nF)
       →    Capacitor_SMD : C_0805_2012Metric
       How: Bulk-select all 100nF decoupling caps.

  C7, C8 : Capacitor (22µF)        →    Capacitor_SMD : C_1210_3216Metric
       How: Buck input/output — 1210 for higher capacitance.

  C13, C14 : Capacitor (10µF)      →    Capacitor_SMD : C_0805_2012Metric
       How: Bulk decoupling — 10µF ceramic in 0805.

  C21 : Capacitor (C_COMP, 10nF)   →    Capacitor_SMD : C_0805_2012Metric
       How: Buck COMP pin compensation cap.

  J1 : Conn_02x20_Odd_Even         →    Connector_PinSocket_2.54mm : PinSocket_2x20_P2.54mm_Vertical
       How: Expand "Connector_PinSocket_2.54mm", search "2x20". Use female socket for shield. Part: Sullins PPPC202LFBN-RC (DigiKey S7057-ND).

  J2 : Conn_02x04_Odd_Even         →    Connector_Molex : Molex_Micro-Fit_3.0_43045-0812_2x04_P3.00mm_Vertical
       How: Expand "Connector_Molex", search "Micro-Fit". Part: Molex 43045-0812 (DigiKey WM1785-ND).

  J3 : Conn_01x06                  →    Connector_PinHeader_2.54mm : PinHeader_1x06_P2.54mm_Vertical
       How: GPS header. Part: Sullins PREC006SAAN-RC (DigiKey S1011EC-06-ND).

  J4 : Screw_Terminal_01x02        →    TerminalBlock_Phoenix : TerminalBlock_Phoenix_MKDS-1,5-2-5.08_1x02_P5.08mm_Horizontal
       How: Expand "TerminalBlock_Phoenix", search "MKDS". Part: Phoenix 1715734 (DigiKey 277-1667-ND).

  J5 : Screw_Terminal_01x02        →    TerminalBlock_Phoenix : TerminalBlock_Phoenix_MKDS-1,5-2-5.08_1x02_P5.08mm_Horizontal
       How: Power input terminal. Part: Phoenix 1715721 (DigiKey 277-1651-ND).

  J6 : Conn_01x02                  →    Connector_PinHeader_2.54mm : PinHeader_1x02_P2.54mm_Vertical
       How: If present — 2-pin header. Part: Sullins PREC002SAAN-RC (DigiKey S1011EC-02-ND).

  J7 : Conn_01x04                  →    Connector_PinHeader_2.54mm : PinHeader_1x04_P2.54mm_Vertical
       How: Display power + USB (GND, +5V, D+, D−). Part: Sullins PREC004SAAN-RC (DigiKey S1011EC-04-ND).

  JP1 : Conn_01x02                 →    Connector_PinHeader_2.54mm : PinHeader_1x02_P2.54mm_Vertical
       How: CAN termination jumper. Header: S1011EC-02-ND; shunt: Sullins STC02SYAN (DigiKey S9001-ND).

  PWR_FLAG, #PWR*, #FLG*           →    (No footprint — virtual symbols)
```

4. **Verify all assignments:** Scroll through the entire center panel and
   confirm no rows have an empty footprint field. Any component without a
   footprint will be skipped when pushing to the PCB layout.

5. Click **Apply, Save Schematic & Continue** (or **OK**) to save all assignments.

**Using the footprint preview:**

In KiCad 9.0, you can preview footprints before assigning them:
- Select a footprint in the right panel
- A 2D preview appears in the bottom-right corner
- Verify the pad count and spacing look correct for your component
- Double-click to confirm the assignment

**Quick filter tip:** The filter bar at the top of each panel accepts partial
matches. Type `0805` to see all 0805-sized footprints across all libraries.
Type `Micro-Fit` to jump straight to the Molex connector footprints.

### 4.14 Generate Netlist

1. In KiCad 9.0, you can go directly from schematic to PCB:
   - **Tools → Update PCB from Schematic** (Shortcut: F8)
   - This pushes all components and nets to the PCB editor

---

## 5. Step-by-Step: PCB Layout in KiCad 9.0

### 5.1 Open the PCB Editor

1. From the KiCad project manager, click the **PCB Editor** icon (or double-click `.kicad_pcb`)
2. If you haven't already, press **Tools → Update PCB from Schematic** (F8 in the schematic editor)
3. Click **Update PCB** → all components appear in a cluster outside the board outline

### 5.2 Set Design Rules

1. **File → Board Setup** (or click the gear icon)
2. Navigate to **Design Rules → Constraints**:
   - Minimum track width: **0.25mm** (10 mil)
   - Minimum clearance: **0.2mm** (8 mil)
   - Minimum via drill: **0.3mm**
   - Minimum via diameter: **0.6mm**
3. Navigate to **Design Rules → Net Classes**:
   - Create a net class `Power` for +12V, +5V, +3V3, GND nets:
     - Track width: **0.5mm** (20 mil) minimum, **1.0mm** preferred for power
   - Default net class (signal nets):
     - Track width: **0.3mm** (12 mil)
4. Navigate to **Design Rules → Pre-defined Sizes**:
   - Add track widths: 0.25, 0.3, 0.5, 1.0mm
   - Add via sizes: 0.8mm diameter / 0.4mm drill

### 5.3 Draw the Board Outline

1. Select **Edge.Cuts** layer from the layer panel (right side)
2. Use **Place → Line** (or press the line tool) to draw a rectangle:
   - Recommended main PCB size: **85mm × 65mm** (fits above Pi, allows room for all components)
   - The Pi 4 is 85mm × 56mm, so the shield should be roughly the same width
3. For rounded corners, use **Place → Arc** on the Edge.Cuts layer
4. Add four **mounting holes** at corners (H1–H4, already placed in this design):
   - Footprint: `MountingHole:MountingHole_2.7mm_M2.5_Pad`
   - Position at the Pi's mounting hole locations (global coords with board origin at 60,35):
     - H1: (63.5, 38.5), H2: (63.5, 87.5), H3: (121.5, 38.5), H4: (121.5, 87.5) mm
     - Equivalent to (3.5, 3.5), (3.5, 52.5), (61.5, 3.5), (61.5, 52.5) mm from board bottom-left

### 5.4 Position the 40-Pin Header

1. Find **J1** (the 2×20 female socket) in the component cluster
2. Click and drag it onto the board
3. Position it to match the Raspberry Pi's GPIO header location (shield plugs onto Pi)
4. **Lock the position:** Right-click J1 → Properties → check "Locked"

### 5.5 Component Placement Strategy

Place components in functional groups:

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   J5 (Power)     F1    D1    U3 (Buck)   C7  C8  L1               │
│   ■■             ■     ■     ■■■■■■      ■   ■   ■                │
│                                                                     │
│                              J7 (Display 5V + USB: GND, +5V, D+, D−)│
│                              ■■■■                                 │
│                                                                     │
│   J4 (CAN)    R3/JP1   U2 (Transceiver)   U1 (MCP2515)            │
│   ■■■         ■■       ■■■■               ■■■■■■■■■               │
│                                            Y1  C1  C2              │
│                                                                     │
│                                                                     │
│   LED1  LED2    J2 (Daughter Board)                                 │
│    ●     ●      ■■■■                                               │
│                                                                     │
│   ┌──────────────────────────────────────────────────────────────┐  │
│   │           J1 — 40-Pin RPi GPIO (Female Socket, Shield)       │  │
│   │    □□□□□□□□□□□□□□□□□□□□                                      │  │
│   │    □□□□□□□□□□□□□□□□□□□□                                      │  │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│   ┌───────────────────────────────────┐                            │
│   │      GPS MODULE MOUNTING AREA     │   (J3 + mounting holes)    │
│   │      SparkFun NEO-M9N            │                            │
│   │      33mm × 40.6mm              │                            │
│   └───────────────────────────────────┘                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Placement order:**

1. **J1** (Pi female socket) — bottom/center, this is the anchor
2. **U3 + power components** — top-left corner, near J5
3. **J7** (display power + USB, 4-pin) — top area near power, edge-accessible
4. **U1 + U2 + CAN components** — middle area, between power and Pi header
5. **J4** (CAN terminal) — board edge, accessible
6. **J5** (power terminal) — board edge, accessible
7. **J2** (daughter board) — board edge, accessible
8. **J3 + GPS area** — bottom section or separate area with clearance
9. **Decoupling caps** — as close as physically possible to their IC's power pins

### 5.6 Route Critical Traces First

Use **X** to start routing a trace. KiCad 9 has an interactive router.

**Route in this priority order:**

1. **Power traces (+5V, +3V3, GND):**
   - Use 1.0mm trace width for +5V from buck converter to J1 and to J7 pin 2 (display power)
   - Use 0.5mm minimum for +3V3 distribution
   - Consider a ground pour (copper fill) on the back layer

2. **SPI bus (SCK, MOSI, MISO, CS):**
   - Keep traces short and parallel
   - Route from J1 directly to U1 (MCP2515)
   - Use 0.3mm traces
   - Keep SPI traces away from power switching (buck converter) area

3. **CAN bus differential pair (CANH, CANL):**
   - Route from U2 to J4 as a pair, keep them parallel and close together
   - Use 0.3mm traces

4. **UART (GPS_TX, GPS_RX):**
   - Route from J1 to J3
   - Use 0.3mm traces

5. **GPIO signals (buttons, LEDs):**
   - Route from J1 to J2
   - Use 0.3mm traces, no special constraints

6. **Display power (J7 pins 1–2):**
   - Route from +5V rail to J7 pin 2 with 1.0mm traces (carries up to 450mA)

### 5.7 Add Ground Plane (Copper Pour)

1. Select **B.Cu** (back copper) layer
2. **Place → Add Filled Zone** (or press the zone tool)
3. Click around the board outline to create a zone matching the board shape
4. In the zone properties dialog:
   - Net: `GND`
   - Clearance: 0.3mm
   - Minimum width: 0.25mm
5. Press **B** to re-fill all zones
6. Optionally add a ground pour on **F.Cu** (front copper) in open areas too

### 5.8 Add Silkscreen Labels

1. Select **F.Silkscreen** layer
2. **Place → Text** to add labels:
   - "FSAE Display — Main PCB v1.0" (title)
   - "12V" and "GND" near J5
   - "CAN_H", "CAN_L" near J4 (GND via J5)
   - "GPS (NEO-M9N)" near J3
   - "DAUGHTER BOARD" near J2
   - "DISPLAY 5V" near J7 pins 1–2
   - "DISPLAY USB" near J7 pins 3–4
   - Component values near each component

### 5.9 Run Design Rule Check (DRC)

1. **Inspect → Design Rules Check**
2. Click **Run DRC**
3. Fix all errors (unconnected nets, clearance violations, etc.)
4. Common issues:
   - **Unrouted nets**: Route any remaining connections
   - **Clearance errors**: Adjust component spacing
   - **Courtyard overlaps**: Move components apart

### 5.10 Generate Gerber Files for Manufacturing

1. **File → Fabrication Outputs → Gerbers**
2. Set output directory to a `gerbers/` subfolder
3. Select layers:
   - F.Cu, B.Cu (copper)
   - F.Silkscreen, B.Silkscreen
   - F.Mask, B.Mask (solder mask)
   - Edge.Cuts (board outline)
4. Click **Generate Drill Files** → set format to Excellon
5. Click **Plot** to generate Gerbers
6. Zip the `gerbers/` folder and upload to your PCB manufacturer (JLCPCB, PCBWay, OSH Park, etc.)

---

## 6. Design Rules & Manufacturing Notes

### 6.1 PCB Specifications

| Parameter | Value |
|---|---|
| Layers | 2 (F.Cu + B.Cu) |
| Board thickness | 1.6mm |
| Copper weight | 1 oz (35µm) |
| Min trace width | 0.25mm (10 mil) |
| Min clearance | 0.2mm (8 mil) |
| Min drill | 0.3mm |
| Solder mask | Green (or black for aesthetics) |
| Silkscreen | White |
| Surface finish | HASL (cheapest) or ENIG |

### 6.2 Power Budget

| Rail | Source | Max Current | Consumers |
|---|---|---|---|
| +12V | Car battery | 3A (fuse limited) | Buck converter input |
| +5V | MP1584EN output | 3A max | Raspberry Pi 4 (~2.5A peak), GPS module (via 3.3V reg, ~30mA), **Display HU-043WISBUAA1-B (~450mA max via J7)** |
| +3V3 | Pi's onboard regulator | ~800mA | MCP2515 (~5mA), SN65HVD230 (~70mA), LEDs (~12mA), Buttons pullups (~1mA), Daughter board LEDs (~12mA), **NEO-M9N GPS (~30mA)** |

> **5V Rail Total Budget:** Pi (~2.5A) + Display (~0.45A) + overhead = ~3.0A.
> This is at the MP1584EN's 3A rating with essentially no margin. **Strongly
> consider upgrading to a 5A-rated buck converter** (e.g., XL4015 module,
> LM2596-based module, or TPS54531 design) for reliable operation under
> sustained load.

> The +3V3 rail comes from the Raspberry Pi's own 3.3V regulator (accessible
> on GPIO header pins 1 and 17). Total 3.3V load is well within the Pi's ~800mA
> 3.3V budget.

### 6.3 Thermal Considerations

- **MP1584EN:** Needs adequate copper area on the exposed pad for heat dissipation.
  Place thermal vias (0.3mm drill, array of 4-6 vias) under the exposed pad
  connecting to the ground plane on the back layer. The added display load
  (~450mA at 5V) increases thermal dissipation — ensure the ground plane
  under U3 has ample copper area.
- **SN65HVD230:** Low power dissipation, no special thermal management needed.
- **MCP2515:** Low power, no special thermal management needed.

### 6.4 EMC / Signal Integrity Tips

- Keep the crystal (Y1) and its load capacitors (C1, C2) as close as possible to
  U1 pins 7 and 8. Minimize trace length to < 10mm.
- Route CAN_H and CAN_L as a differential pair on the same layer.
- Keep the SPI clock (SCK) trace short to minimize radiation.
- Place all decoupling capacitors within 5mm of their associated IC power pins.
- The ground plane on B.Cu provides a solid return path for all signals.
- If routing USB signals for the display touch (J7 pins 3–4), keep D+ and D- as a
  differential pair with controlled impedance (90Ω differential). For short
  runs (< 50mm), this is less critical.

### 6.5 Assembly Notes

1. **Solder SMD components first** (resistors, caps, ICs) — use solder paste and
   hot air or reflow oven
2. **Solder through-hole components second** (headers, connectors, crystal, LEDs)
3. **Test power section independently:**
   - Apply 12V to J5 before connecting Pi
   - Verify +5V output at J1 pins 2/4 AND at J7 with multimeter
   - Verify no shorts on any rail
4. **Test CAN section:**
   - Plug onto Pi, enable SPI and MCP2515 driver in `raspi-config`
   - Run `ip link set can0 up type can bitrate 500000`
   - Use `candump can0` to verify messages
5. **Test GPS (NEO-M9N):**
   - Enable UART in `raspi-config` (disable serial console, enable serial port)
   - Plug NEO-M9N breakout into J3
   - Default baud rate: 38400 (u-blox M9 default)
   - Run `cat /dev/ttyAMA0` or use `gpsd` to verify NMEA sentences
   - For Python: use the `pyubx2` or `pynmeagps` library
6. **Test Display:**
   - Connect HDMI cable from Pi to display CN1
   - Connect J7 to display CN2 (5V power)
   - Connect display CN4 USB to Pi USB port (touch input)
   - Display should show Pi desktop at 800×480

---

## 7. Raspberry Pi 40-Pin Pinout Reference

For quick reference during schematic entry:

```
                    3V3  (1) (2)  5V
          GPIO2/SDA (3) (4)  5V
          GPIO3/SCL (5) (6)  GND
               GPIO4 (7) (8)  GPIO14/TXD   ←── GPS_RX
                 GND (9) (10) GPIO15/RXD   ←── GPS_TX
        GPIO17/BTN1 (11) (12) GPIO18
        GPIO27/BTN2 (13) (14) GND
        GPIO22/BTN3 (15) (16) GPIO23/LED_DB1
                3V3 (17) (18) GPIO24/LED_DB2
     GPIO10/SPI_MOSI(19) (20) GND
     GPIO9/SPI_MISO (21) (22) GPIO25/MCP_INT
     GPIO11/SPI_SCLK(23) (24) GPIO8/SPI_CE0
                 GND (25) (26) GPIO7/CE1
           GPIO0/DNC(27) (28) GPIO1/DNC
       GPIO5/LED_DB3(29) (30) GND
              GPIO6 (31) (32) GPIO12
              GPIO13(33) (34) GND
              GPIO19(35) (36) GPIO16
              GPIO26(37) (38) GPIO20
                 GND(39) (40) GPIO21

    DNC = Do Not Connect (reserved for HAT EEPROM)
```

---

## Appendix A: KiCad 9.0 Library Quick Reference

Open the **Symbol Chooser** (press A in schematic editor) and type these
exactly to find each part:

| Search Term | Library:Symbol Found |
|---|---|
| `MCP2515` | Interface_CAN_LIN : MCP2515-xSO |
| `SN65HVD230` | Interface_CAN_LIN : SN65HVD230 |
| `Crystal` | Device : Crystal |
| `LED` | Device : LED |
| `Conn_02x20` | Connector_Generic : Conn_02x20_Odd_Even |
| `Conn_02x04` | Connector_Generic : Conn_02x04_Odd_Even |
| `Conn_01x06` | Connector_Generic : Conn_01x06 |
| `Conn_01x02` | Connector_Generic : Conn_01x02 |
| `Screw_Terminal_01x02` | Connector : Screw_Terminal_01x02 |
| `Screw_Terminal_01x02` | Connector : Screw_Terminal_01x02 |
| `D_Schottky` | Device : D_Schottky |
| `Fuse` | Device : Fuse |
| `C` | Device : C |
| `C_Polarized` | Device : C_Polarized |
| `R` | Device : R |
| `L` | Device : L |
| `+12V` | power : +12V |
| `+5V` | power : +5V |
| `+3V3` | power : +3V3 |
| `GND` | power : GND |
| `PWR_FLAG` | power : PWR_FLAG |
| `MP1584` | Regulator_Switching : MP1584EN (if available, else see §3.1) |

Open the **Footprint Chooser** (in Assign Footprints dialog) and search:

| Search Term | Library : Footprint |
|---|---|
| `SOIC-18W` | Package_SO : SOIC-18W_7.5x11.6mm_P1.27mm |
| `SOIC-8_3.9` | Package_SO : SOIC-8_3.9x4.9mm_P1.27mm |
| `SOIC-8-1EP` | Package_SO : SOIC-8-1EP_3.9x4.9mm_P1.27mm_EP2.29x3mm |
| `Crystal_HC49` | Crystal : Crystal_HC49-4H_Vertical |
| `LED_D3` | LED_THT : LED_D3.0mm |
| `R_0805` | Resistor_SMD : R_0805_2012Metric |
| `C_0805` | Capacitor_SMD : C_0805_2012Metric |
| `C_1210` | Capacitor_SMD : C_1210_3216Metric |
| `PinSocket_2x20` | Connector_PinSocket_2.54mm : PinSocket_2x20_P2.54mm_Vertical |
| `PinHeader_1x02` | Connector_PinHeader_2.54mm : PinHeader_1x02_P2.54mm_Vertical |
| `PinHeader_1x04` | Connector_PinHeader_2.54mm : PinHeader_1x04_P2.54mm_Vertical |
| `PinHeader_1x06` | Connector_PinHeader_2.54mm : PinHeader_1x06_P2.54mm_Vertical |
| `Micro-Fit_3.0` | Connector_Molex : Molex_Micro-Fit_3.0_43045-0812_2x04_P3.00mm_Vertical |
| `MKDS-1,5-2` | TerminalBlock_Phoenix : TerminalBlock_Phoenix_MKDS-1,5-2-5.08_1x02_P5.08mm_Horizontal |
| `MKDS-1,5-2` | TerminalBlock_Phoenix : TerminalBlock_Phoenix_MKDS-1,5-2-5.08_1x02_P5.08mm_Horizontal |
| `D_SMA` | Diode_SMD : D_SMA |
| `5x20` or `Littelfuse_111` | Fuse : Fuseholder_Clip-5x20mm_Littelfuse_111_Inline_P20.00x5.00mm_D1.05mm_Horizontal |
| `L_Bourns_SRR1260` or `SRR1260` | Inductor_SMD : L_Bourns_SRR1260 |
| `MountingHole_2.7mm` | MountingHole : MountingHole_2.7mm_M2.5_Pad |

---

## Appendix B: Common Pitfalls & Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| Pi won't boot when shield connected | 5V short or wrong pin alignment | Verify 5V and GND pins with meter before inserting Pi |
| No CAN messages | MCP2515 not initialized, wrong SPI wiring | Check SPI wires, enable `dtoverlay=mcp2515-can0,oscillator=8000000,interrupt=25` in `/boot/config.txt` |
| GPS no data | UART not enabled, TX/RX swapped | Enable UART in raspi-config, swap TX↔RX wires. NEO-M9N default baud is 38400. |
| GPS slow first fix | Cold start without sky view | Ensure chip antenna has clear sky view; the onboard backup battery enables ~2s hot starts after initial lock |
| Buck converter overheating | Insufficient copper under exposed pad, high load from display (~450mA) + Pi (~2.5A) = ~3A | Add thermal vias, increase copper pour area. **Upgrade to 5A buck** if running Pi 4 + display at full load |
| Display not powering on | J7 not connected, voltage out of range | Verify 5V ±0.5V at J7 with multimeter. Display accepts 4.5–5.5V |
| Display touch not working | USB not connected, wrong J7 pins 3/4 wiring | Connect display CN4 to Pi USB port. J7 pin 3 = D+, pin 4 = D−. Verify not swapped |
| ERC error: power pin not driven | Missing PWR_FLAG | Add `PWR_FLAG` symbol on +12V input net and GND net |
| Buttons read always LOW | Missing pullup resistors | Verify 10kΩ pullups are connected to +3V3 (on daughter board) |
| LEDs always on/off | Reversed polarity | Check anode/cathode orientation; flat side/short leg = cathode |
| CAN termination issues | Jumper not installed or 120Ω missing | Install jumper on JP1; verify with ohmmeter between CAN_H and CAN_L |

---

## Appendix C: Config.txt Overlay for MCP2515 & GPS

Add to `/boot/config.txt` on the Raspberry Pi:

```
# Enable SPI
dtparam=spi=on

# MCP2515 CAN controller on SPI0.0, 8MHz oscillator, interrupt on GPIO25
dtoverlay=mcp2515-can0,oscillator=8000000,interrupt=25

# Enable UART for GPS (disable Bluetooth on UART, use miniUART for BT)
dtoverlay=miniuart-bt
enable_uart=1

# HDMI settings for HU-043WISBUAA1-B (4.3" 800x480 display)
hdmi_group=2
hdmi_mode=87
hdmi_cvt=800 480 60 6 0 0 0
hdmi_drive=2
```

After reboot, bring up the CAN interface:

```bash
sudo ip link set can0 up type can bitrate 500000
candump can0
```

For GPS (NEO-M9N), read NMEA data:

```bash
# NEO-M9N defaults to 38400 baud
stty -F /dev/ttyAMA0 38400
cat /dev/ttyAMA0
# or use gpsd:
sudo gpsd /dev/ttyAMA0 -F /var/run/gpsd.sock
cgps -s
```

---

## Appendix D: Display Connector Pinout Reference (HU-043WISBUAA1-B)

**CN1 — HDMI Input (HDMI A Type):**
Standard HDMI connector. Connect directly from Pi HDMI port via cable.

**CN2 — Power Input (2.0mm Wafer, 2-pin):**

| Pin | Symbol | Function |
|---|---|---|
| 1 | 5.0V | Power Supply +5V → connect to J7 Pin 2 |
| 2 | GND | Ground → connect to J7 Pin 1 |

**CN3 — Backlight Control (1.25mm Wafer, 3-pin):**

| Pin | Symbol | Function |
|---|---|---|
| 1 | GND | Ground |
| 2 | PWM | Backlight dimming (internal pullup to 3.3V). Leave floating for full brightness. |
| 3 | NC | No connection |

**CN4 — Capacitive Touch Panel USB (1.25mm Wafer, 5-pin):**

| Pin | Symbol | Function |
|---|---|---|
| 1 | GND-EARTH | Earth Ground (Shield) |
| 2 | VDD_5V | USB power supply (+5V) |
| 3 | GND | Power ground |
| 4 | D+ | USB data + |
| 5 | D- | USB data - |

Touch panel uses FT5426 driver IC. Supports Windows, Linux, and Android.
Response time: 25ms. Supports up to 5 simultaneous touch points.

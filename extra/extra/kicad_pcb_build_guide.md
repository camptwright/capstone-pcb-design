# FSAE Display System — Complete KiCad 9.0 PCB Build Guide

> Full schematic, wiring diagrams, exact KiCad library references, and step-by-step
> instructions for the **Main PCB (Pi Shield)** and **Daughter Board (Button/LED Panel)**.

---

## Table of Contents

1. [Bill of Materials & KiCad Library Map](#1-bill-of-materials--kicad-library-map)
2. [Main PCB — Full Schematic & Net List](#2-main-pcb--full-schematic--net-list)
3. [Daughter Board — Full Schematic & Net List](#3-daughter-board--full-schematic--net-list)
4. [Custom Symbol & Footprint Creation](#4-custom-symbol--footprint-creation)
5. [Step-by-Step: Building the Schematic in KiCad 9.0](#5-step-by-step-building-the-schematic-in-kicad-90)
6. [Step-by-Step: PCB Layout in KiCad 9.0](#6-step-by-step-pcb-layout-in-kicad-90)
7. [Design Rules & Manufacturing Notes](#7-design-rules--manufacturing-notes)
8. [Raspberry Pi 40-Pin Pinout Reference](#8-raspberry-pi-40-pin-pinout-reference)

---

## 1. Bill of Materials & KiCad Library Map

Every component below maps to a **KiCad 9.0 default library symbol** and
**footprint**. Where a default symbol does not exist, the table says **CUSTOM**
and Section 4 shows how to create it.

### 1.1 Main PCB Components

| Ref | Component | Value | KiCad Symbol Library : Symbol | KiCad Footprint Library : Footprint | Notes |
|-----|-----------|-------|-------------------------------|-------------------------------------|-------|
| U1 | CAN Controller | MCP2515-I/SO | `Interface_CAN_LIN:MCP2515-xSO` | `Package_SO:SOIC-18W_7.5x11.6mm_P1.27mm` | SPI CAN controller |
| U2 | CAN Transceiver | SN65HVD230DR | `Interface_CAN_LIN:SN65HVD230` | `Package_SO:SOIC-8_3.9x4.9mm_P1.27mm` | 3.3V CAN transceiver |
| U3 | Buck Converter IC | MP1584EN | **CUSTOM** (see §4.1) | `Package_SO:SOIC-8-1EP_3.9x4.9mm_P1.27mm_EP2.41x3.30mm` | 12V→5V step-down. Alt: use pre-built module (see §4.2) |
| Y1 | Crystal | 8 MHz | `Device:Crystal` | `Crystal:Crystal_HC49-4H_Vertical` | For MCP2515 oscillator |
| D1 | Schottky Diode | SS34 | `Device:D_Schottky` | `Diode_SMD:D_SMA` | Reverse polarity protection on 12V input |
| D2 | Schottky Diode | SS34 | `Device:D_Schottky` | `Diode_SMD:D_SMA` | Buck converter freewheeling diode |
| F1 | Fuse | 3A | `Device:Fuse` | `Fuse:Fuseholder5x20_Horizontal` | Input protection (use PTC resettable or blade) |
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
| C3, C4, C5, C6 | Capacitor | 100nF | `Device:C` | `Capacitor_SMD:C_0805_2012Metric` | Decoupling (U1 VDD, U2 VCC, U3, general) |
| C7 | Capacitor | 22µF / 25V | `Device:C_Polarized` | `Capacitor_SMD:C_1210_3216Metric` | Buck input cap |
| C8 | Capacitor | 22µF / 10V | `Device:C_Polarized` | `Capacitor_SMD:C_1210_3216Metric` | Buck output cap |
| C9 | Capacitor | 100nF | `Device:C` | `Capacitor_SMD:C_0805_2012Metric` | Buck bootstrap cap |
| C10 | Capacitor | 10µF | `Device:C_Polarized` | `Capacitor_SMD:C_0805_2012Metric` | Bulk decoupling 3.3V rail |
| C11 | Capacitor | 10µF | `Device:C_Polarized` | `Capacitor_SMD:C_0805_2012Metric` | Bulk decoupling 5V rail |
| JP1 | Jumper Header | 2-pin + shunt | `Connector_Generic:Conn_01x02` | `Connector_PinHeader_2.54mm:PinHeader_1x02_P2.54mm_Vertical` | CAN termination enable. Header: Sullins PREC002SAAN-RC (DigiKey S1011EC-02-ND). Shunt: Sullins STC02SYAN (DigiKey S9001-ND). |
| J1 | 40-pin RPi Header | 2×20 Female socket | `Connector_Generic:Conn_02x20_Odd_Even` | `Connector_PinSocket_2.54mm:PinSocket_2x20_P2.54mm_Vertical` | Shield: female socket plugs onto Raspberry Pi 4 GPIO. **Part:** Sullins PPPC202LFBN-RC. **DigiKey:** S7057-ND. |
| J2 | Daughter Board Connector | Molex Micro-Fit 3.0 8-pin | `Connector_Generic:Conn_02x04_Odd_Even` | `Connector_Molex:Molex_Micro-Fit_3.0_43045-0812_2x04_P3.00mm_Vertical` | Male PCB header. **Part:** Molex 43045-0812. **DigiKey:** WM1785-ND. |
| J3 | GPS Header | 1×16 Female 1.0mm | `Connector_Generic:Conn_01x16` | `Connector_PinHeader_1.0mm:PinHeader_1x16_P1.00mm_Vertical` | SparkFun ZED-F9P mount (see §4.3). For NEO-M9N use 1×06 2.54mm: PinHeader_1x06_P2.54mm_Vertical, Sullins PREC006SAAN-RC (DigiKey S1011EC-06-ND). |
| J4 | CAN Bus Terminal | 2-pos Screw | `Connector:Screw_Terminal_01x02` | `TerminalBlock_Phoenix:TerminalBlock_Phoenix_MKDS-1,5-2-5.08_1x02_P5.08mm_Horizontal` | CAN_H, CAN_L only (2-wire CAN). GND via J5. **Part:** Phoenix 1715721 (MKDS 1.5/2-5.08). **DigiKey:** 277-1651-ND. |
| J5 | Power Input Terminal | 2-pos Screw | `Connector:Screw_Terminal_01x02` | `TerminalBlock_Phoenix:TerminalBlock_Phoenix_MKDS-1,5-2-5.08_1x02_P5.08mm_Horizontal` | 12V, GND from car. **Part:** Phoenix 1715721. **DigiKey:** 277-1651-ND. |

### 1.2 Daughter Board Components

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
| J6 | Daughter Board Connector | Molex Micro-Fit 3.0 8-pin | `Connector_Generic:Conn_02x04_Odd_Even` | `Connector_Molex:Molex_Micro-Fit_3.0_43045-0812_2x04_P3.00mm_Vertical` | Mating connector to main PCB (same as J2). **Part:** Molex 43045-0812. **DigiKey:** WM1785-ND. Harness: Molex 43025-0800 (WM2014-ND), 43030-0007 crimp pins (WM1142CT-ND). |

### 1.3 Power Symbols (used on both boards)

| Symbol | KiCad Power Library : Symbol |
|--------|------------------------------|
| +12V | `power:+12V` |
| +5V | `power:+5V` |
| +3V3 | `power:+3V3` |
| GND | `power:GND` |

---

## 2. Main PCB — Full Schematic & Net List

### 2.1 Block Diagram

```
  12V IN ─► [D1 Schottky] ─► [F1 Fuse] ─► [U3 MP1584EN Buck 12V→5V] ─► +5V Rail
                                                                           │
                                                   ┌──────────────────────┘
                                                   ▼
                                            Raspberry Pi 4
                                          (via J1 40-pin header)
                                                   │
                              ┌────────────────────┼────────────────────┐
                              │ SPI                │ UART               │ GPIO
                              ▼                    ▼                    ▼
                         ┌─────────┐         ┌──────────┐        ┌──────────┐
                         │   U1    │         │  J3 GPS  │        │ J2 Daugh │
                         │ MCP2515 │         │ ZED-F9P  │        │  Board   │
                         └────┬────┘         │ Breakout │        │ 8-pin    │
                              │              └──────────┘        └──────────┘
                              ▼
                         ┌─────────┐
                         │   U2    │
                         │SN65HVD  │
                         │  230    │──── J4 CAN Bus ──── To Car
                         └─────────┘
```

### 2.2 Power Supply Section

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
                    │              │
         +12V_FUSED─►│ IN        BST│──── C9 (100nF) ──── SW pin
                    │              │
                    │ EN        SW │──►── L1 (33µH) ──►── +5V
                    │  │           │           │
                    │  └─ +12V     │         C8 (22µF)
                    │  (EN=high)   │           │
                    │           GND│──── GND   │
                    │              │           GND
                    │ FB       EP │
                    │  │       GND │
                    └──┼───────────┘
                       │
                  ┌────┴────┐
                  │         │
               R_FB1     R_FB2
              (100kΩ)   (19.1kΩ)
                  │         │
                 +5V       GND

    D2 (SS34) cathode to SW pin, anode to GND (freewheeling diode)

    C7 (22µF/25V) between +12V_FUSED and GND (input decoupling)
    C8 (22µF/10V) between +5V and GND (output decoupling)
    C11 (10µF) between +5V and GND (additional bulk)
```

**MP1584EN Pin Connections:**

| MP1584EN Pin | Net | Connection |
|---|---|---|
| 1 - IN | +12V_FUSED | From fuse output, through C7 to GND |
| 2 - SW | SW_NODE | To L1 input, D2 cathode, C9 other side |
| 3 - GND | GND | Ground plane |
| 4 - FB | FB_NODE | Junction of R_FB1 and R_FB2 voltage divider |
| 5 - EN | +12V_FUSED | Tied high to enable (or add enable circuit) |
| 6 - NC | — | No connect |
| 7 - BST | BST_NODE | C9 (100nF) to SW pin |
| 8 - VCC | VCC_INT | Internal regulator, 100nF to GND (C5) |
| EP - Exposed Pad | GND | Thermal ground pad |

> **Alternative (Pre-Built Module):** If using a pre-built MP1584 buck module
> instead, skip U3/D2/L1/C7/C8/C9/R_FB1/R_FB2 and instead place a 4-pin
> header (IN+, IN−, OUT+, OUT−). See Section 4.2.

### 2.3 CAN Bus Section

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
                    │ 7:OSC2 ───┐                   │    │        │
                    │           │                   │    │        │
                    │ 8:OSC1 ─┐ │                   │    │        │
                    │         │ │                   │    │        │
                    │ 9:VSS   │ │                   │    │        │
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
| 4 | ~TX0BF | — | No connect |
| 5 | ~TX1BF | — | No connect |
| 6 | ~TX2RTS | +3V3 | Tie high (inactive) |
| 7 | OSC2 | XTAL2 | Y1 pin 2 + C2 (22pF→GND) |
| 8 | OSC1 | XTAL1 | Y1 pin 1 + C1 (22pF→GND) |
| 9 | VSS | GND | Ground |
| 10 | ~RX1BF | — | No connect |
| 11 | ~RX0BF | — | No connect |
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

### 2.4 GPS Module Section (SparkFun ZED-F9P Breakout)

The SparkFun GPS-RTK-SMA Breakout (GPS-16481) is mounted directly onto the
main PCB using female pin headers. The breakout board has 0.1"-spaced PTH pins
along its edges. We only connect the four signals we need; the rest are left
unconnected on the main PCB side.

**Board dimensions:** 43.5mm × 43.2mm (1.71" × 1.70")

```
SCHEMATIC: GPS Module Connection
=================================

    SparkFun ZED-F9P Breakout (mounted on main PCB via pin headers)

    Main PCB provides these connections through J3 header:

         J3 Female Header (on main PCB, breakout plugs in)

         Pin mapping (matching SparkFun board PTH edge):
         ┌──────────────────────────────────────────┐
         │  GND  3V3  TXO  RXI  SDA  SCL  ...      │
         │   │    │    │    │                        │
         └───┼────┼────┼────┼────────────────────────┘
             │    │    │    │
            GND +3V3   │    │
                       │    │
                       │    └──── Pi GPIO14 / TXD (J1 pin 8)
                       │          (Pi TX → GPS RXI)
                       │
                       └───────── Pi GPIO15 / RXD (J1 pin 10)
                                  (GPS TXO → Pi RX)
```

**Connections used:**

| GPS Breakout Pin | Main PCB Net | RPi GPIO (J1 pin) | Direction |
|---|---|---|---|
| GND | GND | Pin 6, 9, 14, etc. | — |
| 3V3 | +3V3 | Pin 1 (3.3V) | Power to GPS |
| TXO (UART1 TX) | GPS_TX | GPIO15 / RXD (pin 10) | GPS → Pi |
| RXI (UART1 RX) | GPS_RX | GPIO14 / TXD (pin 8) | Pi → GPS |

> **Important:** The ZED-F9P operates at 3.3V logic levels natively — no level
> shifting is needed for the Raspberry Pi's 3.3V GPIO.

**Physical Mounting:** Create a rectangular keep-out zone (45mm × 45mm) on the
main PCB with four M2.5 mounting holes matching the SparkFun board's corner
holes (see §4.3). The female pin header solders to the main PCB; the breakout
board's male pins plug into it.

### 2.5 Status LEDs (On-Board)

```
SCHEMATIC: On-Board Status LEDs
================================

    +3V3 ──► R_LED_PWR (not GPIO-controlled; always on when powered)
              │
             LED1 (Green, 3mm)  ← Power indicator
              │
             GND

    GPIO23 ──► R1 (330Ω) ──► LED_CAN_ANODE ──► LED2 (Blue 3mm) ──► GND
    (J1 pin 16)                                   CAN status
```

Wait — looking at the original design, the on-board LEDs (PWR, CAN) are fixed
status indicators. The three GPIO-driven LEDs (GPIO23, GPIO24, GPIO5) go to
the daughter board. Let me clarify:

**On-board LEDs (Main PCB):**
- LED1 (Green): Power indicator — connected directly from +3V3 through R1 (330Ω) to GND. Always on when 3.3V rail is active.
- LED2 (Blue): CAN activity — optional, could be driven by MCP2515 ~TX0BF or ~RX0BF pin, or directly from software via a spare GPIO.

For simplicity, wire LED2 to MCP2515 pin 11 (~RX0BF) which can be configured in software to pulse on CAN receive:

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

    MCP2515 Pin 11 (~RX0BF)    (Active low = LED on when CAN frame received)
     │
    R2 (330Ω)
     │
    LED2 (Blue) Cathode
     │
    LED2 Anode
     │
    +3V3
```

### 2.6 Daughter Board Connector

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

### 2.7 Raspberry Pi 40-Pin Header (J1) — Complete Pin Assignment

| J1 Phys Pin | RPi GPIO | Net Name | Connects To |
|---|---|---|---|
| 1 | 3.3V | +3V3 | Power rail, GPS 3V3, Daughter board 3V3 |
| 2 | 5V | +5V | Buck converter output (powers Pi) |
| 3 | GPIO2 (SDA1) | — | Reserved (I2C, unused) |
| 4 | 5V | +5V | Buck converter output |
| 5 | GPIO3 (SCL1) | — | Reserved (I2C, unused) |
| 6 | GND | GND | Ground |
| 7 | GPIO4 | — | Unused |
| 8 | GPIO14 (TXD) | GPS_RX | ZED-F9P RXI (Pi transmits to GPS) |
| 9 | GND | GND | Ground |
| 10 | GPIO15 (RXD) | GPS_TX | ZED-F9P TXO (GPS transmits to Pi) |
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

### 2.8 Complete Main PCB Net List

| Net Name | Nodes (Ref:Pin) |
|---|---|
| +12V_IN | J5:1, D1:Anode |
| +12V_FUSED | D1:Cathode, F1:1 |
| +12V_POST_FUSE | F1:2, U3:IN, U3:EN, C7:+, D2:Cathode(note: D2 cathode to SW) |
| SW_NODE | U3:SW, L1:1, D2:Cathode, C9:1 |
| BST_NODE | U3:BST, C9:2 |
| FB_NODE | U3:FB, R_FB1:2, R_FB2:1 |
| +5V | L1:2, C8:+, C11:+, R_FB1:1, J1:2, J1:4 |
| +3V3 | J1:1, J1:17, U1:18, U2:3, U1:6, R4:1, C3:1, C4:1, C10:+, J2:7, J3:3V3, LED1_R1 junction, LED2 anode |
| GND | J5:2, D1:common-anode-return(no), C7:−, C8:−, C9:gnd-side(no), C10:−, C11:−, U3:GND, U3:EP, U1:9, U2:2, U2:8, C1:2, C2:2, C3:2, C4:2, J1:6,9,14,20,25,30,34,39, J2:8, J3:GND, J4:3, LED1:Cathode, R_FB2:2 |
| XTAL1 | U1:8, Y1:1, C1:1 |
| XTAL2 | U1:7, Y1:2, C2:1 |
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
| GPS_TX | J3:TXO, J1:10 (GPIO15/RXD) |
| GPS_RX | J3:RXI, J1:8 (GPIO14/TXD) |
| BTN1 | J2:4, J1:11 (GPIO17) |
| BTN2 | J2:5, J1:13 (GPIO27) |
| BTN3 | J2:6, J1:15 (GPIO22) |
| LED_DB1 | J2:1, J1:16 (GPIO23) |
| LED_DB2 | J2:2, J1:18 (GPIO24) |
| LED_DB3 | J2:3, J1:29 (GPIO5) |

---

## 3. Daughter Board — Full Schematic & Net List

### 3.1 Schematic

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

**How the buttons work:** Each button line is pulled HIGH to +3V3 through a
10kΩ resistor. When the button is pressed, it shorts to GND, pulling the line
LOW. The Pi reads LOW = pressed, HIGH = released. The 100nF caps filter
contact bounce.

**How the LEDs work:** The Pi GPIO pin drives HIGH (3.3V) through the 330Ω
resistor into the LED anode. Current flows through the LED to GND.
I_LED ≈ (3.3V − V_fwd) / 330Ω ≈ (3.3 − 2.0) / 330 ≈ 4mA (safe for GPIO).

### 3.2 Daughter Board Net List

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

## 4. Custom Symbol & Footprint Creation

### 4.1 Custom Symbol: MP1584EN Buck Converter (if not in default library)

KiCad 9.0 may include `Regulator_Switching:MP1584EN` in its default libraries.
Check first: in the Schematic Editor, press **A** (Add Symbol) and search for
`MP1584`. If found, use it. If not:

1. **Schematic Editor → Tools → Symbol Editor** (or open standalone Symbol Editor)
2. **File → New Library** → save as `CustomParts.kicad_sym` in your project folder
3. **Create New Symbol** → Name: `MP1584EN`
4. Draw a rectangle, add pins:

```
    Pin Layout:

          ┌───────────┐
    IN  ──┤ 1       8 ├── VCC
    SW  ──┤ 2       7 ├── BST
    GND ──┤ 3       6 ├── NC
    FB  ──┤ 4       5 ├── EN
          └─────┬─────┘
                │ EP (Exposed Pad = GND)
```

5. Set pin types:
   - IN: Power Input
   - SW: Passive
   - GND: Power Input
   - FB: Input
   - EN: Input
   - NC: Not Connected
   - BST: Passive
   - VCC: Power Output
   - EP: Power Input

6. Save. Assign footprint: `Package_SO:SOIC-8-1EP_3.9x4.9mm_P1.27mm_EP2.41x3.30mm`

> **Easier alternative:** Download the MP1584EN symbol from
> [SnapEDA](https://www.snapeda.com/parts/MP1584EN-LF-Z/MPS/view-part/) or
> [Ultra Librarian](https://www.ultralibrarian.com/) — both export
> directly to KiCad format.

### 4.2 Alternative: Pre-Built MP1584 Buck Module Footprint

If using a pre-built module (the common blue/red $2 boards on Amazon), create
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

### 4.3 Custom Footprint: SparkFun ZED-F9P Breakout Mount

The SparkFun GPS-RTK-SMA board is 43.5mm × 43.2mm with PTH pins along one or
both edges. Create a footprint to mount it:

**Option A — Minimal 4-Pin Connection (Recommended):**

Use a standard 1×4 female pin header. Solder male headers to the GPS board's
GND, 3V3, TXO, RXI pins. Plug into the female header on the main PCB. Add
mounting holes for mechanical support.

1. **Footprint Editor → New Footprint** → Name: `SparkFun_ZED-F9P_Mount`
2. Place a 1×4 THT pad row at 2.54mm pitch for the signal connection
3. Add 4 NPTH (Non-Plated Through Hole) mounting holes at the four corners:
   - Hole diameter: 2.5mm (for M2 screws)
   - Rectangle: 38.1mm × 38.1mm (measure from SparkFun board drawing)
4. Draw courtyard: 45mm × 45mm rectangle
5. Draw silkscreen outline showing board placement area
6. In the schematic, use symbol `Connector_Generic:Conn_01x04` and label pins:
   - Pin 1: GND
   - Pin 2: 3V3
   - Pin 3: TXO
   - Pin 4: RXI

**Option B — Full Edge Header (all breakout pins):**

If you want access to all SparkFun board PTH pins, create a 1×16 header
footprint matching the breakout's edge pins. This allows future use of I2C,
SPI, PPS, etc.

**Schematic for Option A (J3):**

| J3 Pin | Net | Connects To |
|---|---|---|
| 1 | GND | Ground plane |
| 2 | +3V3 | 3.3V power rail |
| 3 | GPS_TX | Pi GPIO15/RXD (J1 pin 10) |
| 4 | GPS_RX | Pi GPIO14/TXD (J1 pin 8) |

---

## 5. Step-by-Step: Building the Schematic in KiCad 9.0

### 5.1 Create the Project

1. Open **KiCad 9.0**
2. **File → New Project**
3. Name: `FSAE_Display_MainPCB`
4. Choose your project directory
5. KiCad creates `.kicad_pro`, `.kicad_sch`, and `.kicad_pcb` files
6. Repeat for the daughter board: create a second project `FSAE_Display_DaughterBoard`

### 5.2 Open the Schematic Editor

1. Double-click the `.kicad_sch` file (or click the schematic icon in the project manager)
2. The **Schematic Editor (Eeschema)** opens with a blank sheet

### 5.3 Set Up the Sheet

1. **File → Page Settings** → Set paper size to A3 (gives more room)
2. Fill in the title block: Project name, date, revision, your name

### 5.4 Add Power Symbols

1. Press **P** (or **Place → Power Port**) to add power symbols
2. Search for and place these power symbols (one of each, in a clear area):
   - `+12V`
   - `+5V`
   - `+3V3`
   - `GND`
3. These are global power nets — every symbol with the same name connects automatically

### 5.5 Place Components — Power Supply Section

1. Press **A** (Add Symbol) → search `D_Schottky` → place **D1** (reverse polarity diode)
2. Press **A** → search `Fuse` → place **F1**
3. Press **A** → search `MP1584` (or your custom symbol) → place **U3**
4. Press **A** → search `D_Schottky` → place **D2** (freewheeling diode)
5. Press **A** → search `L` → place **L1** (inductor)
6. Press **A** → search `Screw_Terminal_01x02` → place **J5** (power input)
7. Add resistors (**R**) for R_FB1, R_FB2
8. Add capacitors (**C**) for C7, C8, C9, C5, C11

**Wiring the power section:**

9. Press **W** to start wiring
10. Connect in this order:
    - J5 pin 1 → D1 Anode
    - D1 Cathode → F1 pin 1
    - F1 pin 2 → U3 IN, U3 EN, C7 positive
    - U3 SW → L1 pin 1, D2 Cathode, C9 pin 1
    - U3 BST → C9 pin 2
    - L1 pin 2 → +5V net, C8 positive, R_FB1 pin 1
    - U3 FB → R_FB1 pin 2, R_FB2 pin 1
    - R_FB2 pin 2 → GND
    - D2 Anode → GND
    - C7 negative, C8 negative → GND
    - U3 GND, U3 EP → GND

### 5.6 Place Components — CAN Bus Section

1. Press **A** → search `MCP2515` → select `MCP2515-xSO` → place **U1**
2. Press **A** → search `SN65HVD230` → place **U2**
3. Press **A** → search `Crystal` → place **Y1**
4. Press **A** → search `Screw_Terminal_01x02` → place **J4** (CAN terminal, 2-pos)
5. Add **R3** (120Ω), **R4** (10kΩ)
6. Add **JP1** (2-pin jumper header: `Conn_01x02`)
7. Add **C1, C2** (22pF), **C3, C4** (100nF)

**Wiring the CAN section:**

8. Wire U1 per the pin table in §2.3:
   - U1:1 (TXCAN) → U2:1 (D)
   - U1:2 (RXCAN) → U2:4 (R)
   - U1:7 (OSC2) → Y1 pin 2, C2 pin 1
   - U1:8 (OSC1) → Y1 pin 1, C1 pin 1
   - C1 pin 2, C2 pin 2 → GND
   - U1:9 (VSS) → GND
   - U1:17 (~RESET) → R4 pin 2; R4 pin 1 → +3V3
   - U1:18 (VDD) → +3V3; C3 between VDD and GND
   - U1:6 (~TX2RTS) → +3V3 (tie high)
9. Wire U2 per §2.3:
   - U2:3 (VCC) → +3V3; C4 between VCC and GND
   - U2:7 (CANH) → J4:1, one side of R3
   - U2:6 (CANL) → J4:2, other side of R3
   - R3 connects through JP1 (jumper enables/disables termination)
   - U2:8 (Rs) → GND
   - U2:2 → GND
   - J4:3 → GND

### 5.7 Place the Raspberry Pi Header

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

### 5.8 Place GPS & Daughter Board Connectors

1. Press **A** → search `Conn_01x04` → place **J3** (GPS header)
   - Label pin 1: GND, pin 2: +3V3, pin 3: `GPS_TX`, pin 4: `GPS_RX`

2. Press **A** → search `Conn_02x04_Odd_Even` → place **J2** (daughter board)
   - Label pin 1: `LED_DB1`, pin 2: `LED_DB2`
   - Label pin 3: `LED_DB3`, pin 4: `BTN1`
   - Label pin 5: `BTN2`, pin 6: `BTN3`
   - Label pin 7: `+3V3`, pin 8: `GND`

### 5.9 Place On-Board LEDs

1. Place **LED1** (Device:LED) and **R1** (330Ω) for power indicator
2. Wire: +3V3 → R1 → LED1 Anode → LED1 Cathode → GND
3. Place **LED2** (Device:LED) and **R2** (330Ω) for CAN status
4. Wire: +3V3 → LED2 Anode → LED2 Cathode → R2 → U1 pin 11 (~RX0BF)
   (Active-low: LED lights when CAN frame received)

### 5.10 Add Decoupling Capacitors

1. Place 100nF caps near every IC power pin:
   - C3: U1 VDD (pin 18) to GND
   - C4: U2 VCC (pin 3) to GND
   - C5: U3 VCC (pin 8) to GND (if using MP1584EN)
   - C6: General 100nF on +3V3 rail near Pi header

2. Place bulk caps:
   - C10: 10µF on +3V3 rail
   - C11: 10µF on +5V rail

### 5.11 Annotate & Check

1. **Tools → Annotate Schematic** → click **Annotate** → assigns reference designators (U1, R1, C1, etc.)
2. **Inspect → Electrical Rules Check (ERC)** → fix any errors:
   - Unconnected pins: either wire them or mark as "No Connect" (press **Q** and click the pin)
   - Power flag warnings: add a `PWR_FLAG` symbol to +12V_IN and GND nets
3. Mark unused MCP2515 pins (3, 4, 5, 10) with No Connect flags

### 5.12 Assign Footprints

1. **Tools → Assign Footprints** (or click the footprint assignment icon)
2. The Footprint Assignment tool opens showing all components
3. For each component, double-click and select the footprint from the tables in §1.1 and §1.2
4. If a footprint is already assigned (from the symbol's default), verify it matches
5. Click **Apply** then **OK**

### 5.13 Generate Netlist

1. **Tools → Generate Netlist** (or in KiCad 9, this step may be automatic)
2. In KiCad 9.0, you can go directly from schematic to PCB:
   - **Tools → Update PCB from Schematic** (Shortcut: F8)
   - This pushes all components and nets to the PCB editor

---

## 6. Step-by-Step: PCB Layout in KiCad 9.0

### 6.1 Open the PCB Editor

1. From the KiCad project manager, click the **PCB Editor** icon (or double-click `.kicad_pcb`)
2. If you haven't already, press **Tools → Update PCB from Schematic** (F8 in the schematic editor)
3. Click **Update PCB** → all components appear in a cluster outside the board outline

### 6.2 Set Design Rules

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

### 6.3 Draw the Board Outline

1. Select **Edge.Cuts** layer from the layer panel (right side)
2. Use **Place → Line** (or press the line tool) to draw a rectangle:
   - Recommended main PCB size: **85mm × 65mm** (fits above Pi, allows room for all components)
   - The Pi 4 is 85mm × 56mm, so the shield should be roughly the same width
3. For rounded corners, use **Place → Arc** on the Edge.Cuts layer
4. Add four **mounting holes** at corners:
   - Place → Footprint → search `MountingHole_2.7mm_M2.5_Pad`
   - Position at the Pi's mounting hole locations:
     - (3.5, 3.5), (3.5, 52.5), (61.5, 3.5), (61.5, 52.5) mm from bottom-left

### 6.4 Position the 40-Pin Header

1. Find **J1** (the 2×20 female socket) in the component cluster
2. Click and drag it onto the board
3. Position it to match the Raspberry Pi's GPIO header location:
   - The GPIO header on Pi 4 is offset from the board edge
   - Place J1 such that when the shield sits on top, the header aligns with the Pi's pins
   - Typical position: centered along one long edge, header extends toward Pi
4. **Lock the position:** Right-click J1 → Properties → check "Locked"

### 6.5 Component Placement Strategy

Place components in functional groups. Here's the recommended layout:

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   J5 (Power)     F1    D1    U3 (Buck)   C7  C8  L1               │
│   ■■             ■     ■     ■■■■■■      ■   ■   ■                │
│                                                                     │
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
│   │           J1 — 40-Pin Raspberry Pi GPIO Header               │  │
│   │    □□□□□□□□□□□□□□□□□□□□                                      │  │
│   │    □□□□□□□□□□□□□□□□□□□□                                      │  │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│   ┌─────────────────────────────────────┐                          │
│   │      GPS MODULE MOUNTING AREA       │   (J3 + mounting holes)  │
│   │      SparkFun ZED-F9P              │                          │
│   │      43.5mm × 43.2mm              │                          │
│   └─────────────────────────────────────┘                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Placement order:**

1. **J1** (Pi header) — bottom/center, this is the anchor
2. **U3 + power components** — top-left corner, near J5
3. **U1 + U2 + CAN components** — middle area, between power and Pi header
4. **J4** (CAN terminal) — board edge, accessible
5. **J5** (power terminal) — board edge, accessible
6. **J2** (daughter board) — board edge, accessible
7. **J3 + GPS area** — bottom section or separate area with clearance
8. **Decoupling caps** — as close as physically possible to their IC's power pins

### 6.6 Route Critical Traces First

Use **X** to start routing a trace. KiCad 9 has an interactive router.

**Route in this priority order:**

1. **Power traces (+5V, +3V3, GND):**
   - Use 1.0mm trace width for +5V from buck converter to J1
   - Use 0.5mm minimum for +3V3 distribution
   - Consider a ground pour (copper fill) on the back layer — this provides a solid ground plane

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

### 6.7 Add Ground Plane (Copper Pour)

1. Select **B.Cu** (back copper) layer
2. **Place → Add Filled Zone** (or press the zone tool)
3. Click around the board outline to create a zone matching the board shape
4. In the zone properties dialog:
   - Net: `GND`
   - Clearance: 0.3mm
   - Minimum width: 0.25mm
5. Press **B** to re-fill all zones
6. Optionally add a ground pour on **F.Cu** (front copper) in open areas too

### 6.8 Add Silkscreen Labels

1. Select **F.Silkscreen** layer
2. **Place → Text** to add labels:
   - "FSAE Display — Main PCB v1.0" (title)
   - "12V" and "GND" near J5
   - "CAN_H", "CAN_L" near J4 (GND via J5)
   - "GPS" near J3
   - "DAUGHTER BOARD" near J2
   - Component values near each component

### 6.9 Run Design Rule Check (DRC)

1. **Inspect → Design Rules Check**
2. Click **Run DRC**
3. Fix all errors (unconnected nets, clearance violations, etc.)
4. Common issues:
   - **Unrouted nets**: Route any remaining connections
   - **Clearance errors**: Adjust component spacing
   - **Courtyard overlaps**: Move components apart

### 6.10 Generate Gerber Files for Manufacturing

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

## 7. Design Rules & Manufacturing Notes

### 7.1 PCB Specifications

| Parameter | Main PCB | Daughter Board |
|---|---|---|
| Layers | 2 (F.Cu + B.Cu) | 2 (F.Cu + B.Cu) |
| Board thickness | 1.6mm | 1.6mm |
| Copper weight | 1 oz (35µm) | 1 oz (35µm) |
| Min trace width | 0.25mm (10 mil) | 0.25mm (10 mil) |
| Min clearance | 0.2mm (8 mil) | 0.2mm (8 mil) |
| Min drill | 0.3mm | 0.3mm |
| Solder mask | Green (or black for aesthetics) | Green |
| Silkscreen | White | White |
| Surface finish | HASL (cheapest) or ENIG | HASL |

### 7.2 Power Budget

| Rail | Source | Max Current | Consumers |
|---|---|---|---|
| +12V | Car battery | 3A (fuse limited) | Buck converter input |
| +5V | MP1584EN output | 3A max | Raspberry Pi 4 (~2.5A peak), GPS module (~130mA) |
| +3V3 | Pi's onboard regulator | ~800mA | MCP2515 (~5mA), SN65HVD230 (~70mA), LEDs (~12mA), Buttons pullups (~1mA), Daughter board LEDs (~12mA) |

> The +3V3 rail comes from the Raspberry Pi's own 3.3V regulator (accessible
> on GPIO header pins 1 and 17). The Pi regulates 5V down to 3.3V internally.
> Total 3.3V load is well within the Pi's ~800mA 3.3V budget.

### 7.3 Thermal Considerations

- **MP1584EN:** Needs adequate copper area on the exposed pad for heat dissipation.
  Place thermal vias (0.3mm drill, array of 4-6 vias) under the exposed pad
  connecting to the ground plane on the back layer.
- **SN65HVD230:** Low power dissipation, no special thermal management needed.
- **MCP2515:** Low power, no special thermal management needed.

### 7.4 EMC / Signal Integrity Tips

- Keep the crystal (Y1) and its load capacitors (C1, C2) as close as possible to
  U1 pins 7 and 8. Minimize trace length to < 10mm.
- Route CAN_H and CAN_L as a differential pair on the same layer.
- Keep the SPI clock (SCK) trace short to minimize radiation.
- Place all decoupling capacitors within 5mm of their associated IC power pins.
- The ground plane on B.Cu provides a solid return path for all signals.

### 7.5 Assembly Notes

1. **Solder SMD components first** (resistors, caps, ICs) — use solder paste and
   hot air or reflow oven
2. **Solder through-hole components second** (headers, connectors, crystal, LEDs)
3. **Test power section independently:**
   - Apply 12V to J5 before connecting Pi
   - Verify +5V output with multimeter
   - Verify no shorts on any rail
4. **Test CAN section:**
   - Plug onto Pi, enable SPI and MCP2515 driver in `raspi-config`
   - Run `ip link set can0 up type can bitrate 500000`
   - Use `candump can0` to verify messages
5. **Test GPS:**
   - Enable UART in `raspi-config` (disable serial console, enable serial port)
   - Connect GPS breakout to J3
   - Run `cat /dev/ttyS0` or use `gpsd` to verify NMEA sentences

---

## 8. Raspberry Pi 40-Pin Pinout Reference

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

## Appendix A: Daughter Board KiCad Build (Abbreviated)

The daughter board is simpler. Follow the same process as §5 and §6 with these specifics:

1. **Create new project:** `FSAE_Display_DaughterBoard`
2. **Schematic:** Place J6, SW1-3, LED3-5, R5-10, C12-14, and wire per §3.1
3. **Board size:** ~60mm × 40mm (adjust to fit your enclosure face)
4. **Component placement:**
   - Buttons on one side (through enclosure face)
   - LEDs below buttons (through enclosure face)
   - Molex connector on back edge
   - Resistors and caps on back side (behind buttons/LEDs)
5. **Add 4 mounting holes** (M3) at corners for standoffs
6. **Route traces** — simple point-to-point wiring, 0.3mm traces are fine for everything
7. **Ground pour** on back copper layer

---

## Appendix B: KiCad 9.0 Library Quick Reference

Open the **Symbol Chooser** (press A in schematic editor) and type these
exactly to find each part:

| Search Term | Library:Symbol Found |
|---|---|
| `MCP2515` | Interface_CAN_LIN : MCP2515-xSO |
| `SN65HVD230` | Interface_CAN_LIN : SN65HVD230 |
| `Crystal` | Device : Crystal |
| `LED` | Device : LED |
| `SW_Push` | Switch : SW_Push |
| `Conn_02x20` | Connector_Generic : Conn_02x20_Odd_Even |
| `Conn_02x04` | Connector_Generic : Conn_02x04_Odd_Even |
| `Conn_01x04` | Connector_Generic : Conn_01x04 |
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
| `MP1584` | Regulator_Switching : MP1584EN (if available, else see §4.1) |

Open the **Footprint Chooser** (in Assign Footprints dialog) and search:

| Search Term | Library : Footprint |
|---|---|
| `SOIC-18W` | Package_SO : SOIC-18W_7.5x11.6mm_P1.27mm |
| `SOIC-8_3.9` | Package_SO : SOIC-8_3.9x4.9mm_P1.27mm |
| `SOIC-8-1EP` | Package_SO : SOIC-8-1EP_3.9x4.9mm_P1.27mm_EP2.41x3.30mm |
| `Crystal_HC49` | Crystal : Crystal_HC49-4H_Vertical |
| `LED_D3` | LED_THT : LED_D3.0mm |
| `LED_D5` | LED_THT : LED_D5.0mm |
| `SW_PUSH_6mm` | Button_Switch_THT : SW_PUSH_6mm_H5mm |
| `R_0805` | Resistor_SMD : R_0805_2012Metric |
| `C_0805` | Capacitor_SMD : C_0805_2012Metric |
| `C_1210` | Capacitor_SMD : C_1210_3216Metric |
| `PinSocket_2x20` | Connector_PinSocket_2.54mm : PinSocket_2x20_P2.54mm_Vertical |
| `PinHeader_1x02` | Connector_PinHeader_2.54mm : PinHeader_1x02_P2.54mm_Vertical |
| `PinHeader_1x04` | Connector_PinHeader_2.54mm : PinHeader_1x04_P2.54mm_Vertical |
| `Micro-Fit_3.0` | Connector_Molex : Molex_Micro-Fit_3.0_43045-0812_2x04_P3.00mm_Vertical |
| `MKDS-1,5-2` | TerminalBlock_Phoenix : TerminalBlock_Phoenix_MKDS-1,5-2-5.08_1x02_P5.08mm_Horizontal |
| `MKDS-1,5-2` | TerminalBlock_Phoenix : TerminalBlock_Phoenix_MKDS-1,5-2-5.08_1x02_P5.08mm_Horizontal |
| `D_SMA` | Diode_SMD : D_SMA |
| `Fuseholder5x20` | Fuse : Fuseholder5x20_Horizontal |
| `L_Bourns_SRR1260` or `SRR1260` | Inductor_SMD : L_Bourns_SRR1260 |
| `MountingHole_2.7mm` | MountingHole : MountingHole_2.7mm_M2.5_Pad |

---

## Appendix C: Common Pitfalls & Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| Pi won't boot when shield connected | 5V short or wrong pin alignment | Verify 5V and GND pins with meter before inserting Pi |
| No CAN messages | MCP2515 not initialized, wrong SPI wiring | Check SPI wires, enable `dtoverlay=mcp2515-can0,oscillator=8000000,interrupt=25` in `/boot/config.txt` |
| GPS no data | UART not enabled, TX/RX swapped | Enable UART in raspi-config, swap TX↔RX wires |
| Buck converter overheating | Insufficient copper under exposed pad | Add thermal vias, increase copper pour area |
| ERC error: power pin not driven | Missing PWR_FLAG | Add `PWR_FLAG` symbol on +12V input net and GND net |
| Buttons read always LOW | Missing pullup resistors | Verify 10kΩ pullups are connected to +3V3 |
| LEDs always on/off | Reversed polarity | Check anode/cathode orientation; flat side/short leg = cathode |
| CAN termination issues | Jumper not installed or 120Ω missing | Install jumper on JP1; verify with ohmmeter between CAN_H and CAN_L |

---

## Appendix D: Config.txt Overlay for MCP2515

Add to `/boot/config.txt` on the Raspberry Pi:

```
# Enable SPI
dtparam=spi=on

# MCP2515 CAN controller on SPI0.0, 8MHz oscillator, interrupt on GPIO25
dtoverlay=mcp2515-can0,oscillator=8000000,interrupt=25

# Enable UART for GPS (disable Bluetooth on UART, use miniUART for BT)
dtoverlay=miniuart-bt
enable_uart=1
```

After reboot, bring up the CAN interface:

```bash
sudo ip link set can0 up type can bitrate 500000
candump can0
```

For GPS, read NMEA data:

```bash
cat /dev/ttyAMA0
# or use gpsd:
sudo gpsd /dev/ttyAMA0 -F /var/run/gpsd.sock
cgps -s
```

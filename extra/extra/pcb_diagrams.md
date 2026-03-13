# FSAE Display PCB Diagrams

## Main PCB (Pi Shield)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              MAIN PCB (Pi Shield)                           │
│                                                                             │
│   POWER INPUT                    CAN BUS                      CONNECTORS   │
│  ┌─────────────┐              ┌─────────────┐              ┌─────────────┐  │
│  │ 12V   GND   │              │  H   L  GND │              │ Daughter BD │  │
│  │  ◉     ◉    │              │  ◉   ◉   ◉  │              │ LED1  LED2  │  │
│  └──────┬──────┘              └──────┬──────┘              │ LED3  BTN1  │  │
│         │                            │                     │ BTN2  BTN3  │  │
│         ▼                            ▼                     │ GND   3.3V  │  │
│  ┌─────────────┐              ┌─────────────┐              └──────┬──────┘  │
│  │    Buck     │              │  Termination│                     │         │
│  │  Converter  │              │   120Ω      │                     │         │
│  │  12V → 5V   │              │  (jumper to │                     │         │
│  │             │              │   enable)   │                     │         │
│  └──────┬──────┘              └──────┬──────┘                     │         │
│         │ 5V                         │                            │         │
│         ▼                            ▼                            │         │
│  ┌─────────────────────────────────────────────────────┐          │         │
│  │                                                     │          │         │
│  │                      ┌───────────┐                  │          │         │
│  │                      │  MCP2515  │                  │          │         │
│  │                      │   CAN     │                  │          │         │
│  │    8 MHz             │Controller │                  │          │         │
│  │   Crystal ──────────►│           │                  │          │         │
│  │                      │  SPI      │                  │          │         │
│  │                      └─────┬─────┘                  │          │         │
│  │                            │                        │          │         │
│  │                            │ TX/RX                  │          │         │
│  │                            ▼                        │          │         │
│  │                      ┌───────────┐                  │          │         │
│  │                      │SIT65HVD230│                  │          │         │
│  │                      │   CAN     │◄─────────────────┼── CAN Bus│         │
│  │                      │Transceiver│                  │          │         │
│  │                      └───────────┘                  │          │         │
│  │                                                     │          │         │
│  └─────────────────────────────────────────────────────┘          │         │
│                                                                   │         │
│         GPS CONNECTOR              STATUS LEDS                    │         │
│        ┌─────────────┐            ┌─────────┐                     │         │
│        │VCC GND TX RX│            │ PWR CAN │ (active/error)      │         │
│        │ ◉   ◉  ◉  ◉ │            │  ◉   ◉  │                     │         │
│        └──────┬──────┘            └─────────┘                     │         │
│               │                                                   │         │
│               │ UART to Pi GPIO 14/15                             │         │
│               ▼                                                   │         │
│  ┌─────────────────────────────────────────────────────────────┐  │         │
│  │                                                             │  │         │
│  │                     40-Pin GPIO Header                      │◄─┘         │
│  │           (directly plugs onto Raspberry Pi 4)              │            │
│  │                                                             │            │
│  │  3.3V  5V  SDA SCL GP4 GND ... MOSI MISO SCLK CE0 CE1 ...   │            │
│  │   □    □    □   □   □   □       □    □    □    □   □        │            │
│  │   □    □    □   □   □   □       □    □    □    □   □        │            │
│  │                                                             │            │
│  └─────────────────────────────────────────────────────────────┘            │
│                                                                             │
│  ACTIVE PINS:                                                               │
│  ├── SPI: MOSI (GPIO10), MISO (GPIO9), SCLK (GPIO11), CE0 (GPIO8)          │
│  ├── CAN Interrupt: GPIO25                                                  │
│  ├── UART: TX (GPIO14), RX (GPIO15) - GPS                                   │
│  ├── Buttons: GPIO17, GPIO27, GPIO22                                        │
│  ├── LEDs: GPIO23, GPIO24, GPIO5                                            │
│  └── Power: 5V, 3.3V, GND                                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Daughter Board (Button/LED Panel)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DAUGHTER BOARD (Front Panel)                        │
│                                                                             │
│                        Mounts to enclosure face                             │
│                    Buttons and LEDs poke through holes                      │
│                                                                             │
│  FRONT (visible to driver)                                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                                                                     │    │
│  │      ┌─────┐         ┌─────┐         ┌─────┐                        │    │
│  │      │ BTN │         │ BTN │         │ BTN │                        │    │
│  │      │  1  │         │  2  │         │  3  │                        │    │
│  │      │SCRN │         │ REC │         │ UP  │                        │    │
│  │      └─────┘         └─────┘         └─────┘                        │    │
│  │                                                                     │    │
│  │       (●)             (●)             (●)                           │    │
│  │       LED1            LED2            LED3                          │    │
│  │       PWR             CAN             REC                           │    │
│  │      (green)         (blue)          (red)                          │    │
│  │                                                                     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
│  BACK (PCB side)                                                            │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                                                                     │    │
│  │   BTN1 ────────► 10kΩ pullup to 3.3V                                │    │
│  │   BTN2 ────────► 10kΩ pullup to 3.3V                                │    │
│  │   BTN3 ────────► 10kΩ pullup to 3.3V                                │    │
│  │                                                                     │    │
│  │   LED1 ────────► 330Ω resistor                                      │    │
│  │   LED2 ────────► 330Ω resistor                                      │    │
│  │   LED3 ────────► 330Ω resistor                                      │    │
│  │                                                                     │    │
│  │                  ┌─────────────────┐                                │    │
│  │                  │  Molex Micro-Fit│                                │    │
│  │                  │    8-pin        │                                │    │
│  │                  │                 │                                │    │
│  │                  │ 1: LED1 (GPIO23)│                                │    │
│  │                  │ 2: LED2 (GPIO24)│                                │    │
│  │                  │ 3: LED3 (GPIO5) │                                │    │
│  │                  │ 4: BTN1 (GPIO17)│                                │    │
│  │                  │ 5: BTN2 (GPIO27)│                                │    │
│  │                  │ 6: BTN3 (GPIO22)│                                │    │
│  │                  │ 7: 3.3V         │                                │    │
│  │                  │ 8: GND          │                                │    │
│  │                  └────────┬────────┘                                │    │
│  │                           │                                         │    │
│  └───────────────────────────┼─────────────────────────────────────────┘    │
│                              │                                              │
│                              │ Molex harness                                │
│                              │ (to main PCB)                                │
│                              ▼                                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Complete System Assembly

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           ENCLOSURE (side view)                             │
│                                                                             │
│         Enclosure Front                              Enclosure Back         │
│               │                                            │                │
│               │    ┌──────────────────────────┐            │                │
│    ┌──────────┤    │        Daughter          │            │                │
│    │  Display │    │         Board            │            │                │
│    │  (HDMI)  │    │    [BTN] [BTN] [BTN]     │            │                │
│    │          │    │    (LED) (LED) (LED)     │            │                │
│    └──────────┤    └────────────┬─────────────┘            │                │
│               │                 │ Molex                    │                │
│               │                 │ harness                  │                │
│               │    ┌────────────┴─────────────┐            │                │
│               │    │        Main PCB          │            │                │
│               │    │  ┌─────────────────┐     │            │                │
│               │    │  │  Raspberry Pi 4 │     │◄───────────┤ CAN bus        │
│               │    │  └─────────────────┘     │            │ (to car)       │
│               │    │                          │◄───────────┤ 12V power      │
│               │    │  GPS module (or ext.)    │            │ (from car)     │
│               │    └──────────────────────────┘            │                │
│               │                                            │                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Wiring Summary

| Connection | From | To | Cable Type |
|------------|------|-----|------------|
| Main PCB → Pi | 40-pin header | Pi GPIO | Direct plug (no cable) |
| Main PCB → Daughter | Molex 8-pin | Molex 8-pin | Crimped harness |
| Main PCB → GPS | JST 4-pin | GPS module | Crimped harness |
| Main PCB → Car CAN | Screw terminal | Car harness | Twisted pair |
| Main PCB → Car Power | Screw terminal | Car 12V | 18 AWG wire |
| Pi → Display | HDMI | Display | Short HDMI cable |

## Bill of Materials (Main PCB)

| Component | Quantity | Notes |
|-----------|----------|-------|
| MCP2515 | 1 | CAN controller, SOP-18 |
| SIT65HVD230DR | 1 | CAN transceiver, SOP-8 |
| 8 MHz crystal | 1 | For MCP2515 |
| 22pF capacitors | 2 | Crystal load caps |
| MP1584 buck module | 1 | 12V→5V, 3A |
| 120Ω resistor | 1 | CAN termination |
| 2-pin jumper | 1 | Termination enable |
| 40-pin header | 1 | Female, for Pi |
| Molex Micro-Fit 8-pin | 1 | Daughter board connector |
| JST-XH 4-pin | 1 | GPS connector |
| Screw terminals | 2 | CAN (3-pos), Power (2-pos) |
| 100nF capacitors | 4 | Decoupling |
| 10µF capacitors | 2 | Bulk decoupling |
| LEDs (3mm) | 2 | Power, CAN status |
| 330Ω resistors | 2 | LED current limit |

## Bill of Materials (Daughter Board)

| Component | Quantity | Notes |
|-----------|----------|-------|
| Tactile buttons | 3 | 12mm, long actuator |
| LEDs (5mm) | 3 | Green, blue, red |
| 330Ω resistors | 3 | LED current limit |
| 10kΩ resistors | 3 | Button pullups |
| Molex Micro-Fit 8-pin | 1 | Main PCB connector |
| Mounting holes | 4 | M3 for standoffs |

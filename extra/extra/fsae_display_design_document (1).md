# Formula SAE Electric Race Car Driver Display System
## Executive Summary

This project will develop a professional-grade, real-time telemetry display system for the TAMU Formula SAE Electric Racing Team. The system provides drivers with critical vehicle data during races while enabling pit crews to configure displays and analyze performance data remotely.

---

## Goals



### Primary Goals

1. **Real-time telemetry visualization** displaying at minimum: RPM, battery percentage, motor temperature, battery temperature, speed, and lap time
2. **Driver-controlled data logging** via steering wheel buttons with session management
3. **Web-based configuration interface** for pit crew to design screen layouts without code changes
4. **reliable 60 FPS rendering** on embedded hardware under race conditions
5. **Post-race data analysis** through CSV export of logged CAN bus data

### Success Criteria
- System boots and displays telemetry within 30 seconds of car power-on
- Display updates at stable 60 FPS with <50ms latency from CAN message to screen
- Driver can start/stop logging and navigate screens without looking away from track
- Pit crew can modify display layouts and see changes within 5 seconds
- System operates reliably in temperatures from 0°C to 50°C with vehicle vibration

---

## System Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│                         RACE CAR                             │
│                                                              │
│  ┌──────────────┐         ┌─────────────────┐                │
│  │ BMS          │────┐────│  Motor          │                │
│  │(Battery Mgmt)│    │    │  Controller     │                │
│  └──────────────┘    │    └─────────────────┘                │
│                      │                                       │ 
│              ┌───────▼────────┐                              │
│              │   CAN Bus      │                              │
│              │   Network      │                              │
│              └───────┬────────┘                              │
│                      │                                       │
│              ┌───────▼────────────────────────┐              │
│              │   Raspberry Pi 4               │              │
│              │  ┌──────────────────────────┐  │              │
│              │  │  CAN Reader (C)          │  │              │
│              │  │  Graphics Engine (C)     │  │              │
│              │  │  Data Logger (C)         │  │              │
│              │  │  Web Server (Node.js)    │  │              │
│              │  └──────────────────────────┘  │              │
│              │         │            │         │              │
│              └─────────┼────────────┼─────────┘              │
│                        │            │                        │
│              ┌─────────▼──┐    ┌───▼────────┐                │
│              │ 7" Screen  │    │  Ethernet  │────► Pit Laptop│
│              │ (Dashboard)│    │   Port     │                │
│              └────────────┘    └────────────┘                │
│                                                              │
│              ┌─────────────────┐                             │
│              │ Steering Wheel  │                             │
│              │ Buttons (GPIO)  │                             │
│              └─────────────────┘                             │
└──────────────────────────────────────────────────────────────┘
```

---

## Hardware Components

### Core Hardware

| Component | Specification | Est. Cost |
|-----------|---------------|-----------|
| Raspberry Pi 4 | 4GB RAM model | $55 |
| 7" Touchscreen | 800x480, HDMI + USB touch | $70 |
| CAN Hat/Module | MCP2515 or similar | $40 |
| Momentary Buttons | Panel mount, automotive grade (×3) | $15 |
| DC-DC Converter | 12V to 5V, 3A minimum | $20 |
| MicroSD Card | 64GB, high endurance | $15 |
| Enclosure | Custom 3D printed | $30 |
| Wiring & Connectors |  | $35 |
| **Total** | | **~$280** |

### Physical Integration

**Display Mounting:**
- 7" screen mounted on dashboard in driver's line of sight

**Raspberry Pi Housing:**
- Ventilated enclosure mounted behind dashboard or under seat

**Steering Wheel Buttons:**
- Three buttons mounted within thumb reach on steering wheel
- Functions: UP, DOWN, RECORD/SELECT
- Wired to Pi GPIO pins

**CAN Bus Connection:**
- Tap into existing CAN network
- Twisted pair wiring with proper termination resistors
- Shielded cable to prevent electrical noise interference

**Power Supply:**
- Todo

**Network Access:**
- Ethernet jack mounted in accessible location for pit crew

---

## Software Architecture

### Component 1: CAN Bus Reader & Parser (C) --  only WRITES to shared mem

**Responsibilities:**
- Initialize CAN interface using SocketCAN
- Read CAN messages in real-time from BMS and motor controller
- Parse message IDs and decode data fields
- Update shared memory structure with latest telemetry values
- Write raw CAN frames to ring buffer for logging

**Key Technologies:**
- SocketCAN API (Linux kernel CAN interface)
- MCP2515 driver

**Data Flow:**
```
CAN Bus → MCP2515 → SocketCAN → CAN Reader → Shared Memory
                                           ↓
                                     Ring Buffer (for logging)
```

---

### Component 2: Graphics Rendering Engine (C + raylib) -- only READS from shared mem

**Responsibilities:**

- Initialize framebuffer and raylib in headless mode
- Render dashboard at locked 60 FPS
- Read telemetry from shared memory
- Load widget configurations from `/tmp/display_config.json`
- Handle button inputs via GPIO
- Trigger visual/audio warnings when thresholds exceeded
- no need to recompile, continuous while loop rendering each widget with updated telem values

**Widget Types:**

- **Gauge Widget:** Circular RPM/speed gauge with configurable min/max/redline
- **Bar Widget:** Horizontal/vertical bars for battery %, temperature
- **Numeric Widget:** Large numbers for critical values
- **Graph Widget:** Scrolling line graph for trends over time
- **Status Widget:** Icons/text for warnings (low battery, overtemp)

**preliminary configuration format (JSON):**

```json
{
  "screens": [
    {
      "name": "Main",
      "widgets": [
        {
          "type": "gauge",
          "x": 100,
          "y": 100,
          "radius": 80,
          "data_field": "rpm",
          "min": 0,
          "max": 12000,
          "redline": 10000,
          "color": "#00FF00",
          "warning_color": "#FF0000"
        },
        {
          "type": "bar",
          "x": 400,
          "y": 300,
          "width": 300,
          "height": 40,
          "data_field": "battery_percent",
          "color": "#00FF00",
          "warning_threshold": 20
        }
      ]
    }
  ]
}
```

**Rendering Loop Pseudo Code:**

```c
while (running) {
    // Read latest telemetry from shared memory
    telemetry_t data = read_shared_telemetry();
    
    // Check for button inputs
    handle_gpio_buttons();
    
    // Render current screen
    BeginDrawing();
    ClearBackground(BLACK);
    
    for (widget in current_screen.widgets) {
        render_widget(widget, data);
    }
    
    EndDrawing();
    
    // Maintain 60 FPS
    wait_for_next_frame();
}
```

---

### Component 3: Data Logger (C)

**Responsibilities:**
- Monitor GPIO for record button press
- Write ring buffer contents to binary log files when recording
- here, memory isn't a concern... we are using a ring buffer and writing to an SD card! 4gb ram is good
- Manage disk space (delete old logs if storage fills)
- Create session metadata (timestamp, duration, etc.)

**File Format:**
```
Session Directory: /home/pi/logs/2026-01..../
├── metadata.json    (session info: start time, duration, driver)
├── raw_can.bin      (binary CAN frames with timestamps)
└── telemetry.csv    (optional: pre-processed CSV for quick viewing)
```

**preliminary binary Log Format...**
each entry: `[timestamp (8 bytes)][CAN ID (4 bytes)][data length (1 byte)][data (8 bytes)]`

**Recording Behavior:**

- Driver presses RECORD button -> LED indicator turns on, logging starts
- All CAN messages written to file  with timestamp
- Driver presses RECORD again -> logging stops, file finalized
- maybe safety limit that stops recording after some amount of time..?

---

### Component 4: Web Server Backend (Node.js + Express)?

**Responsibilities:**

- Serve React web application
- Provide REST API for configuration management
- handle log file listing and CSV export
- broadcast live telemetry via WebSocket -- idk if this is necessary but its cool 
- Send control commands to C processes via named pipe

**Preliminary api endpoints:**

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/config` | Retrieve current display configuration |
| POST | `/api/config` | Update display configuration |
| GET | `/api/logs` | List all recorded sessions |
| GET | `/api/logs/:id/download` | Download session as CSV |
| DELETE | `/api/logs/:id` | Delete a log session |
| POST | `/api/control/record` | Start/stop recording remotely |
| WS (websocket) | `/ws/telemetry` | Real-time telemetry stream (10 Hz) |

**Communication with C Programs:**
- **Shared Files:**
  - `/tmp/display_config.json` - Widget layouts and settings
  - `/tmp/telemetry.json` - Latest telemetry snapshot (updated by C reader)
- **Named Pipe:**
  - `/tmp/control_pipe` - Commands from web server to C processes (e.g., "START_RECORDING")

**WebSocket Telemetry Stream (again idk if this is necessary):**

```javascript
// Server broadcasts every 100ms
setInterval(() => {
    const telemetry = JSON.parse(fs.readFileSync('/tmp/telemetry.json'));
    wss.clients.forEach(client => {
        client.send(JSON.stringify(telemetry));
    });
}, 100);
```

---

### Component 5: Web Configuration Interface (React)

**Responsibilities:**
- Drag-and-drop screen layout designer
- Widget configuration (type, position, data field, colors, thresholds)
- Session log browser and CSV downloader
- System status monitoring (disk space, recording state)

**Key Features:**

**Layout Designer:**
- Canvas representing 800×480 screen
- widget palette (gauge, bar, number, graph, status)
- drag widgets onto canvas,, configure properties
- live preview using WebSocket telemetry data (maybe)
- Save/load multiple layout presets

**Configuration Panel:**

- Select data field from dropdown (RPM, battery %, temp, etc...)
- Set warning thresholds with color pickers
- configure min/max ranges for gauges
- Font size and style options for numeric displays

**Log Management:**
- Table showing all sessions with: date, duration, file size
- Filter by date range
- Download as CSV (converted from binary format)
- Delete old sessions

**Live Telemetry View:**

- Real-time preview of how widgets look with current car data
- Useful for testing configurations during practice runs

**UI Mockup Structure, thanks AI:**

```
┌─────────────────────────────────────────────────────────┐
│  FSAE Display Configurator          [System Status: OK] │
├──────────────┬──────────────────────────────────────────┤
│              │  ┌────────────────────────────────────┐  │
│  Widget      │  │                                    │  │
│  Palette:    │  │      Canvas (800×480 preview)      │  │
│              │  │                                    │  │
│  [Gauge]     │  │   [Live telemetry renders here]   │  │
│  [Bar]       │  │                                    │  │
│  [Number]    │  │                                    │  │
│  [Graph]     │  └────────────────────────────────────┘  │
│  [Status]    │                                          │
│              │  Widget Properties:                      │
│              │  Type: [Gauge ▼]                        │
│  Screens:    │  Data: [RPM ▼]                          │
│  • Main      │  Color: [🎨]  Warning: [🎨]             │
│  • Detail    │  Min: [0]  Max: [12000]  Red: [10000]   │
│  + Add       │  [Apply Changes]                        │
│              │                                          │
│  Logs:       │  Recent Sessions:                       │
│  [View All]  │  • 2026-01-27 14:30 (45 min) [Download] │
│              │  • 2026-01-26 10:15 (30 min) [Download] │
└──────────────┴──────────────────────────────────────────┘
```

---

## System Startup & Runtime Flow

### Runtime Operation

**Normal Operation:**
```
CAN Reader ──┬──> Shared Memory ──> Graphics Engine ──> Display
             │                             ↑
             └──> Ring Buffer              │
                      ↓                    │
                 Data Logger ──────────────┘
                      ↓
                  Log Files ──> Web Server ──> Pit Laptop
```

**Driver Interactions:**
- **UP/DOWN Buttons:** Navigate between configured screens
- **RECORD Button:** 
  - Short press: Start/stop logging current session
  - Long press (3s): Switch to detailed telemetry screen?

**Pit Crew Workflow:**
1. Connect laptop to Pi via Ethernet
2. Open browser to Pi's IP
3. Modify layout in configurator
4. click on "Apply Changes" -> new config saves to `/tmp/display_config.json`
5. Graphics C engine detects file change and reloads
6. Driver sees updated layout immediately!

---

## Data Specifications

### CAN Bus Protocol

**Expected CAN Messages:** (note, these aren't the real frames... i need to get them. just to give some idea of what these frames look like)

| CAN ID | Source | Data Fields |
|--------|--------|-------------|
| 0x101 | Motor Controller | RPM (2 bytes), Torque (2 bytes) |
| 0x102 | Motor Controller | Motor Temp (2 bytes), Controller Temp (2 bytes) |
| 0x200 | BMS | Battery Voltage (2 bytes), Current (2 bytes) |
| 0x201 | BMS | Battery % (1 byte), Max Cell Temp (2 bytes) |
| 0x202 | BMS | Min Cell Voltage (2 bytes), Max Cell Voltage (2 bytes) |
| 0x300 | Wheel Speed Sensors | Speed (2 bytes), Distance (4 bytes) |

**Parsing Example (RPM from Motor Controller):**
```c
// CAN ID 0x101: [RPM_HIGH][RPM_LOW][TRQ_HIGH][TRQ_LOW][0][0][0][0]
void parse_motor_data(uint8_t *data) {
    uint16_t rpm = (data[0] << 8) | data[1];
    uint16_t torque = (data[2] << 8) | data[3];
    
    telemetry.rpm = rpm;
    telemetry.torque = torque * 0.1; // Scale factor
}
```

### Telemetry Data Structure (Shared Memory)

```c
typedef struct {
    // Motor
    uint16_t rpm;
    float torque_nm;
    float motor_temp_c;
    float controller_temp_c;
    
    // Battery
    float battery_voltage;
    float battery_current;
    uint8_t battery_percent;
    float max_cell_temp_c;
    float min_cell_voltage;
    float max_cell_voltage;
    
    // Vehicle
    float speed_kmh;
    uint32_t distance_m;
    
    // Status flags
    bool bms_fault;
    bool motor_fault;
    bool overtemp_warning;
    
    // Metadata
    uint64_t timestamp_us;
} telemetry_t;
```

---

## Stretch Goals

### Tier 1 (Medium Priority)

1. **Configurable Alerts **-- i like this
   - Audio warnings (buzzer) for critical thresholds
   - Haptic feedback (steering wheel vibration) for warnings
   - Customizable alert priorities
2. **Multi-Screen Profiles**
   - Driver-specific layout presets (e.g., "Rookie Mode" vs "Race Mode")
   - Quick-switch between profiles via button combo
   - Cloud sync of favorite layouts

### Tier 2 (Advanced Features)

5. **Integration with Telemetry**
   - hook up our pi over uart to the cellular modem
   - Currently, this is done on an STM32 microcontroller. it just gets every single can frame,
   - and sends it over MQTT using the modem. we would replace this flow
6. **Machine Learning Insights**
   - Detect anomalous patterns (e.g., unusual battery drain)
   - Predict component failures based on temperature trends
   - Suggest optimal shift points based on track conditions



---


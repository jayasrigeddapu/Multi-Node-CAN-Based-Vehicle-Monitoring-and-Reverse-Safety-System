
::: {align="center"} # 🚗 CAN-Based Vehicle Monitoring & Reverse Safety System **Three LPC2129 nodes · One CAN bus · Distributed vehicle monitoring** A modular Embedded C prototype integrating temperature monitoring, turn indicators, vehicle-mode switching, and reverse-obstacle alerts. `<img src="images/BLOCKDIAGRAM_major.jpeg" alt="CAN-based vehicle monitoring system block diagram" width="82%">`{=html} ```{=html} <p> ``` `<img src="https://img.shields.io/badge/Platform-ARM7%20LPC2129-blue" alt="Platform ARM7 LPC2129">`{=html} `<img src="https://img.shields.io/badge/Language-Embedded%20C-informational" alt="Language Embedded C">`{=html} `<img src="https://img.shields.io/badge/Communication-CAN-orange" alt="Communication CAN">`{=html} `<img src="https://img.shields.io/badge/Status-Educational%20Prototype-yellow" alt="Educational prototype">`{=html} ```{=html} </p> ``` :::
!
N
O
T
E
!NOTE Educational prototype only. This project is not
production automotive safety equipment and is not automotive-certified
or safety-validated.

📑 Table of Contents
At a Glance
Features
System Architecture
CAN Protocol
Node Breakdown
Hardware and Tools
Pin Mapping
Sensors in Detail
Operating Sequence
Repository Layout
Build and Flash
Prototype Gallery
Testing Checklist
Limitations and Safety Notes
Roadmap
Conclusion
⚡ At a Glance
Parameter Details

Microcontrollers 3 × ARM7 LPC2129
Nodes Main Node, Indicator Node, Reverse Alert Node
CAN bit rate 125 kbit/s (Common/can_defines.h)
CAN transceiver MCP2551 or compatible --- one per node
Temperature sensor DS18B20 (1-Wire), Main Node
Distance sensor HC-SR05-compatible ultrasonic module
Obstacle threshold 20 cm software threshold; 3-sample average
CAN identifiers 0x101, 0x102, 0x201
Firmware language Embedded C
Development environment Keil µVision with compatible ARM toolchain

Node responsibilities
Node Responsibility

🧠 Main Node Reads temperature, updates LCD,
handles mode/left/right switches,
sends CAN commands, and drives the
reverse-warning buzzer.

💡 Indicator Node Receives left/right/off commands
and runs an eight-LED sweep.

📡 Reverse Alert Node Measures distance in reverse mode
and reports distance and obstacle
status over CAN.
✨ Features
🌡️ Engine-temperature monitoring using the DS18B20 digital sensor
🔗 CAN communication between independent embedded nodes
⏱️ Interrupt-driven left, right, and mode switches
🔄 Forward/reverse mode toggling
📏 Ultrasonic obstacle detection while in reverse
🖥️ LCD status display with buzzer and LED warning outputs
🚦 Eight-LED indicator animation
🧩 Modular Embedded C drivers for CAN, LCD, DS18B20, ultrasonic
sensing, interrupts, and buzzer control
🏗️ System Architecture
CAN bus topology
                    Shared CAN Bus (CANH / CANL)
       ┌──────────────────────┬──────────────────────┐
       │                      │                      │
┌──────▼─────────┐    ┌───────▼────────┐    ┌────────▼─────────┐
│   MAIN NODE    │    │ INDICATOR NODE │    │ REVERSE ALERT    │
│   LPC2129      │    │   LPC2129      │    │ NODE - LPC2129   │
│                │    │                │    │                  │
│ DS18B20        │    │ 8 Indicator    │    │ HC-SR05-compatible│
│ LCD            │    │ LEDs            │    │ Ultrasonic Sensor │
│ Switches       │    │                │    │                  │
│ Buzzer/Alert   │    │                │    │                  │
└────────────────┘    └────────────────┘    └──────────────────┘
Each LPC2129 on-chip CAN controller connects to the bus through a CAN
transceiver such as the MCP2551.

!
I
M
P
O
R
T
A
N
T
!IMPORTANT Terminate the CAN bus at its two physical ends
only. Intermediate nodes should not each have a 120 Ω termination
resistor.

Visual guide
<table>
<tr>
<td width="50%" align="center">
<b>{=html}Physical CAN bus and node
interfaces`</b>`{=html}`
`{=html}`
`{=html}
<img src="images/interface.png" alt="Physical CAN bus and node interfaces" width="100%">{=html}

</td>
<td width="50%" align="center">
<b>{=html}Development and integration
workflow`</b>`{=html}`
`{=html}`
`{=html}
<img src="images/sequential.png" alt="Development and integration workflow" width="100%">{=html}

</td>
</tr>
<tr>
<td width="50%" align="center">
<b>{=html}Reverse-alert node
concept`</b>`{=html}`
`{=html}`
`{=html}
<img src="images/reverse.png" alt="Reverse-alert node concept" width="100%">{=html}

</td>
<td width="50%" align="center">
<b>{=html}Indicator-node
concept`</b>`{=html}`
`{=html}`
`{=html}
<img src="images/indicator.png" alt="Indicator-node concept" width="100%">{=html}

</td>
</tr>
<tr>
<td colspan="2" align="center">
<b>{=html}Main-node software
flow`</b>`{=html}`
`{=html}`
`{=html}
<img src="images/master.png" alt="Main-node software flow" width="65%">{=html}

</td>
</tr>
</table>
The workflow diagram represents the intended development stages: LCD
verification → DS18B20 1-Wire bring-up → interrupt/LCD validation →
ultrasonic range testing → CAN loopback/communication testing →
three-node integration. It is a plan, not evidence that every stage
passed.

The Reverse Alert diagram illustrates echo measurement, distance
calculation, threshold comparison, and CAN reporting. The Indicator
diagram illustrates command-to-LED animation mapping. The Main Node
diagram illustrates initialization, monitoring, switch-event handling,
CAN transmission, and received-status checks.

!
W
A
R
N
I
N
G
!WARNING These diagrams are explanatory. If labels differ from the
actual source or hardware (for example, HC-SR04 versus an
HC-SR05-compatible module), trust the board markings, datasheets, pin
configuration, and compiled firmware. Diagrams do not verify wiring or
test results.

📨 CAN Protocol
All three message types use standard CAN identifiers. An identifier
names a message type, not a physical node address. Each node processes
the identifiers relevant to it.

Message definitions
CAN ID Direction Data 1 Data 2 Meaning

0x101 Main → ASCII L, Unused Left, right,
Indicator R, or S or indicators
off

0x102 Main → Reverse 0 = forward, Unused Current
Alert 1 = reverse vehicle mode

0x201 Reverse Alert Distance (cm) 0 = clear, Distance and
→ Main 1 = obstacle alert status
Message triggers
CAN ID Trigger Receiver action

0x101 Left/right switch event Updates LED pattern
or stop command

0x102 Mode switch change Enables or disables
reverse measurements

0x201 End of measurement Updates reverse-warning
cycle decision
Payload notes
Data1 and Data2 are single bytes (0--255).
If the protocol is extended, define explicit scaling and codes for
invalid, timeout, and stale data.
A timeout or missing frame must not be interpreted as a measured
distance of 0 cm.
Temperature is handled locally on the Main Node. No CAN temperature
message exists in the current source.
All nodes must agree on bit rate (125 kbit/s), frame format, ID
meaning, and payload layout.
🧩 Node Breakdown
<details>
<summary>
<b>{=html}🧠 Node 1 --- Main Node`</b>`{=html} (central monitoring and
command)

</summary>
Initializes the LCD, DS18B20 interface, CAN controller, buzzer
output, and external interrupts.
Reads the DS18B20 over 1-Wire and displays temperature on the LCD.
Uses three falling-edge external interrupts: mode, left indicator,
and right indicator.
Mode switch toggles 0 = Forward / 1 = Reverse and sends the
state in CAN ID 0x102.
Indicator events are encoded as L, R, or S and sent in CAN ID
0x101.
Receives CAN ID 0x201: distance in Data 1 and alert flag in Data
In reverse mode, activates the warning output when the alert flag
reports an obstacle within range.
ISR guideline: ISRs should only record the event and set a flag.
CAN transmission and LCD updates belong in the main loop.
</details>
<details>
<summary>
<b>{=html}💡 Node 2 --- Indicator Node`</b>`{=html} (LED sequencing
from CAN commands)

</summary>
Initializes CAN1 and configures P0.0--P0.7 as LED outputs.
Polls CAN receive status and reads available frames.
Processes only CAN ID 0x101.
Reads Data 1 as L, R, or S.
L: sweeps LEDs 0 → 7.
R: sweeps LEDs 7 → 0.
S: turns all LEDs off.
Source logic assumes active-low LEDs (low = on). Confirm this
against the board wiring.
</details>
<details>
<summary>
<b>{=html}📡 Node 3 --- Reverse Alert Node`</b>`{=html} (distance
measurement and alert reporting)

</summary>
Initializes CAN1 and the ultrasonic Trigger/Echo pins.
Listens for CAN ID 0x102.
Measures only when vehicle mode is reverse (1).
Averages three readings to reduce unstable samples.
Compares the average against the 20 cm threshold: below 20 cm →
alert 1; otherwise alert 0.
Sends CAN ID 0x201 with distance in Data 1 and alert in Data 2.
Waits approximately 100 ms, then repeats.
Driver uses a timeout so a missing Echo does not block forever.
Treat timeout as an invalid measurement and confirm the application
handles the returned value safely.
</details>
🔧 Hardware and Tools
Component Quantity Role

LPC2129 3 Main, Indicator, and
Reverse Alert nodes

MCP2551 or compatible 1 per node CAN controller ↔
transceiver CANH/CANL interface

DS18B20 1 Main Node temperature
sensing

HC-SR05-compatible 1 Reverse distance
ultrasonic module measurement

Character LCD 1 Main Node display

LEDs 8 + warning LED(s) Indicator animation
and warning

Buzzer 1 Audible reverse
warning

Switches 3 Mode, left, and right

USB-to-UART adapter / As needed Flashing and
programmer debugging

CAN wiring and As needed Shared CAN bus
termination resistors
Software: Embedded C, Keil µVision with a compatible ARM toolchain,
Flash Magic (if the board bootloader supports it), LPC21xx headers, and
LPC2129 startup/linker configuration.

📍 Pin Mapping
!
W
A
R
N
I
N
G
!WARNING Pin maps below are derived from GPIO/peripheral
definitions in the source, not from a verified schematic. Check them
against the LPC2129 package, board schematic, and pin-multiplexing
table before powering up.

<details>
<summary>
<b>{=html}Main Node`</b>`{=html}

</summary>
Function Pin Direction Notes

LCD D0--D7 P0.8--P0.15 Output 8-bit data bus

LCD RS P0.16 Output Command (0) /
data (1)

LCD EN P0.17 Output Enable strobe

LCD RW P0.18 Output Low = write; tie
low in hardware
if LCD is never
read and driver
allows

DS18B20 DQ P0.19 Bidirectional 1-Wire; requires
pull-up

Buzzer P0.20 Output Initialized low,
low = active;
verify polarity
and drive circuit

Mode switch P0.1 / EINT0 Input / IRQ Falling edge

Left switch P0.3 / EINT1 Input / IRQ Falling edge

Right switch P0.7 / EINT2 Input / IRQ Falling edge

CAN1 RX P0.25 / RD1 Peripheral input From transceiver
RXD

CAN1 TX P0.24 / TD1 Peripheral output Verify PINSEL
configuration in
driver and
datasheet
</details>
<details>
<summary>
<b>{=html}Indicator Node`</b>`{=html}

</summary>
Function Pin Direction Notes

LED 0--7 P0.0--P0.7 Output Active-low per
source logic

CAN1 RX P0.25 / RD1 Peripheral input Via transceiver

CAN1 TX P0.24 / TD1 Peripheral output Verify peripheral
selection
</details>
<details>
<summary>
<b>{=html}Reverse Alert Node`</b>`{=html}

</summary>
Function Pin Direction Notes

Ultrasonic Trigger P0.20 Output Trigger pulse to sensor
Ultrasonic Echo P0.19 Input Echo pulse width measured
CAN1 RX P0.25 / RD1 Peripheral input Receives mode messages
CAN1 TX P0.24 / TD1 Peripheral output Sends distance/alert

</details>
CAN bus wiring
LPC2129 CAN TX → transceiver TXD.
Transceiver RXD → LPC2129 CAN RX.
Connect all CANH lines together and all CANL lines together.
Provide a common ground between nodes.
Terminate at the two bus ends only.
Follow the transceiver manufacturer's supply and decoupling
guidance.
!
I
M
P
O
R
T
A
N
T
!IMPORTANT The source explicitly configures CAN1 RX on P0.25.
Inspect the CAN driver and LPC2129 datasheet to confirm CAN1 TX
(P0.24) is also selected. A comment alone does not prove that the pin
mux is configured.

🔬 Sensors in Detail
🌡️ DS18B20 --- Engine temperature
The DS18B20 is a digital sensor using a single 1-Wire data line; no ADC
conversion is required.

Read sequence

Controller issues a reset pulse.
Sensor responds with a presence pulse.
Controller sends the ROM command (single device) and
convert-temperature command.
After conversion time, the scratchpad temperature registers are
read.
Firmware converts the signed raw value into a temperature for the
LCD.
Wiring

VDD → supply allowed by the datasheet and board.
GND → common ground.
DQ → Main Node P0.19, with a pull-up (commonly approximately 4.7
kΩ) to logic supply. Verify based on sensor, cable length, and
board.
Keep the data line within safe GPIO levels. Do not pull a 3.3 V MCU
input up to 5 V unless the pin is documented as 5 V tolerant.
📏 HC-SR05-compatible ultrasonic sensor --- Reverse distance
Driver pulls Trigger low briefly, then sends a high pulse of
approximately 10 µs.
Module emits an ultrasonic burst and holds Echo high for the
round-trip time.
Timer0 measures the Echo pulse width.
Distance is estimated using:
distance_cm ≈ echo_time_us / 58
This assumes room-temperature sound speed and typical HC-SR04-style
timing. Verify against the module datasheet and timer calibration.

Configuration: Trigger P0.20, Echo P0.19 on the Reverse Alert Node.
Alert threshold: 3-sample average below 20 cm. This is a software
constant, not a universal safe stopping distance.

!
C
A
U
T
I
O
N
!CAUTION Many HC-SR04-style modules drive Echo at approximately 5
V. Check the exact module output level and LPC2129 input tolerance.
Use a level shifter or resistor divider if required; never exceed the
pin's absolute maximum rating.

🔁 Operating Sequence
This describes intended behavior documented from the source. Exact
timing depends on clock and timer configuration.

Stage Operation

1. Power-on Configure clock and pin functions →
initialize LCD and outputs →
initialize CAN at 125 kbit/s →
configure Main Node interrupts →
initialize node-specific sensors

2. Main loop Request/read DS18B20 → convert and
display temperature (avoid
full-screen clear/repaint) →
process recorded switch events →
transmit CAN commands on state
change → check received
reverse-alert frames

3. Indicator Node Receive frames → ignore IDs other
than 0x101 → decode L/R/S → run
sweep or turn LEDs off

4. Reverse Alert Node Receive 0x102 → if reverse, take
readings → average three → compare
with 20 cm → send 0x201

5. Main Node alert Accept 0x201 → confirm reverse
mode and alert flag → drive buzzer
according to polarity and circuit
ISR rule: Acknowledge the event, set a flag, and return. Keep LCD
updates, delays, ultrasonic timing, and CAN processing out of ISRs.
Verify the actual code follows this before claiming it.

📂 Repository Layout
CAN-Based-Vehicle-Monitoring-System/
├── README.md
├── images/                         # Diagrams and prototype photos
├── Common/
│   ├── can.c / can.h
│   ├── can_defines.h
│   ├── delay.c / delay.h
│   ├── defines.h
│   └── types.h
├── Main_Node/
│   ├── main_node.c
│   ├── interrupt.c
│   ├── buzzer.c / buzzer.h
│   ├── lcd.c / lcd.h / lcd_defines.h
│   └── ds18b20.c / ds18b20.h
├── Indicator_Node/
│   └── indicator_node.c
├── Reverse_Alert_Node/
│   ├── reverse_node.c
│   └── ultrasonic.c
├── Hardware/
│   └── BLOCKDIAGRAM.jpeg           # Add actual diagram
└── Documentation/                  # Test logs, screenshots, wiring notes
🛠️ Build and Flash
The repository contains Embedded C sources but does not include a
complete Keil project/workspace or confirmed linker/startup
configuration. Build each node as a separate firmware image.

Create a Keil µVision project for the exact LPC2129 device.

Add the sources for each node:

Firmware image Sources

Main Node main_node.c, LCD, DS18B20,
interrupt, buzzer, plus common
CAN/delay

Indicator Node indicator_node.c, CAN driver,
delay

Reverse Alert Node reverse_node.c, ultrasonic.c,
CAN driver
Check startup file, clock configuration, memory layout, compiler
version, and CAN pin multiplexing.

Compile and clear all warnings/errors. Ensure sources from one node
are not accidentally included in another build.

Flash each board using its supported method (Flash Magic only if
bootloader and serial wiring allow).

Test each node individually, then connect the CAN bus and test the
complete system.

Do not describe the firmware as build-verified until all three images
compile using the actual toolchain.

📸 Prototype Gallery
The diagrams explain the design; the photos below are intended to show
the assembled boards and LCD states during demonstration.

<table>
<tr>
<td align="center" width="50%">
<b>{=html}1. Startup screen`</b>`{=html}`
`{=html}
<sub>{=html}Project title on the
LCD`</sub>`{=html}`
`{=html}`
`{=html}
<img src="images/output-title-screen.jpeg" alt="LCD startup screen showing project title" width="100%">{=html}

</td>
<td align="center" width="50%">
<b>{=html}2. Complete hardware setup`</b>`{=html}`
`{=html}
<sub>{=html}LPC2129 boards, jumper wires, LCD, and ultrasonic
sensor`</sub>`{=html}`
`{=html}`
`{=html}
<img src="images/output-hardware-setup.jpeg" alt="Complete hardware prototype" width="100%">{=html}

</td>
</tr>
<tr>
<td align="center">
<b>{=html}3. Forward mode`</b>`{=html}`
`{=html}
<sub>{=html}Temperature, indicator status, and forward
state`</sub>`{=html}`
`{=html}`
`{=html}
<img src="images/output-forward-mode.jpeg" alt="Forward driving mode displayed on LCD" width="100%">{=html}

</td>
<td align="center">
<b>{=html}4. Reverse mode and obstacle alert`</b>`{=html}`
`{=html}
<sub>{=html}Reverse state with distance/alert
status`</sub>`{=html}`
`{=html}`
`{=html}
<img src="images/output-reverse-alert.jpeg" alt="Reverse mode and distance alert output" width="100%">{=html}

</td>
</tr>
<tr>
<td align="center">
<b>{=html}5. Left indicator`</b>`{=html}`
`{=html}
<sub>{=html}Left-indicator hardware
demonstration`</sub>`{=html}`
`{=html}`
`{=html}
<img src="images/output-left-indicator.jpeg" alt="Left-indicator demonstration" width="100%">{=html}

</td>
<td align="center">
<b>{=html}6. Right indicator`</b>`{=html}`
`{=html}
<sub>{=html}Right-indicator hardware
demonstration`</sub>`{=html}`
`{=html}`
`{=html}
<img src="images/output-right-indicator.jpeg" alt="Right-indicator demonstration" width="100%">{=html}

</td>
</tr>
</table>
What the photos show
LCD displays the project title and operating information.
Temperature appears in °C, with readings around 38--40 °C.
Forward and reverse states are displayed.
Indicator states are demonstrated on the physical setup.
Reverse mode displays distance or alert status from the ultrasonic
path.
!
N
O
T
E
!NOTE These are qualitative visual observations, not calibrated
measurements. They do not prove sensor accuracy, error-free CAN
operation, response time, or automotive safety. For quantitative
claims, log known target distances, displayed distances, CAN
reception, alert on/off points, and failures in a results table.
Clearly label simulation results as simulation and hardware results as
hardware.

✅ Testing Checklist
Use this checklist during system validation. Mark items only after they
have been tested.

LCD initialization and string/character display
DS18B20 presence detection and reading checked against a
reference
Mode switch toggles forward/reverse
Left/right interrupts send expected CAN commands
Indicator Node receives 0x101 and sweeps in the correct
direction
Reverse Node receives 0x102 and measures only in reverse
Distances tested on both sides of the 20 cm threshold
Main Node receives 0x201 and drives the warning correctly
CAN wiring, termination, common ground, bit rate, and error
conditions checked
Missing/invalid Echo does not produce a false close-obstacle
alert
Behavior verified after power cycling and with sensors
disconnected
Add photos, LCD captures, measured results, and a demo video only from
working hardware. Do not use placeholder images as proof of completed
tests.

⚠️ Limitations and Safety Notes
Not suitable as the sole safety system in a real vehicle.
A fixed distance threshold ignores vehicle speed, braking distance,
sensor blind zones, and environment.
Ultrasonic readings can be unreliable on angled, soft, narrow, or
irregular obstacles.
Add handling for sensor failure, stale CAN data, bus-off/error
states, switch bounce, and timeouts before claiming stronger
reliability.
Verify pin mappings, voltage levels, pull-ups, transceiver wiring,
and termination against real hardware and datasheets.
Byte-sized payloads limit transmitted distance range; define scaling
and validity codes when extending the protocol.
🗺️ Roadmap
CAN timeout and error-state handling with a safe response to
stale sensor data
Sensor-fault detection and explicit invalid-measurement status
Improved switch debounce and indicator state handling
Temperature threshold warnings and, if needed, a documented CAN
temperature message
CAN message database/specification and host-side automated test
tool
System-level testing, timing measurements, and hardware wiring
diagram
Data logging or IoT monitoring as a separate extension
🏁 Conclusion
This project demonstrates modular Embedded C on the LPC2129 and
practical integration of CAN messaging, digital temperature sensing,
external interrupts, LCD output, LED control, and ultrasonic ranging.
Its three-node architecture provides a learning platform for
understanding how distributed controllers exchange commands and sensor
status over a shared bus.


# tec-APG
TEC-1 programmable avalanche pulse generator

Danger HV

Combine the programmability of a Z80 single-board computer (SBC) with an avalanche transistor circuit to generate adjustable high-voltage pulses, ideal for pulse electronics experiments and retro-tech applications. By integrating optocouplers for safety and utilizing the Z80's CTC timers for precise control, this setup enables dynamic pulse shaping with software-driven adjustments. Rigorous oscilloscope testing ensures reliability, making it a powerful tool for exploring high-speed switching and transient phenomena. ⚡

---

### **System Overview**
- **Goal**: Generate fast, high-voltage pulses (10–200 V, ~ns–µs duration) using avalanche transistor circuits, controlled by a Z80 SBC for adjustable timing, frequency, and pulse width.
- **Key Components**:
  1. **Z80 SBC**: For programmable timing and control logic.
  2. **Avalanche Transistor Circuit**: Generates high-voltage pulses.
  3. **Interface Circuitry**: Bridges the Z80’s 5V logic to high-voltage triggering.
  4. **Power Supplies**: +5V for the Z80 and isolated high-voltage DC (e.g., 100–200V).

---

### **Step 1: Avalanche Pulse Circuit Design**
#### **Core Components**:
- **Avalanche Transistor**: Use a fast-switching NPN transistor (e.g., 2N3904, PN2222, or ZTX415) in avalanche mode.
- **High-Voltage Supply**: 100–200V DC (isolated from the Z80 for safety).
- **Energy Storage**: Capacitor bank (e.g., 100pF–10nF ceramic capacitors) charged by the HV supply.
- **Trigger Circuit**: Optocoupler or MOSFET driver to isolate the Z80 from HV.

#### **Circuit Schematic**:
```
      +HV (100–200V)
        |
        R_charge (10k–100kΩ)
        |
        C_storage (100pF–10nF)
        |
Avalanche Transistor (Collector)
        |
Emitter ── Ground
        |
Trigger ── Optocoupler/MOSFET Driver ── Z80 GPIO
```
- **Operation**:
  1. **Charge Phase**: The capacitor charges through `R_charge`.
  2. **Discharge Phase**: The Z80 sends a 5V trigger pulse to the optocoupler/MOSFET driver, saturating the transistor and causing avalanche breakdown, rapidly discharging the capacitor.

---

### **Step 2: Z80 SBC Interface**
#### **Hardware Setup**:
1. **GPIO Trigger Output**: Connect a Z80 GPIO pin (e.g., via a parallel port) to the trigger circuit.
   - Use a **MOSFET driver** (e.g., TC4420) or **optocoupler** (e.g., 4N35) to isolate the Z80 from HV.
2. **Timing Control**: Use the Z80’s internal timers or CTC (Counter/Timer Circuit) for precise pulse timing.
3. **User Input**: Add buttons/DIP switches or a keypad to adjust parameters (frequency, pulse width).

#### **Example Z80 Code (Assembly)**:
```asm
; Configure CTC for pulse timing
ORG 0000h
    LD A, 0x47       ; CTC control word: Timer mode, prescaler=16
    OUT (0x01), A    ; Write to CTC channel 1
    LD A, 0xFF       ; Time constant (adjust for frequency)
    OUT (0x01), A

; Main loop to trigger pulses
LOOP:
    LD A, 0x01       ; Set GPIO pin high
    OUT (0x02), A
    CALL DELAY       ; Adjust delay for pulse width
    LD A, 0x00       ; Set GPIO pin low
    OUT (0x02), A
    JP LOOP

DELAY:
    LD B, 0xFF       ; Adjust B for pulse width
DELAY_LOOP:
    DJNZ DELAY_LOOP
    RET
```

---

### **Step 3: Safety & Isolation**
1. **Optical Isolation**: Use optocouplers to separate the Z80’s 5V logic from the HV circuit.
2. **Ground Separation**: Keep the Z80 and HV grounds isolated to prevent noise/backlash.
3. **Enclosure**: House the HV circuit in an insulated, shielded box.

---

### **Step 4: Power Supplies**
1. **Z80 Power**: Standard 5V DC (e.g., USB or wall adapter).
2. **HV Supply**:
   - Use a **flyback converter** or **Cockcroft-Walton multiplier** to generate 100–200V DC.
   - Add a current-limiting resistor (1–10kΩ) to protect the transistor.

---

### **Step 5: Testing & Calibration**
1. **Oscilloscope**: Measure pulse width, amplitude, and rise time (aim for <10ns rise time).
2. **Adjustments**:
   - Tune the Z80’s CTC time constants for desired frequency.
   - Modify `C_storage` and `R_charge` to adjust pulse energy.
3. **Safety Check**: Verify isolation and grounding before powering on.

---

### **Applications**
- **Laser/LED Drivers**: Short pulses for time-resolved spectroscopy.
- **Radar/Particle Detectors**: Fast triggering signals.
- **Retro-Computing Projects**: Syncing analog hardware to a Z80.

---

### **Challenges & Solutions**
- **Z80 Speed Limitation**:
  - Use hardware timers (CTC) instead of software delays for sub-µs precision.
  - Add external logic (e.g., 555 timer) for ultrafast pulses.
- **HV Stability**:
  - Use low-ESR capacitors and fast-recovery diodes in the HV supply.
- **Noise**:
  - Add decoupling capacitors and ferrite beads to the Z80’s power lines.

---

### **Final Assembly**
1. Solder the avalanche circuit on a protoboard.
2. Connect the Z80 GPIO to the trigger via optocoupler/MOSFET.
3. Program the Z80 with timing code.
4. Test with a dummy load (e.g., 50Ω resistor) before connecting sensitive devices.

---


# functional
 
```plaintext
+------------------------------------------------------------+
|                        Z80 SBC                             |
|  +-------------------------------+                        |
|  | CTC Timer (0x03, 0x04)        |   +------------------+ |
|  | - Controls pulse timing       |   | Digital Potentiometer |
|  | - Triggers pulse event        |   | (MCP4131, 8-bit SPI)  |
|  +-------------------------------+   | - Adjusts frequency  |
|  | SPI Interface (0x05, 0x06, 0x07) | - Adjusts pulse width |
|  | - Sends commands to digipot     |   +------------------+ |
|  +-------------------------------+                        |
|  | GPIO Output (0x02)            |   +--------------------+ |
|  | - Sends trigger pulse         |   |  555 Timer One-Shot |
|  +-------------------------------+   |  - Generates short  |
|                                       |    trigger pulse    |
|                                       +--------------------+ |
+------------------------------------------------------------+
            │
            │
            ▼
+------------------------------------------------------------+
|                    555 Timer One-Shot Circuit              |
|  - Converts GPIO trigger into a precise short pulse        |
|  - Pulse width determined by R & C values                 |
|  - Prevents false triggering                              |
+------------------------------------------------------------+
            │
            │
            ▼
+------------------------------------------------------------+
|                      Optocoupler (4N35)                   |
|  - Electrically isolates Z80 from high-voltage section     |
|  - Translates low-voltage pulse to HV trigger signal       |
+------------------------------------------------------------+
            │
            │
            ▼
+------------------------------------------------------------+
|                     MOSFET Driver (TC4420)                |
|  - Boosts trigger signal from optocoupler                 |
|  - Ensures fast switching to drive avalanche transistor    |
+------------------------------------------------------------+
            │
            │
            ▼
+------------------------------------------------------------+
|                 Avalanche Transistor Circuit              |
|  - Uses a high-speed NPN transistor (ZTX415)              |
|  - C_storage charges via R_charge from HV supply          |
|  - When triggered, rapid breakdown generates HV pulse     |
|  - Pulse duration controlled by capacitance               |
+------------------------------------------------------------+
            │
            │
            ▼
+------------------------------------------------------------+
|                  High Voltage (100V - 200V)               |
|  - Provides energy for avalanche breakdown                |
|  - Isolated from Z80 SBC for safety                       |
+------------------------------------------------------------+
```

  
# cct

 
```plaintext
                          +5V
                           │
         +-----------------┴----------------+
         │            Z80 SBC (CPU)         │
         │                                  │
         │  GPIO_TRIGGER_PORT (0x02) ───────┼───────────┐
         │                                  │           │
         │  SPI_CS  (0x05) ───────────────┐ │           │
         │  SPI_MOSI (0x06) ───────────┐  │ │           │
         │  SPI_CLK  (0x07) ───────────┘  │ │           │
         +---------------------------------+ │
                                           │
        +----------------+                 │
        | Digital Pot    | (MCP4131)       │
        | 8-bit, SPI     |                 │
        |---------------+                  │
        | Wiper (Vout)   |─────────────────┘
        +----------------+

        +----------------------------------+
        │  CTC TIMER (Z80)                │
        │  CH1 = Pulse ON Timer (0x03)    │
        │  CH2 = Pulse OFF Timer (0x04)   │
        +----------------------------------+

        +----------------------------------+
        │  555 Timer (Monostable Mode)    │
        │                                  │
        │  +5V ────────┬──────────┬───[8] VCC
        │              │          │
        │             [10kΩ]      │
        │              │         [4] RESET
        │ Z80 GPIO ────┤[2] TRIG │
        │              │         │
        │              │        [5] CV (Cap to GND)
        │              │         │
        │              │        [3] OUTPUT ───────────┐
        │              │         │                    │
        │              │        [6] THR ───┐         │
        │              │                    │         │
        │              └────────[C]─────────┘         │
        │                (Timing Cap)                 │
        │                                             │
        +---------------------------------------------+

        +----------------------------------+
        │  Optocoupler (4N35 / PC817)      │
        │                                  │
        │  Anode (+5V) ───[1kΩ]───────┐    │
        │                             │    │
        │  Cathode (Input) ──── 555 ──┴────┼────────┐
        │                                  │        │
        │  Transistor Collector ────┐      │        │
        │                           │      │        │
        │  Transistor Emitter ─── GND      │        │
        +----------------------------------+        │
                                                   │
        +------------------------------------------│--+
        │  MOSFET Driver (TC4420)                  │  |
        │                                          │  |
        │  Gate ─────────── Optocoupler Output    │  |
        │  Source ──────── GND                    │  |
        │  Drain ─────────+ HV Trigger Signal     │  |
        +------------------------------------------+  |
                                                      │
        +---------------------------------------------│--+
        │  Avalanche Transistor Circuit (HV Pulse)   │  |
        │                                            │  |
        │  +HV (100-200V) ────────┐                  │  |
        │                          │                 │  |
        │  R_charge (10k-100kΩ) ───┤                 │  |
        │                          │                 │  |
        │  C_storage (100pF-10nF) ─┴──┐              │  |
        │                            │              │  |
        │  Avalanche Transistor (ZTX415)            │  |
        │       ┌──────────────┐                    │  |
        │       │  Collector    │───────────────────┘  |
        │       │  Base (Trigger)│────── MOSFET Drain  |
        │       │  Emitter      │────── GND           |
        │       └──────────────┘                      |
        +---------------------------------------------+
```

 
---

# I/O Port Assignments
| **Port Address** | **Name**               | **Function** |
|---------------|-------------------|------------|
| **0x01**      | **CTC_CONTROL**   | Configures Z80 **CTC timer** (sets pulse timing, frequency, and delays). |
| **0x02**      | **GPIO_TRIGGER**  | Outputs **trigger signal** to the **555 Timer One-Shot** for generating a sharp pulse. |
| **0x03**      | **CTC_PULSE_ON**  | Sets the **ON duration** of the pulse (width). |
| **0x04**      | **CTC_PULSE_OFF** | Sets the **OFF duration** (pulse frequency control). |
| **0x05**      | **SPI_CS**        | SPI **Chip Select** for controlling the **MCP4131 Digital Potentiometer** (used to adjust pulse parameters). |
| **0x06**      | **SPI_MOSI**      | SPI **Master Out, Slave In** (sends data to the digital potentiometer). |
| **0x07**      | **SPI_CLK**       | SPI **Clock** for synchronizing data transfer to the digital potentiometer. |

---


# sudo

````
# TEC-APG v2 - Optimized I/O Bit-Masked Control
# Uses a single I/O port (0x01) and manipulates bits for control functions

# ---------------------- DEFINE CONSTANTS ---------------------- #
DEFINE IO_PORT         = 0x01   # Single I/O port for all control signals

# Bit masks for individual functions
DEFINE BIT_TRIGGER     = 0x01   # GPIO_TRIGGER (555 Timer One-Shot)
DEFINE BIT_SPI_CS      = 0x02   # SPI Chip Select
DEFINE BIT_SPI_MOSI    = 0x04   # SPI Data Out
DEFINE BIT_SPI_CLK     = 0x08   # SPI Clock
DEFINE BIT_CTC_ON      = 0x10   # CTC_PULSE_ON (Pulse Width)
DEFINE BIT_CTC_OFF     = 0x20   # CTC_PULSE_OFF (Pulse Frequency)

# ---------------------- DECLARE VARIABLES ---------------------- #
DECLARE INTEGER io_state = 0x00   # Stores the current state of the I/O port
DECLARE INTEGER pulse_frequency = 1000  # Default frequency (Hz)
DECLARE INTEGER pulse_width     = 100   # Default pulse width (µs)
DECLARE INTEGER user_input_freq  # User input for frequency
DECLARE INTEGER user_input_width # User input for width

# ---------------------- INITIALIZATION ---------------------- #
FUNCTION INITIALIZE():
    io_state = 0x00         # Clear all control bits
    OUTPUT(IO_PORT, io_state) # Initialize I/O port

# ---------------------- MAIN LOOP ---------------------- #
FUNCTION MAIN_LOOP():
    WHILE TRUE:
        # Adjust Pulse Frequency using Digital Potentiometer
        IF BUTTON_PRESSED(FREQ_BUTTON):
            user_input_freq = READ_INPUT()
            IF user_input_freq > 65535:
                user_input_freq = 65535
            pulse_frequency = user_input_freq

        # Adjust Pulse Width using Digital Potentiometer
        IF BUTTON_PRESSED(WIDTH_BUTTON):
            user_input_width = READ_INPUT()
            IF user_input_width > 65535:
                user_input_width = 65535
            pulse_width = user_input_width

        # Trigger a Single Pulse
        TRIGGER_PULSE()

        # SPI Communication with Digital Potentiometer
        SET_DIGIPOT(pulse_width, pulse_frequency)

        # Wait for next event
        WAIT_FOR_INTERRUPT()

# ---------------------- TRIGGER PULSE FUNCTION ---------------------- #
FUNCTION TRIGGER_PULSE():
    io_state = io_state | BIT_TRIGGER  # Set BIT_TRIGGER HIGH
    OUTPUT(IO_PORT, io_state)          # Send command to I/O port

    CALL SHORT_DELAY                   # Maintain pulse for a brief period

    io_state = io_state & ~BIT_TRIGGER # Clear BIT_TRIGGER (LOW)
    OUTPUT(IO_PORT, io_state)          # Send updated state to I/O port

# ---------------------- SPI WRITE FUNCTION ---------------------- #
FUNCTION SET_DIGIPOT(width, frequency):
    io_state = io_state & ~BIT_SPI_CS   # Activate SPI_CS (LOW)
    OUTPUT(IO_PORT, io_state)

    SEND_SPI_BYTE(width)                 # Send pulse width setting
    SEND_SPI_BYTE(frequency)              # Send frequency setting

    io_state = io_state | BIT_SPI_CS    # Deactivate SPI_CS (HIGH)
    OUTPUT(IO_PORT, io_state)

# ---------------------- SEND SPI BYTE FUNCTION ---------------------- #
FUNCTION SEND_SPI_BYTE(DATA_BYTE):
    INTEGER BIT_INDEX
    FOR BIT_INDEX = 7 TO 0:
        io_state = io_state & ~BIT_SPI_CLK  # Set SPI_CLK LOW
        OUTPUT(IO_PORT, io_state)

        IF (DATA_BYTE & (1 << BIT_INDEX)) != 0:
            io_state = io_state | BIT_SPI_MOSI  # Send '1' bit
        ELSE:
            io_state = io_state & ~BIT_SPI_MOSI # Send '0' bit
        OUTPUT(IO_PORT, io_state)

        io_state = io_state | BIT_SPI_CLK  # Set SPI_CLK HIGH (Clock Data)
        OUTPUT(IO_PORT, io_state)

    io_state = io_state & ~BIT_SPI_CLK  # Finish last bit transfer
    OUTPUT(IO_PORT, io_state)

# ---------------------- INTERRUPT HANDLER (CTC-BASED TIMING) ---------------------- #
FUNCTION ISR_CTC():
    io_state = io_state | BIT_CTC_ON   # Set pulse ON time
    OUTPUT(IO_PORT, io_state)

    WAIT_FOR_TIMER(pulse_width)        # Wait for ON duration

    io_state = io_state & ~BIT_CTC_ON  # Turn pulse OFF
    OUTPUT(IO_PORT, io_state)

    WAIT_FOR_TIMER(pulse_frequency - pulse_width)  # Wait for next cycle

# ---------------------- SYSTEM STARTUP ---------------------- #
INITIALIZE()
MAIN_LOOP()

```



# ref
- https://hackaday.com/2013/08/06/avalanche-pulse-generator-design/
- https://www.codrey.com/electronic-circuits/avalanche-pulse-generator-an-introduction/
- https://en.wikipedia.org/wiki/Avalanche_transistor
- https://dodgyengineering.com/2016/08/10/junk-box-2n3904-avalanche-pulse-generator/
- 

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

Here is the **functional hardware block diagram** of **TEC-APG v2**, focusing on how different hardware components interact to generate precise **high-voltage avalanche pulses**.

---

### **🔹 TEC-APG v2 Functional Hardware Block Diagram (ASCII)**
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

---

### **🔹 How It Works (Step-by-Step)**
1. **Z80 SBC Generates Pulse Timing**
   - The **CTC timer** calculates when to trigger a pulse.
   - **GPIO (0x02) outputs a HIGH signal** to start the pulse.

2. **555 Timer One-Shot Generates Sharp Trigger Pulse**
   - The **555 monostable circuit** ensures a precise, **short-duration** pulse.
   - Prevents false triggering or excessive ON time.

3. **Optocoupler Electrically Isolates High-Voltage Circuit**
   - The optocoupler translates the **5V trigger** into a **high-voltage-safe signal**.

4. **MOSFET Driver Amplifies Trigger Pulse**
   - Converts the **low-power optocoupler output** into a **high-speed switching pulse**.

5. **Avalanche Transistor Discharges High-Voltage Pulse**
   - When triggered, the **ZTX415 transistor breaks down** and **rapidly discharges C_storage**.
   - This generates a **fast, high-voltage pulse**.

6. **Circuit Resets for Next Pulse**
   - The **capacitor recharges** via **R_charge**.
   - The system waits for the next Z80 **CTC-generated timing event**.

---

### **🔹 Key Functional Improvements**
✅ **Ensures Fast, Stable, and Repeatable High-Voltage Pulses**  
✅ **Prevents Over-triggering with a Monostable One-Shot (555 Timer)**  
✅ **Fully Isolates Z80 from HV using an Optocoupler**  
✅ **SPI-Controlled Digital Potentiometer for Real-Time Pulse Adjustments**  

---

### **🔹 Next Steps**
Would you like:
1. **Z80 Assembly Code for GPIO + 555 Timer Triggering?**  
2. **Component Values for Precise Pulse Width Control?**  
3. **Additional Safety Features (HV Current Limiting, Filtering)?**  

This **hardware block diagram** ensures a **fully functional TEC-APG v2**, optimized for **precise high-voltage pulse generation**. 🚀

# cct

Here's the **updated ASCII circuit diagram** incorporating the **555 Timer One-Shot Pulse Generator** to ensure a **sharp trigger pulse** for the avalanche transistor circuit.

---

### **🔹 Updated TEC-APG v2 Circuit Diagram with 555 Timer One-Shot Pulse**
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

### **🔹 How the 555 Timer One-Shot Works**
1. **Z80 sends a GPIO HIGH pulse** to the **555 Timer's Trigger Input**.
2. **The 555 Timer generates a short, precise pulse** (width set by `R` and `C` values).
3. **555 Timer output goes HIGH** for a fixed time, then automatically goes LOW.
4. **This pulse is passed through the optocoupler**, isolating the **Z80 from HV circuits**.
5. **The MOSFET driver amplifies the signal**, triggering the **avalanche transistor breakdown**.

---

### **🔹 Why This Works Better**
✅ **Ensures a sharp, fast trigger pulse**  
✅ **Prevents accidental continuous triggering**  
✅ **Fully resets before next pulse cycle**  
✅ **Controlled by `R` and `C` values in the 555 Timer**  
✅ **Isolates Z80 SBC from high-voltage circuits**

---

### **🔹 Next Steps**
Would you like:
1. **Component values for R and C in the 555 timer circuit?**  
2. **Full Z80 assembly implementation for one-shot pulse triggering?**  
3. **Simulated waveforms for expected output?**  

This **updated circuit** makes TEC-APG **more reliable and accurate** while ensuring **proper avalanche transistor operation**. 🚀




# ref
- https://hackaday.com/2013/08/06/avalanche-pulse-generator-design/
- https://www.codrey.com/electronic-circuits/avalanche-pulse-generator-an-introduction/
- https://en.wikipedia.org/wiki/Avalanche_transistor
- https://dodgyengineering.com/2016/08/10/junk-box-2n3904-avalanche-pulse-generator/
- 

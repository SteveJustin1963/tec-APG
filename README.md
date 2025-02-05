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





# ref
- https://hackaday.com/2013/08/06/avalanche-pulse-generator-design/
- https://www.codrey.com/electronic-circuits/avalanche-pulse-generator-an-introduction/
- https://en.wikipedia.org/wiki/Avalanche_transistor
- https://dodgyengineering.com/2016/08/10/junk-box-2n3904-avalanche-pulse-generator/
- 

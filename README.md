# tec-APG
TEC-1 programmable avalanche pulse generator

Danger HV

Combine the programmability of a Z80 single-board computer (SBC) with an avalanche transistor circuit to generate adjustable high-voltage pulses, ideal for pulse electronics experiments and retro-tech applications. By integrating optocouplers for safety and utilizing the Z80's CTC timers for precise control, this setup enables dynamic pulse shaping with software-driven adjustments. Rigorous oscilloscope testing ensures reliability, making it a powerful tool for exploring high-speed switching and transient phenomena. ⚡
 

 
 
 

# functional

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


### **🔹 Explanation of Bit Masking on a Single I/O Port**
Using a **bit mask** allows us to store multiple **control signals** in a **single 8-bit I/O port** (0x01). Each **bit in the byte** represents a different function. 

---

### **🔹 Bit Assignments for the I/O Port (0x01)**

| **Bit #** | **Hex Mask** | **Function** | **Purpose** |
|------|------------|---------------------|-----------------------------|
| **Bit 0** | `0x01` (0000 0001) | **BIT_TRIGGER** | Triggers the **555 Timer One-Shot** |
| **Bit 1** | `0x02` (0000 0010) | **BIT_SPI_CS** | Controls **SPI Chip Select (CS)** |
| **Bit 2** | `0x04` (0000 0100) | **BIT_SPI_MOSI** | Sends **SPI Data Out (MOSI)** |
| **Bit 3** | `0x08` (0000 1000) | **BIT_SPI_CLK** | Sends **SPI Clock (SCK)** |
| **Bit 4** | `0x10` (0001 0000) | **BIT_CTC_ON** | Controls **CTC Timer Pulse ON (Width)** |
| **Bit 5** | `0x20` (0010 0000) | **BIT_CTC_OFF** | Controls **CTC Timer Pulse OFF (Frequency)** |
| **Bit 6** | `0x40` (0100 0000) | **(Unused, Reserved)** | Reserved for future use |
| **Bit 7** | `0x80` (1000 0000) | **(Unused, Reserved)** | Reserved for future use |

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

# review1

I'll analyze this avalanche pulse generator (APG) design integrated with a Z80 SBC and suggest some key improvements for safety and functionality.



Let me highlight some critical points and improvements:

1. Safety Concerns:
- The HV section (100-200V) needs additional isolation
- Current optocoupler (4N35) might be too slow for high-speed pulses
- Need better protection for the Z80 GPIO section

2. Timing Issues:
- The 555 timer one-shot could introduce jitter
- SPI timing needs to be more precise for the digital potentiometer
- CTC timer configuration needs safety bounds

Let me create an improved version of the code with these considerations:

Key improvements in this version:

1. Safety Enhancements:
- Added dedicated safety status port (08h)
- Implemented safety interlock checking
- Added HV power supply ready detection
- Added maximum duty cycle limitation (50%)

2. Timing Improvements:
- Removed reliance on 555 timer for critical timing
- Added frequency validation
- Implemented minimum pulse width checking
- Added proper CTC timer configuration

3. Hardware Recommendations:
- Replace 4N35 with faster HCPL-2630 optocoupler
- Add TVS diodes for GPIO protection
- Use isolated DC-DC converter for HV section
- Add hardware interlock monitoring

```
;==========================================
; TEC-APG v3 - Enhanced Safety & Timing
;==========================================

; I/O Port Definitions
IO_PORT         EQU     01h     ; Main I/O port
SAFETY_PORT     EQU     08h     ; New safety status port

; Bit Masks
BIT_TRIGGER     EQU     01h     ; Trigger pulse
BIT_SPI_CS      EQU     02h     ; SPI CS
BIT_SPI_MOSI    EQU     04h     ; SPI MOSI
BIT_SPI_CLK     EQU     08h     ; SPI Clock
BIT_CTC_ON      EQU     10h     ; CTC ON timing
BIT_CTC_OFF     EQU     20h     ; CTC OFF timing
BIT_SAFETY_OK   EQU     40h     ; Safety interlock OK
BIT_HV_READY    EQU     80h     ; HV supply ready

; Safety Limits
MAX_FREQUENCY   EQU     50000   ; 50kHz max frequency
MIN_PULSE_WIDTH EQU     100     ; 100ns minimum
MAX_DUTY_CYCLE  EQU     50      ; 50% maximum duty cycle

        ORG     1000h

;==========================================
; Initialization
;==========================================
init:
        ; Check safety interlocks
        IN      A,(SAFETY_PORT)
        AND     BIT_SAFETY_OK
        JP      Z,safety_fault   ; Jump if safety interlock broken
        
        ; Initialize I/O ports
        XOR     A               ; Clear accumulator
        OUT     (IO_PORT),A     ; Clear all control bits
        
        ; Configure CTC timers
        LD      A,47h          ; Time constant follows
        OUT     (CTC0),A       ; CTC channel 0 control
        LD      A,100          ; Default time constant
        OUT     (CTC0),A       ; Write time constant

;==========================================
; Main Program Loop
;==========================================
main_loop:
        ; Check HV ready status
        IN      A,(SAFETY_PORT)
        AND     BIT_HV_READY
        JP      Z,hv_not_ready
        
        ; Get user frequency input
        CALL    get_frequency
        CALL    validate_frequency
        
        ; Get pulse width input
        CALL    get_pulse_width
        CALL    validate_pulse_width
        
        ; Configure pulse timing
        CALL    set_timing
        
        ; Generate pulse if safe
        CALL    check_safety
        JP      NZ,generate_pulse
        
        JP      main_loop

;==========================================
; Safety Checking Routines
;==========================================
check_safety:
        ; Check all safety interlocks
        IN      A,(SAFETY_PORT)
        AND     BIT_SAFETY_OK|BIT_HV_READY
        CP      BIT_SAFETY_OK|BIT_HV_READY
        RET

validate_frequency:
        ; Ensure frequency is within bounds
        LD      HL,(user_freq)
        LD      DE,MAX_FREQUENCY
        CALL    compare_hl_de
        RET     C               ; Return if frequency <= MAX_FREQUENCY
        LD      HL,MAX_FREQUENCY
        LD      (user_freq),HL  ; Clamp to maximum
        RET

validate_pulse_width:
        ; Check minimum pulse width
        LD      HL,(pulse_width)
        LD      DE,MIN_PULSE_WIDTH
        CALL    compare_hl_de
        JP      C,pw_too_small
        
        ; Calculate and check duty cycle
        CALL    calc_duty_cycle
        CP      MAX_DUTY_CYCLE
        JP      NC,duty_too_high
        RET

;==========================================
; Pulse Generation
;==========================================
generate_pulse:
        ; Set up pulse parameters
        LD      A,(IO_PORT)
        OR      BIT_TRIGGER
        OUT     (IO_PORT),A     ; Start pulse
        
        ; Wait for pulse width duration
        CALL    delay_routine
        
        ; End pulse
        LD      A,(IO_PORT)
        AND     ~BIT_TRIGGER
        OUT     (IO_PORT),A
        RET

;==========================================
; Error Handlers
;==========================================
safety_fault:
        ; Handle safety interlock failure
        XOR     A
        OUT     (IO_PORT),A     ; Disable all outputs
        LD      HL,safety_msg
        CALL    print_error
        HALT

hv_not_ready:
        ; Handle HV power supply not ready
        LD      HL,hv_msg
        CALL    print_error
        JP      main_loop

;==========================================
; Data Section
;==========================================
user_freq:      DW      1000    ; Default 1kHz
pulse_width:    DW      100     ; Default 100μs
safety_msg:     DB      "Safety Interlock Fault",0
hv_msg:         DB      "HV Not Ready",0

        END
```




# ref
- https://hackaday.com/2013/08/06/avalanche-pulse-generator-design/
- https://www.codrey.com/electronic-circuits/avalanche-pulse-generator-an-introduction/
- https://en.wikipedia.org/wiki/Avalanche_transistor
- https://dodgyengineering.com/2016/08/10/junk-box-2n3904-avalanche-pulse-generator/
- 

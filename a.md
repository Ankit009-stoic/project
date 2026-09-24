### 1. Load Subsystem (Induction Motor)

| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
| --- | --- | --- | --- | --- |
| **3-Phase Induction Motor** | $60\text{ W}$ ($0.08\text{ HP}$ / $0.06\text{ kW}$), $230\text{ V}$ ($\Delta$) / $415\text{ V}$ ($Y$), $0.41\text{ A}$ ($\Delta$), $1440\text{ RPM}$, $50\text{ Hz}$, 4-Pole

 | Frame 56 Squirrel-Cage AC Motor (Tachometric Controls / Oriental Motor equivalent)

 | 1 | Dynamic mechanical/inductive load; connected in **Delta ($\Delta$)** configuration for $230\text{ V}$ VFD output.

 |

---

### 2. AC Input Rectification & DC-Link Stage

| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
| --- | --- | --- | --- | --- |
| **Bridge Rectifier IC** | Single-phase full-bridge, $V_{RRM} \ge 600\text{ V}$, $I_{F(AV)} \ge 2\text{ A}$ | **KBP206**, **GBU4J**, or **W10M** | 1 | Rectifies $230\text{ V}$ RMS grid AC to unregulated peak $325.2\text{ V}$ DC. |
| **DC-Link Filter Capacitor** | $150\,\mu\text{F} \text{ to } 220\,\mu\text{F}$, $400\text{ V} - 450\text{ V DC}$, High-ripple electrolytic | Nichicon / Rubycon Radial Aluminum Electrolytic | 1 | Filters the $100\text{ Hz}$ rectification ripple below $5\%$ peak-to-peak. |
| **DC Bleeder Resistor** | $100\text{ k}\Omega \text{ to } 220\text{ k}\Omega$, $2\text{ W}$ Metal Oxide / Ceramic | Flameproof Power Resistor | 1 | Discharges the $325\text{ V}$ capacitor safely to $0\text{ V}$ when mains AC is disconnected. |
| **Inrush Current Limiter (NTC)** | $10\,\Omega \text{ to } 20\,\Omega$ cold resistance, $2\text{ A}$ steady-state | **NTC 10D-9** or **10D-11** | 1 | Limits initial charging surge current into the uncharged DC capacitor at power-up. |
| **High-Frequency Decoupling Cap** | $0.1\,\mu\text{F}$ ($100\text{ nF}$), $630\text{ V} - 1000\text{ V}$ Metallized Polypropylene | Film Capacitor (MKP / Box type) | 1 | Sits directly across the DC+ and DC- inverter rails to absorb high-frequency switching spikes. |

---

### 3. 3-Phase Inverter Power Stage

| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
| --- | --- | --- | --- | --- |
| **Power MOSFETs** | N-Channel, $V_{DSS} \ge 500\text{ V} - 600\text{ V}$, $I_D \ge 4\text{ A} - 8\text{ A}$, TO-220 package | **STP4NK60Z**, **IRFBC30**, or **IRF840** | 6 | Arranged in 3 half-bridge legs to switch the $325\text{ V}$ bus and synthesize 3-phase AC. |
| **Heatsinks** | TO-220 clip-on aluminum heat spreaders | Standard finned TO-220 heatsink | 6 | Dissipates conduction and switching thermal losses ($< 0.5\text{ W}$ per device). |
| **Mica Insulators & Bushings** | Thermal insulating pads + nylon screw bushings | Standard TO-220 mounting kit | 6 | Electrically isolates the MOSFET drain tabs (connected to DC+) from a shared heatsink. |

---

### 4. Gate Driver, Bootstrap & Isolation Board

| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
| --- | --- | --- | --- | --- |
| **Gate Driver ICs** | High- and Low-Side Driver, floating high-side channel up to $600\text{ V}$, $2\text{ A}$ sink/source | **IR2110** or **IR2113** (DIP-14) | 3 | Drives high-side and low-side MOSFETs per inverter leg from logic-level PWM. |
| **High-Speed Optocouplers** | Single-channel high-speed digital isolator, $10\text{ Mbps}$, $V_{ISO} \ge 2500\text{ V}_{rms}$ | **6N137** (or **HCPL-2601**) | 6 | Provides galvanic isolation between the low-voltage controller and high-voltage power ground. |
| **Bootstrap Diodes** | Ultra-Fast Recovery, $V_{RRM} \ge 600\text{ V} - 1000\text{ V}$, $I_F \ge 1\text{ A}$, $t_{rr} \le 75\text{ ns}$ | **UF4007** or **MUR160** | 3 | Charges the high-side floating bootstrap capacitors when low-side switches conduct. |
| **Bootstrap Capacitors** | $1.0\,\mu\text{F} \text{ to } 2.2\,\mu\text{F}$, $50\text{ V}$, Low ESR | Multilayer Ceramic (MLCC) / Tantalum | 3 | Supplies floating gate drive voltage ($V_B - V_S$) for high-side MOSFET conduction. |
| **Gate Resistors ($R_g$)** | $15\,\Omega \text{ to } 22\,\Omega$, $0.25\text{ W}$, Metal Film | Standard 1/4W Resistor | 6 | Limits gate peak turn-on current and prevents parasitic gate-source ringing. |
| **Fast Turn-Off Diodes** | Fast Switching Diode, $V_R \ge 75\text{ V}$, $I_F \ge 150\text{ mA}$ | **1N4148** | 6 | Placed anti-parallel across $R_g$ to bypass the resistor during turn-off for faster discharge. |
| **Gate Pull-Down Resistors** | $10\text{ k}\Omega$, $0.25\text{ W}$ | Standard 1/4W Resistor | 6 | Tied between Gate and Source to keep MOSFETs safely OFF during controller reset/startup. |
| **Optocoupler Current Limiters** | $330\,\Omega \text{ to } 470\,\Omega$, $0.25\text{ W}$ | Standard 1/4W Resistor | 6 | Limits forward current ($I_F \approx 10\text{ mA}$) from microcontroller pins through optocoupler LEDs. |
| **Optocoupler Pull-Up Resistors** | $1\text{ k}\Omega \text{ to } 4.7\text{ k}\Omega$, $0.25\text{ W}$ | Standard 1/4W Resistor | 6 | Pulls open-collector outputs of the 6N137 up to the $+5\text{ V}$ driver logic rail. |
| **IC Bypass Capacitors** | $0.1\,\mu\text{F}$ ($100\text{ nF}$), $50\text{ V}$, Ceramic | 104 MLCC / Ceramic Disc | 9 | Decouples $V_{CC}$ and $V_{DD}$ supply pins on the IR2110 (3x) and optocouplers (6x). |

---

### 5. Controller, Sensing & Feedback

| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
| --- | --- | --- | --- | --- |
| **Microcontroller Board** | $16\text{ MHz}$ AVR, Hardware PWM Timers, External Interrupt Pins | **Arduino Uno**, **Nano**, or **Mega 2560** | 1 | Executes speed setpoint ramps, discrete PI control loop, and SPWM generation. |
| **Rotor Speed Sensor** | Incremental Optical / Magnetic Rotary Shaft Encoder, $5\text{ V}$, Quadrature AB output | Omron **E6B2-CWZ6C** ($100 - 360\text{ PPR}$) or Slotted Optical Disc Module | 1 | Measures physical motor shaft speed to feed real-time RPM back into the closed-loop PI controller. |
| **Speed Reference Potentiometer** | $10\text{ k}\Omega$ Linear Rotary Potentiometer | B10K Potentiometer with dial knob | 1 | Provides manual target speed adjustment ($0\text{ to } 1440\text{ RPM}$) via Arduino ADC input. |
| **Pushbuttons (Tactile)** | SPST Momentary Tactile Switch ($6\times 6\text{ mm}$) | Standard PCB Tactile Switch | 2 | Manual START / RUN and STOP commands. |
| **Status Indicator LEDs** | $3\text{ mm}$ or $5\text{ mm}$ LEDs (Red, Green, Yellow) | Standard THT LEDs + $330\,\Omega$ series resistors | 3 | Displays system states: Power ON (Green), Inverter Running (Yellow), Fault/Trip (Red). |

---

### 6. Auxiliary Power Supply & Circuit Protection

| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
| --- | --- | --- | --- | --- |
| **Gate Driver DC Supply** | $+12\text{ V} \text{ to } +15\text{ V DC}$, $1\text{ A}$ regulated | Regulated SMPS Wall Adapter or Linear Supply | 1 | Powers the IR2110 gate driver output stages ($V_{CC}$). |
| **Logic DC Supply** | $+5\text{ V DC}$, $1\text{ A}$ regulated | **LM7805** Linear Regulator or 5 V Step-Down Module | 1 | Powers Arduino logic, 6N137 optocouplers, and rotary optical encoder. |
| **AC Mains Input Fuse** | Fast-acting / Slow-blow Glass Fuse, $250\text{ V}$, $2\text{ A}$ | $5\times 20\text{ mm}$ Cartridge Fuse + Panel/PCB Holder | 1 | Overcurrent and short-circuit protection for mains input. |
| **DC Bus Fuse** | Fast-acting Ceramic Fuse, $500\text{ V DC}$, $2\text{ A}$ | $6.3\times 32\text{ mm}$ High-Voltage DC Fuse | 1 | Protects the DC bus against inverter shoot-through or MOSFET breakdown. |

---

### 7. OPAL-RT Real-Time HIL Interface Hardware

| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
| --- | --- | --- | --- | --- |
| **DB37 Breakout Board** | 37-Pin D-Sub Female connector with screw terminals | Standard DB37-M6 Breakout Module | 1 | Connects physical oscilloscope probes to OPAL-RT Slot 2 Group B analog output pins.

 |
| **Analog Scaling Resistors** | Precision Metal Film ($0.1\%$ or $1\%$), $0.25\text{ W}$ | $100\text{ k}\Omega$ and $3.16\text{ k}\Omega$ (Divider Ratio $\approx 0.03077$)

 | 3 sets | Hardware passive attenuator if scaling high external voltages into OPAL-RT analog inputs.

 |
| **Oscilloscope Interconnect** | BNC to Alligator Clip / Minigrabber Test Leads | Standard $50\,\Omega$ BNC Cable | 3 | Connects DB37 Phase A, B, C channels (Pins 1, 2, 3 and grounds 20, 21, 22) to DSO channels.

 |

---

### 8. Mechanical & Wiring Hardware

| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
| --- | --- | --- | --- | --- |
| **PCB / Prototyping Board** | Double-sided FR4 copper clad perfboard ($10\times 15\text{ cm}$) or Custom 2-Layer PCB | FR4 Prototype PCB | 1 | Physical substrate for soldering inverter switches, drivers, and isolation. |
| **High-Voltage Screw Terminals** | 2-Pin and 3-Pin PCB Terminal Blocks, $300\text{ V} - 600\text{ V}$, $10\text{ A}$ rating, $5.08\text{ mm}$ pitch | Phoenix Contact / Degson Terminal Blocks | 4 | Wire terminals for AC Mains Input, DC Bus monitor, and 3-Phase Motor Out ($U, V, W$). |
| **Hookup Wire** | $18\text{ AWG}$ stranded wire ($600\text{ V}$ rated) + $24\text{ AWG}$ wire | UL1007 / UL1015 hookup wire | Assorted | $18\text{ AWG}$ for AC/DC power tracks and motor phases; $24\text{ AWG}$ for gate drive and logic signals. |

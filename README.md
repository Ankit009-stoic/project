## 1. Motor Equivalent Circuit Modeling & Parameter Identification

The physical machine under control is a fractional-horsepower, squirrel-cage induction motor operating in a Delta ($\Delta$) configuration.

```
                Stator Winding                    Rotor Winding (Referred)
             Rs             Lls                  Llr'            Rr'/s
        ───/\/\/\──────────UUUUU───────┬────────UUUUU──────────/\/\/\────
                                       │
                                       │
                                    ┌──┴──┐
                                    │     │
                                    │ Lm  │
                                    │     │
                                    └──┬──┘
                                       │
        ───────────────────────────────┴─────────────────────────────────

```

### Motor Nameplate Specifications

* **Mechanical Shaft Power ($P_{mech}$):** $0.06\text{ kW} = 60\text{ W}$ ($0.08\text{ HP}$)


* **Rated Line Voltage ($V_{line,rms}$):** $230\text{ V AC}$ (Delta, $\Delta$)


* **Rated Line Current ($I_{line,rms}$):** $0.41\text{ A}$

* **Base Electrical Frequency ($f_{rated}$):** $50\text{ Hz}$

* **Rated Rotor Speed ($N_r$):** $1440\text{ RPM}$

* **Pole Pairs ($p$):** 2 (giving a 4-pole machine with synchronous speed $N_s = 1500\text{ RPM}$)



---

### Step 1.1: Mechanical Velocity, Rated Slip, and Shaft Torque

#### Calculation:

* **Synchronous Speed ($N_s$):**

$$N_s = \frac{120 \times f_{rated}}{2p} = \frac{120 \times 50}{4} = 1500\text{ RPM}$$


* **Rated Mechanical Angular Velocity ($\omega_m$):**

$$\omega_m = N_r \times \frac{2\pi}{60} = 1440 \times \frac{2\pi}{60} \approx 150.80\text{ rad/s}$$


* **Rated Slip ($s$):**

$$s = \frac{N_s - N_r}{N_s} = \frac{1500 - 1440}{1500} = \frac{60}{1500} = 0.04\quad (4.0\%)$$


* **Rated Mechanical Torque ($T_{nom}$):**

$$T_{nom} = \frac{P_{mech}}{\omega_m} = \frac{60\text{ W}}{150.80\text{ rad/s}} \approx 0.398\text{ N}\cdot\text{m}$$



#### Significance & Defense:

$T_{nom} \approx 0.40\text{ N}\cdot\text{m}$ defines the continuous torque the motor produces without thermal breakdown. In Simulink, this value dictates the constant block fed into the mechanical torque input ($T_m$) of the `Asynchronous Machine` block to verify full-load operation.

---

### Step 1.2: Motor Base Quantities

#### Calculation:

* **Apparent Electrical Power ($S_{max}$):**

$$S_{max} = \sqrt{3} \times V_{line,rms} \times I_{line,rms} = \sqrt{3} \times 230 \times 0.41 \approx 163.33\text{ VA}$$


* **Base Impedance ($Z_{base}$):**

$$Z_{base} = \frac{V_{line,rms}^2}{S_{max}} = \frac{230^2}{163.33} \approx 323.88\,\Omega \approx 324\,\Omega$$


* **Base Electrical Inductance ($L_{base}$):**

$$L_{base} = \frac{Z_{base}}{\omega_{elec}} = \frac{Z_{base}}{2\pi \times f_{rated}} = \frac{323.88}{2\pi \times 50} \approx 1.031\text{ H}$$



#### Significance & Defense:

Small fractional-horsepower motors exhibit high internal impedance relative to their power throughput. $Z_{base}$ establishes the base unit for converting standard per-unit (pu) induction machine parameters into the physical SI quantities (Ohms and Henrys) required by MATLAB/Simulink without relying on 60 Hz software presets.

---

### Step 1.3: Equivalent Circuit Parameters (SI Units)

Using IEEE/IEC standardized per-unit distribution factors for Design B fractional-horsepower induction motors:

* Stator resistance $R_s = 0.05\,\text{pu}$
* Rotor resistance referred to stator $R_r' = 0.05\,\text{pu}$
* Stator leakage inductance $L_{ls} = 0.08\,\text{pu}$
* Rotor leakage inductance referred to stator $L_{lr}' = 0.08\,\text{pu}$
* Magnetizing inductance $L_m = 1.49\,\text{pu}$

#### Calculation:

* **Stator Resistance ($R_s$):**

$$R_s = 0.05 \times Z_{base} = 0.05 \times 323.88 = 16.19\,\Omega \approx 16.2\,\Omega$$


* **Rotor Resistance ($R_r'$):**

$$R_r' = 0.05 \times Z_{base} = 0.05 \times 323.88 = 16.19\,\Omega \approx 16.2\,\Omega$$


* **Leakage Inductances ($L_{ls}, L_{lr}'$):**

$$L_{ls} = L_{lr}' = 0.08 \times L_{base} = 0.08 \times 1.031 \approx 0.0825\text{ H} \approx 0.08\text{ H}$$


* **Magnetizing/Mutual Inductance ($L_m$):**

$$L_m = 1.49 \times L_{base} = 1.49 \times 1.031 \approx 1.536\text{ H} \approx 1.54\text{ H}$$


* **Rotor Inertia ($J$) & Friction Coefficient ($F$):**
For a standard Frame 56 motor body:



$$J = 0.0005\text{ kg}\cdot\text{m}^2,\quad F = 0.0001\text{ N}\cdot\text{m}\cdot\text{s}$$



#### Simulink Parameter Vector:

`[Pn, Vn, fn]` $\rightarrow$ `[163.3, 230, 50]`

`[Rs, Lls]` $\rightarrow$ `[16.2, 0.08]`

`[Rr', Llr']` $\rightarrow$ `[16.2, 0.08]`

`Lm` $\rightarrow$ `1.54`

`[J, F, p]` $\rightarrow$ `[0.0005, 0.0001, 2]`

---

## 2. AC Rectification & DC-Link Filter Stage

```
   AC Mains 230V RMS                       DC Bus (V_dc = 325.2V)
      ───o────┬───────[ D1 ]──┬───[ D3 ]───────┬──────o (+)
              │               │                │
              │          AC~  │                │
          ( ~ )               │              ┌─┴─┐
              │          AC~  │              │   │ C_dc
              │               │              │   │ (150 µF)
      ───o────┴───────[ D2 ]──┴───[ D4 ]──┐  └─┬─┘
                                          │    │
                                          └────┴──────o (-)

```

### Step 2.1: Nominal DC Bus Voltage ($V_{dc}$)

#### Calculation:

* **RMS Grid Supply:** $V_{AC,rms} = 230\text{ V}$
* **Peak Unloaded DC Voltage:**

$$V_{dc} = \sqrt{2} \times V_{AC,rms} = 1.4142 \times 230\text{ V} \approx 325.27\text{ V DC}$$


* **Loaded Voltage Accounting for Diode Drops:**
Each conduction path passes through two series bridge diodes ($2 \times V_F$, where $V_F \approx 1.0\text{ V}$ for silicon rectifiers):

$$V_{dc,loaded} = (\sqrt{2} \times 230) - 2(1.0) \approx 323.27\text{ V DC}$$



#### Significance & Hardware Connection:

$325.2\text{ V}$ establishes the baseline potential of the high-voltage rails. This dictates the minimum breakdown rating required for the DC-link capacitor, switching transistors, and isolation boundaries.

---

### Step 2.2: Mains Input Current & Rectifier Sizing

#### Calculation:

* **Apparent Power Input:** $S_{max} \approx 163.3\text{ VA}$
* **Input RMS AC Current:**

$$I_{in,rms} = \frac{S_{max}}{V_{AC,rms}} = \frac{163.33\text{ VA}}{230\text{ V}} \approx 0.71\text{ A}_{rms}$$


* **Diode Peak Inverse Voltage ($V_{RRM}$):**

$$V_{RRM} \ge 1.5 \times V_{dc} = 1.5 \times 325.27 = 487.9\text{ V}$$


* **Diode Continuous Forward Current ($I_F$):**

$$I_F \ge 2.0 \times I_{in,rms} = 2.0 \times 0.71 = 1.42\text{ A}$$



#### Hardware Connection:

Selected Component: **KBP206** or **W10M** Single-Phase Bridge Rectifier IC.

* **Component Specs:** $V_{RRM} = 600\text{ V}$, $I_{F(AV)} = 2.0\text{ A}$ (with surge rating $I_{FSM} = 50\text{ A}$).
* **Justification:** The $600\text{ V}$ rating provides a $1.84\times$ margin against utility line surges, while the $2.0\text{ A}$ continuous rating accommodates the $0.71\text{ A}$ operating current without requiring a dedicated heatsink.

---

### Step 2.3: DC-Link Capacitor Sizing ($C_{dc}$)

#### Derivation:

A single-phase full-wave rectifier supplies energy in pulses at twice the grid frequency:


$$f_{ripple} = 2 \times f_{grid} = 2 \times 50\text{ Hz} = 100\text{ Hz}$$


The time between rectified voltage peaks is $T_{ripple} = \frac{1}{100\text{ Hz}} = 10\text{ ms}$.

During the interval where the AC line voltage drops below the DC-bus voltage, the capacitor alone must supply the full electrical power ($P_{elec} \approx S_{max}$) to the inverter. The charge discharged is:


$$\Delta Q = I_{dc} \times \Delta t \approx \frac{S_{max}}{V_{dc}} \times \left(\frac{1}{2 \times f_{ripple}}\right)$$


Since $\Delta Q = C_{dc} \times \Delta V_{dc}$:


$$C_{dc} = \frac{S_{max}}{2 \times f_{ripple} \times V_{dc} \times \Delta V_{dc}}$$

#### Calculation:

* **Allowable Peak-to-Peak Ripple ($\Delta V_{dc}$):** Limit ripple to $5\%$ of nominal DC voltage:

$$\Delta V_{dc} = 0.05 \times 325.27\text{ V} \approx 16.26\text{ V}$$


* **Capacitance Calculation:**

$$C_{dc} = \frac{163.33}{2 \times 100 \times 325.27 \times 16.26} = \frac{163.33}{1,057,378} \approx 154.46 \times 10^{-6}\text{ F} \approx 154.5\,\mu\text{F}$$


* **Minimum Voltage Rating:**

$$V_{cap,rating} \ge 1.25 \times V_{dc} = 1.25 \times 325.27 = 406.6\text{ V}$$



#### Hardware Connection:

Selected Component: **$150\,\mu\text{F}$ or $220\,\mu\text{F}$ Aluminum Electrolytic Capacitor, rated for $400\text{ V} \text{ or } 450\text{ V DC}$**.

* **Justification:** Choosing $220\,\mu\text{F}$ provides conservative ripple suppression ($\Delta V_{dc} < 3.5\%$). A $450\text{ V}$ rating prevents dielectric punch-through during input transients and regenerative deceleration events.

---

## 3. Inverter Power Stage & Switching Device Sizing

```
           (+) DC Bus (325.2V)
            ───┬──────────────┬──────────────┬───
               │              │              │
             ┌─┴─┐ Q1       ┌─┴─┐ Q3       ┌─┴─┐ Q5
             │   │ (UH)     │   │ (VH)     │   │ (WH)
             └─┬─┘          └─┬─┘          └─┬─┘
               ├────── U      ├────── V      ├────── W   ──► To 3-Phase Motor
             ┌─┴─┐          ┌─┴─┐          ┌─┴─┐
             │   │ Q4       │   │ Q6       │   │ Q2
             └─┬─┘ (UL)     └─┬─┘ (VL)     └─┬─┘ (WL)
               │              │              │
            ───┴──────────────┴──────────────┴───
           (-) DC Bus Ground

```

### Step 3.1: Switch Voltage Breakdown Rating ($V_{DS}$)

#### Calculation:

* Peak operational DC voltage: $V_{dc} = 325.27\text{ V}$
* Parasitic switching overvoltage: During inductive turn-off, stray loop inductance ($L_{\sigma}$) induces a spike $V_{spike} = L_{\sigma} \frac{di}{dt}$, commonly reaching $50\text{ V}$ to $100\text{ V}$.
* Required breakdown safety factor ($SF_v \ge 1.8$):

$$V_{DS,min} \ge 1.8 \times V_{dc} = 1.8 \times 325.27 \approx 585.5\text{ V}$$



#### Hardware Connection:

Selected Component: **STP4NK60Z** or **IRFBC30** (N-Channel Power MOSFETs).

* **Component Specs:** $V_{DSS} = 600\text{ V}$.
* *(Alternative: **IRF840**, rated at $500\text{ V}$, acceptable if inductive snubber/decoupling limits transients below $400\text{ V}$).*
* **Justification:** $600\text{ V}$ allows an operating margin of $274.7\text{ V}$ above the nominal $325.3\text{ V}$ rail, safely absorbing commutation spikes without avalanche breakdown.

---

### Step 3.2: Switch Current Capacity ($I_D$)

#### Calculation:

* **Motor Peak Operational Current:**

$$I_{peak} = \sqrt{2} \times I_{line,rms} = \sqrt{2} \times 0.41\text{ A} \approx 0.58\text{ A}$$


* **Motor Locked-Rotor / Inrush Multiplier:**
Induction motors draw roughly $3\times$ to $5\times$ rated current during direct online startup or low-frequency transients:

$$I_{inrush} = 3 \times I_{peak} = 3 \times 0.58\text{ A} \approx 1.74\text{ A}$$


* **Safety Factor ($SF_i \ge 2.0$ over inrush):**

$$I_{D,min} \ge 2.0 \times 1.74\text{ A} \approx 3.48\text{ A}$$



#### Hardware Connection:

* **STP4NK60Z:** $I_D = 4.0\text{ A}$ continuous at $25^\circ\text{C}$ ($I_{DM} = 16\text{ A}$ pulsed), $R_{DS(on)} = 2.0\,\Omega$.
* **IRF840:** $I_D = 8.0\text{ A}$ continuous ($I_{DM} = 32\text{ A}$ pulsed), $R_{DS(on)} = 0.85\,\Omega$.
* **Conduction Losses ($P_{cond}$):**

$$P_{cond} = I_{rms,switch}^2 \times R_{DS(on)} = \left(\frac{0.41}{\sqrt{2}}\right)^2 \times 2.0\,\Omega \approx 0.17\text{ W per switch}$$


* **Justification:** The conduction loss is low ($\approx 170\text{ mW}$), meaning the switches operate coolly with miniature slip-on TO-220 heat spreaders.

---

### Step 3.3: Carrier Switching Frequency ($f_{carrier}$)

#### Calculation:

* Selected carrier frequency: $f_{carrier} = 5000\text{ Hz} = 5\text{ kHz}$

* Switching Period ($T_{carrier}$):

$$T_{carrier} = \frac{1}{f_{carrier}} = \frac{1}{5000} = 200\,\mu\text{s}$$




#### Significance & Defense:

* **Audible Noise vs. Thermal Trade-Off:** While human hearing extends to $20\text{ kHz}$, operating at $5\text{ kHz}$ provides lower switching losses ($E_{sw} \propto f_{carrier}$), allowing the use of basic gate drivers and small heatsinks.
* **Current Ripple Filtering:** The motor's stator leakage inductance ($L_{ls} \approx 80\text{ mH}$) provides an inductive reactance of $X_L = 2\pi \times 5000 \times 0.08 \approx 2513\,\Omega$ at the carrier frequency, attenuating high-frequency current harmonics to $< 1\%$ without requiring an external output filter.

---

## 4. Gate Driver, Isolation, and Bootstrap Calculations

```
                        +15V Vcc
                           │
                           ├───[ Bootstrap Diode UF4007 ]───┐
                           │                                │
                       ┌───┴───┐                            │
                       │       │ Vb ────────────────────────┼─────┐
                       │ IR2110│                            │     │
   Logic (PWM) ───────►│ Gate  │ HO ───[ Rg=15Ω ]───┐       │   ┌─┴─┐
                       │ Driver│                    │     ┌─┴─┐ │   │ Q1 (High)
                       │       │ Vs ────────────────┴─────┤   │ └─┬─┘
                       └───┬───┘                     Cboot│   │   │
                           │                              └───┬─┘ │ Phase Out
                           ▼ To Low-Side Gate                 └───┼───► To Motor
                                                                  ▼

```

### Step 4.1: High-Side Bootstrap Capacitor ($C_{boot}$)

#### Derivation:

The high-side N-Channel MOSFET requires its gate voltage ($V_G$) to remain $10\text{ V} - 15\text{ V}$ above its source terminal ($V_S$). When the high-side switch turns ON, $V_S$ rises to the $325\text{ V}$ rail. The floating capacitor ($C_{boot}$) supplies this gate charge without a separate isolated DC-DC power supply.

The capacitor must supply:

1. Total MOSFET gate charge: $Q_g \approx 60\text{ nC}$ (for IRF840 / STP4NK60Z)
2. Gate driver internal level-shift charge: $Q_{ls} \approx 5\text{ nC}$
3. Floating section quiescent current during maximum low-frequency conduction period ($t_{on,max} = \frac{1}{2 \times 5\text{ Hz}} = 100\text{ ms}$):

$$Q_{leak} = I_{QBS} \times t_{on,max} = 100\,\mu\text{A} \times 0.1\text{ s} = 10{,}000\text{ nC} = 10\,\mu\text{C}$$



Total charge delivered:


$$Q_{total} = Q_g + Q_{ls} + Q_{leak} \approx 0.06\,\mu\text{C} + 0.005\,\mu\text{C} + 10\,\mu\text{C} \approx 10.065\,\mu\text{C}$$

To prevent high-side gate droop, limit the discharge voltage drop across $C_{boot}$ to $\Delta V_{boot} \le 0.5\text{ V}$:


$$C_{boot} \ge \frac{Q_{total}}{\Delta V_{boot}} = \frac{10.065\,\mu\text{C}}{0.5\text{ V}} \approx 20.13\,\mu\text{F}\quad (\text{for continuous } 5\text{ Hz operation})$$

For conventional SPWM where low-side pulses recharge the capacitor every $200\,\mu\text{s}$ ($t_{on,max} \le 200\,\mu\text{s}$):


$$Q_{total} = 60\text{ nC} + 5\text{ nC} + (100\,\mu\text{A} \times 200\,\mu\text{s}) = 65\text{ nC} + 20\text{ nC} = 85\text{ nC}$$

$$C_{boot,min} = \frac{85\text{ nC}}{0.5\text{ V}} = 0.17\,\mu\text{F}$$

#### Hardware Connection:

Selected Component: **$1.0\,\mu\text{F} \text{ to } 2.2\,\mu\text{F}$ Multilayer Ceramic / Tantalum Capacitor ($50\text{ V}$ rated)** placed in parallel with a $0.1\,\mu\text{F}$ ceramic bypass capacitor directly between pins $V_B$ and $V_S$ of the IR2110 IC.

---

### Step 4.2: Bootstrap Diode ($D_{boot}$)

#### Calculation:

* Reverse Blocking Voltage: The diode experiences the full DC rail when the high-side switch is closed:

$$V_{RRM} \ge 1.5 \times V_{dc} = 1.5 \times 325.27 = 487.9\text{ V}$$


* Reverse Recovery Time ($t_{rr}$): The diode must recover quickly as the half-bridge midpoint commutates at $5\text{ kHz}$:

$$t_{rr} \le 100\text{ ns}$$



#### Hardware Connection:

Selected Component: **UF4007** or **MUR160**.

* **Specs:** $V_{RRM} = 1000\text{ V}$, $I_F = 1.0\text{ A}$, ultra-fast reverse recovery time $t_{rr} = 75\text{ ns}$.
* **Justification:** Standard $1\text{N}4007$ diodes ($t_{rr} \approx 2\,\mu\text{s}$) cannot be used; their slow recovery would dump high-voltage spikes back into the $+15\text{ V}$ gate driver logic supply, destroying the IR2110.

---

### Step 4.3: Gate Drive Damping Resistor ($R_g$)

#### Derivation:

The MOSFET gate-to-source capacitance forms an RLC resonant circuit with stray PCB trace inductance ($L_{trace} \approx 20\text{ nH}$). To avoid underdamped voltage ringing that can cause parasitic gate re-triggering:


$$R_{g,min} \ge 2\sqrt{\frac{L_{trace}}{C_{iss}}}$$


For $C_{iss} \approx 800\text{ pF}$, $R_{g,min} \ge 2\sqrt{\frac{20 \times 10^{-9}}{800 \times 10^{-12}}} = 2 \times 5 = 10\,\Omega$.

Peak gate drive current from the IR2110 (limited to $2.0\text{ A}$):


$$I_{gate,peak} = \frac{V_{CC} - V_{diode}}{R_g} = \frac{15\text{ V} - 0.7\text{ V}}{R_g} \le 2.0\text{ A} \implies R_g \ge 7.15\,\Omega$$

#### Hardware Connection:

Selected Component: **$15\,\Omega \text{ or } 22\,\Omega$, $0.25\text{ W}$ Metal Film Resistor**, paired with a reverse-connected **$1\text{N}4148$ diode** in parallel with $R_g$ to provide faster turn-off than turn-on.

---

## 5. Mathematical Synthesis of SPWM & V/f Characteristic

```
      Speed Setpoint
       f_fund (Hz)
            │
            ├──────────────────────────┐
            ▼                          ▼
      [ Gain: 4.6 ]             [ Gain: 1/50 ]
            │                          │
            ▼                          ▼
      V_ref (Volts)             m_a (Modulation Index)
                                       │
                                       ▼
                             Va_ref = m_a * sin(θ)

```

### Step 5.1: The Linear Volts-per-Hertz ($V/f$) Law

#### Derivation:

Faraday's Law governs core flux density in the induction motor:


$$V_{phase} \approx E = 4.44 \times f_{fund} \times N_{turns} \times \Phi_{max}$$


To maximize torque without driving the stator iron laminations into magnetic saturation:


$$\Phi_{max} \propto \frac{V_{phase}}{f_{fund}} = \text{Constant } (K)$$

#### Calculation:

* **Rated Phase Voltage:** $V_{line,rms} = 230\text{ V}$ (Delta connection)


* **Rated Frequency:** $f_{rated} = 50\text{ Hz}$

* **$V/f$ Gain Ratio ($K$):**

$$K = \frac{V_{rated}}{f_{rated}} = \frac{230\text{ V}}{50\text{ Hz}} = 4.60\text{ V/Hz}$$



---

### Step 5.2: Modulation Index ($m_a$) Derivation

#### Derivation:

In a classic 3-phase full-bridge inverter operating in the linear SPWM region ($0 \le m_a \le 1.0$), the fundamental peak phase voltage synthesized is:


$$V_{phase,peak} = m_a \times \frac{V_{dc}}{2}$$


The line-to-line RMS fundamental voltage is:


$$V_{LL,rms} = \frac{\sqrt{3}}{\sqrt{2}} \times V_{phase,peak} = \frac{\sqrt{3}}{\sqrt{2}} \times m_a \times \frac{V_{dc}}{2} = m_a \times \frac{\sqrt{3}}{2\sqrt{2}} \times V_{dc} \approx m_a \times 0.6124 \times V_{dc}$$

Substituting the nominal DC-bus voltage $V_{dc} = \sqrt{2} \times 230 = 325.27\text{ V}$:


$$V_{LL,rms} = m_a \times 0.6124 \times 325.27 = m_a \times 200.0\text{ V}$$

*(Note: Pure sinusoidal SPWM without third-harmonic injection supplies up to $200\text{ V}_{rms}$ in the linear range $m_a = 1.0$. Operating slightly into overmodulation, or shifting to Space Vector PWM, reaches the full $230\text{ V}_{rms}$. For linear operation, scaling $m_a$ linearly with frequency preserves the flux ratio):*


$$m_a(f) = \frac{f_{fund}}{50.0\text{ Hz}}$$

* At $f_{fund} = 50\text{ Hz} \implies m_a = 1.0$
* At $f_{fund} = 25\text{ Hz} \implies m_a = 0.5$

---

## 6. Closed-Loop PI Speed Controller Mathematical Tuning

```
                          Speed Error e(t)
 N_ref (RPM) ──(+)──►( - )────────────────► [ PI Controller ] ──► Commanded f_fund (Hz)
               ▲       ▲                          │
               │       │ Measured Speed           ▼
               │       └────────────────── [ Motor Dynamics ]
               │                              G_p(s)

```

### Step 6.1: Mathematical Transfer Function of the Speed Plant ($G_p(s)$)

#### Derivation:

The rotor mechanical speed dynamics follow Newton's second law:


$$J \frac{d\omega_m}{dt} + F \omega_m = T_e - T_L$$


For small perturbations around the linear operating region of the torque-speed curve ($s \le 0.04$), electromagnetic torque is approximately proportional to slip speed:


$$T_e \approx K_t (\omega_s - \omega_m)$$


where $\omega_s = \frac{2\pi f_{fund}}{p}$ is the electrical synchronous frequency.

Substituting $T_e$:


$$J \frac{d\omega_m}{dt} + (F + K_t)\omega_m = K_t \left(\frac{2\pi}{p}\right) f_{fund}$$

Taking the Laplace transform ($T_L = 0$):


$$\frac{\Omega_m(s)}{F_{fund}(s)} = \frac{K_t \frac{2\pi}{p}}{J s + (F + K_t)} = \frac{\frac{K_t \frac{2\pi}{p}}{F + K_t}}{\left(\frac{J}{F + K_t}\right)s + 1}$$

Converting output from $\omega_m$ ($\text{rad/s}$) to shaft speed $N$ ($\text{RPM}$) via $N = \frac{30}{\pi} \omega_m$:


$$G_p(s) = \frac{N(s)}{F_{fund}(s)} = \frac{K_{plant}}{\tau_m s + 1}$$

#### Numerical Evaluation:

* Synchronous torque slope constant: $K_t \approx \frac{T_{nom}}{\omega_s \times s} = \frac{0.398}{(157.08) \times 0.04} \approx 0.0633\text{ N}\cdot\text{m}/(\text{rad/s})$
* Plant steady-state gain ($K_{plant}$):

$$K_{plant} \approx \frac{120}{2p} \times (1 - s) = \frac{120}{4} \times (1 - 0.04) = 28.80\text{ RPM/Hz}$$


* Mechanical time constant ($\tau_m$):

$$\tau_m = \frac{J}{F + K_t} = \frac{0.0005}{0.0001 + 0.0633} = \frac{0.0005}{0.0634} \approx 0.00788\text{ s} \approx 8.0\text{ ms}$$



---

### Step 6.2: Analytical Tuning of $K_p$ and $K_i$ (Pole-Zero Cancellation)

The parallel PI controller has the transfer function:


$$C(s) = K_p + \frac{K_i}{s} = \frac{K_p s + K_i}{s} = K_p \left(\frac{s + \frac{K_i}{K_p}}{s}\right)$$

The open-loop loop gain is:


$$L(s) = C(s) G_p(s) = K_p \left(\frac{s + \frac{K_i}{K_p}}{s}\right) \left(\frac{K_{plant}}{\tau_m s + 1}\right) = \frac{K_p K_{plant}}{\tau_m} \frac{\left(s + \frac{K_i}{K_p}\right)}{s \left(s + \frac{1}{\tau_m}\right)}$$

#### Pole-Zero Cancellation Criterion:

Set the controller zero to cancel the motor's mechanical pole:


$$\frac{K_i}{K_p} = \frac{1}{\tau_m} = \frac{1}{0.00788} \approx 126.9\text{ rad/s}$$

With this cancellation, the closed-loop transfer function simplifies to a pure first-order system:


$$T_{cl}(s) = \frac{L(s)}{1 + L(s)} = \frac{1}{\left(\frac{\tau_m}{K_p K_{plant}}\right) s + 1} = \frac{1}{\tau_{cl} s + 1}$$


where $\tau_{cl}$ is the desired closed-loop response time.

#### Calculation for Target Settling Time:

Target a closed-loop settling time of $t_s \approx 0.4\text{ seconds}$ without overshoot:


$$\tau_{cl} = \frac{t_s}{4} = \frac{0.4}{4} = 0.10\text{ s}$$

Equating time constants:


$$\tau_{cl} = \frac{\tau_m}{K_p K_{plant}} \implies K_p = \frac{\tau_m}{K_{plant} \times \tau_{cl}} = \frac{0.00788}{28.80 \times 0.10} \approx 0.00273$$


Using the zero-cancellation ratio:


$$K_i = K_p \times \frac{1}{\tau_m} = \frac{0.00788}{28.80 \times 0.10} \times \frac{1}{0.00788} = \frac{1}{28.80 \times 0.10} \approx 0.347$$

#### Tuning Adjustments for Real-World Noise and Sensor Quantization:

In the actual simulation and Arduino implementation, discrete sensor delay and encoder discretization introduce phase lag:

* **Selected Practical $K_p$:** `0.02` (Provides faster dynamic reaction to load steps)
* **Selected Practical $K_i$:** `0.10` (Guarantees zero steady-state error without driving the integrator into oscillatory limit cycles)

---

### Step 6.3: Deceleration Rate & Kinetic Energy Dissipation

#### Derivation:

When commanded to stop from full speed ($1440\text{ RPM}$, $\omega_m = 150.8\text{ rad/s}$), the rotating shaft holds stored kinetic energy:


$$E_{kin} = \frac{1}{2} J \omega_m^2 = \frac{1}{2} (0.0005) (150.80)^2 \approx 5.685\text{ Joules}$$

The maximum energy the $150\,\mu\text{F}$ DC capacitor can store between its nominal voltage ($325.27\text{ V}$) and its absolute surge limit ($400.0\text{ V}$) before venting is:


$$\Delta E_{cap} = \frac{1}{2} C_{dc} \left(V_{max}^2 - V_{dc}^2\right) = \frac{1}{2} (150 \times 10^{-6}) \left(400^2 - 325.27^2\right)$$

$$\Delta E_{cap} = 75 \times 10^{-6} \times (160{,}000 - 105{,}800) = 75 \times 10^{-6} \times 54{,}200 \approx 4.065\text{ Joules}$$

#### Significance & Defense:

Because $E_{kin} (5.685\text{ J}) > \Delta E_{cap} (4.065\text{ J})$, stepping the reference frequency instantly to $0\text{ Hz}$ forces the motor to act as an induction generator. The kinetic energy dumped back into the DC link exceeds the capacitor's absorption capacity, raising the DC bus voltage beyond $400\text{ V}$ and damaging the electrolytic capacitor or MOSFETs.

#### Deceleration Ramp Rate Calculation:

Allow internal motor iron/copper losses and friction ($P_{loss} \approx 25\text{ W}$) to absorb excess energy during deceleration. The minimum safe ramp-down time ($t_{decel}$) is:


$$t_{decel} \ge \frac{E_{kin} - \Delta E_{cap}}{P_{loss}} = \frac{5.685 - 4.065}{25} \approx 0.065\text{ s}$$

Applying an engineering safety margin of $30\times$:


$$t_{decel} = 2.0\text{ to } 3.0\text{ seconds}$$

$$\text{Decel Ramp Rate} = \frac{50\text{ Hz} - 5\text{ Hz}}{2.5\text{ s}} = 18.0\text{ Hz/s}$$

---

## 7. OPAL-RT Simulation Setup & HIL Scaling

```
  Simulated Inverter Output Voltage
           (±325.2V Peak)
                 │
                 ▼
       [ Gain: 0.03077 ]  ──► Attenuates to ±10.0V Peak
                 │
                 ▼
    [ OPAL-RT Analog Out ]
                 │
                 ▼ (Physical DB37 Pin 1 & Pin 20)
       [ Oscilloscope Probe ]  ──► Display matches real-world voltage

```

### Step 7.1: Simulation Step Size Selection ($T_{step}$)

#### Calculation:

* Carrier Frequency: $f_{carrier} = 5000\text{ Hz}$ ($T_{carrier} = 200\,\mu\text{s}$)


* For the numerical solver to reconstruct PWM transitions without phase jitter, there must be at least 10 discrete solver evaluations per carrier cycle:

$$T_{step} \le \frac{T_{carrier}}{10} = \frac{200\,\mu\text{s}}{10} = 20\,\mu\text{s}\quad (20 \times 10^{-6}\text{ s})$$




#### Defense:

$T_{step} = 20\,\mu\text{s}$ matches the execution limits of OPAL-RT target processor cores running under RT-LAB with the `ode4` (Runge-Kutta 4th Order) fixed-step solver. It captures the $5\text{ kHz}$ carrier wave dynamics while avoiding CPU overrun warnings.

---

### Step 7.2: Analog Output Attenuation Gain Calculation

#### Derivation:

The OPAL-RT digital-to-analog converter (DAC) channels (Slot 2 Group B) operate within an absolute output range of $-10.0\text{ V} \text{ to } +10.0\text{ V}$. The model's simulated phase voltage reaches $V_{peak} = \pm 325.27\text{ V}$.

Direct connection would saturate the DAC outputs. An attenuation gain ($K_{scale}$) is inserted prior to the DAC output block:


$$K_{scale} = \frac{V_{DAC,max}}{V_{sim,peak}} = \frac{10.0\text{ V}}{325.27\text{ V}} \approx 0.030744 \approx 0.03077$$

#### Hardware DSO Verification:

* On the external oscilloscope (DSO), set the channel probe attenuation to:

$$\text{DSO Multiplier} = \frac{1}{K_{scale}} = \frac{325.27}{10.0} \approx 32.53\times$$


* Connecting DSO Channel 1 to **Pin 1 (+)** and **Pin 20 (-)** on the OPAL-RT DB37 breakout board displays the reconstructed phase voltage directly in high-voltage engineering units ($325\text{ V}_{peak}$).



---

## 8. Master Parameter Mapping Table

| Parameter / Variable | Derivation / Formula | Exact Calculated Value | Implemented Hardware / Simulink Value | Primary Significance & Justification |
| --- | --- | --- | --- | --- |
| **Motor Apparent Power ($S_{max}$)** | $\sqrt{3} \cdot V_L \cdot I_L$ | $163.33\text{ VA}$ | **`163.3` VA** | Baseline capacity for sizing all upstream power stages. |
| **Base Motor Impedance ($Z_{base}$)** | $V_L^2 / S_{max}$ | $323.88\,\Omega$ | **`324` $\Omega$** | Establishes standard conversion scale for per-unit parameters. |
| **Stator Resistance ($R_s$)** | $0.05 \times Z_{base}$ | $16.19\,\Omega$ | **`16.2` $\Omega$** | Defines stator copper loss and low-speed $I R_s$ voltage drop. |
| **Magnetizing Inductance ($L_m$)** | $1.49 \times L_{base}$ | $1.536\text{ H}$ | **`1.54` H** | Governs air-gap magnetic flux generation. |
| **DC Bus Voltage ($V_{dc}$)** | $\sqrt{2} \times 230\text{ V}$ | $325.27\text{ V}$ | **`325.2` V DC** | Dictates minimum voltage breakdown margins for semiconductors. |
| **DC-Link Capacitor ($C_{dc}$)** | $\frac{S_{max}}{2 f_{rip} V_{dc} \Delta V_{dc}}$ | $154.5\,\mu\text{F}$ | **$220\,\mu\text{F} / 450\text{ V}$** | Holds DC ripple below $5\%$; $450\text{ V}$ rating absorbs decel surges. |
| **Rectifier Diode Rating ($V_{RRM}, I_F$)** | $1.5 V_{dc}, 2 I_{in}$ | $488\text{ V}, 1.42\text{ A}$ | **$600\text{ V} / 2.0\text{ A}$ (KBP206)** | Accommodates utility grid spikes and inrush current without a heatsink. |
| **MOSFET Breakdown ($V_{DSS}$)** | $1.8 \times V_{dc}$ | $585.5\text{ V}$ | **$600\text{ V}$ (STP4NK60Z)** | Blocks high-voltage rail and handles inductive turn-off spikes. |
| **MOSFET Continuous Current ($I_D$)** | $2 \times I_{inrush}$ | $3.48\text{ A}$ | **$4.0\text{ A}$ to $8.0\text{ A}$** | Carries 3-phase motor starting inrush current without overheating. |
| **Gate Resistor ($R_g$)** | $2\sqrt{L_{trace}/C_{iss}}$ | $10.0\,\Omega$ | **$15\,\Omega \text{ to } 22\,\Omega$** | Suppresses gate voltage ringing and prevents spurious turn-on. |
| **Bootstrap Capacitor ($C_{boot}$)** | $Q_{tot} / \Delta V_{boot}$ | $0.17\,\mu\text{F}$ | **$1.0\,\mu\text{F} / 50\text{ V}$ MLCC** | Maintains stable floating high-side gate driver power supply. |
| **$V/f$ Control Ratio ($K$)** | $230\text{ V} / 50\text{ Hz}$ | $4.60\text{ V/Hz}$ | **`4.6` V/Hz** | Keeps magnetic core flux constant; prevents iron saturation. |
| **Carrier Frequency ($f_{carrier}$)** | System Design | $5000\text{ Hz}$<br> | **`5000` Hz ($200\,\mu\text{s}$)**<br> | Reduces acoustic noise while keeping switching losses low. |
| **Proportional Gain ($K_p$)** | $\frac{\tau_m}{K_{plant} \tau_{cl}}$ | $0.00273$ | **`0.02`** | Adjusts dynamic response speed during speed transitions. |
| **Integral Gain ($K_i$)** | $\frac{1}{K_{plant} \tau_{cl}}$ | $0.347$ | **`0.10`** | Removes steady-state RPM offset under load. |
| **Arduino Control Loop ($T_s$)** | Hardware execution rate | $10\text{ ms}$ | **`0.01` s ($100\text{ Hz}$)** | Matches the standard timer interrupt loop of an Arduino Uno. |
| **Ramp-Down Time ($t_{decel}$)** | $\frac{E_{kin} - \Delta E_{cap}}{P_{loss}}$ | $\ge 0.065\text{ s}$ | **`2.5` seconds** | Protects the DC bus from overvoltage during regeneration. |
| **OPAL-RT Step Size ($T_{step}$)** | $T_{carrier} / 10$ | $20\,\mu\text{s}$ | **`20e-6` s ($50\text{ kHz}$)**<br> | Meets Nyquist sampling criteria for PWM pulse capture.

 |
| **HIL DAC Attenuation ($K_{scale}$)** | $10\text{ V} / 325.27\text{ V}$ | $0.03074$ | **`0.03077`**<br> | Scales the $325\text{ V}$ bus to match OPAL-RT's $\pm 10\text{ V}$ analog DAC limits.







1. Load Subsystem (Induction Motor)
| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
|---|---|---|---|---|
| 3-Phase Induction Motor | 60\text{ W} (0.08\text{ HP} / 0.06\text{ kW}), 230\text{ V} (\Delta) / 415\text{ V} (Y), 0.41\text{ A} (\Delta), 1440\text{ RPM}, 50\text{ Hz}, 4-Pole | Frame 56 Squirrel-Cage AC Motor (Tachometric Controls / Oriental Motor equivalent) | 1 | Dynamic mechanical/inductive load; connected in Delta (\Delta) configuration for 230\text{ V} VFD output. |
2. AC Input Rectification & DC-Link Stage
| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
|---|---|---|---|---|
| Bridge Rectifier IC | Single-phase full-bridge, V_{RRM} \ge 600\text{ V}, I_{F(AV)} \ge 2\text{ A} | KBP206, GBU4J, or W10M | 1 | Rectifies 230\text{ V} RMS grid AC to unregulated peak 325.2\text{ V} DC. |
| DC-Link Filter Capacitor | 150\,\mu\text{F} \text{ to } 220\,\mu\text{F}, 400\text{ V} - 450\text{ V DC}, High-ripple electrolytic | Nichicon / Rubycon Radial Aluminum Electrolytic | 1 | Filters the 100\text{ Hz} rectification ripple below 5\% peak-to-peak. |
| DC Bleeder Resistor | 100\text{ k}\Omega \text{ to } 220\text{ k}\Omega, 2\text{ W} Metal Oxide / Ceramic | Flameproof Power Resistor | 1 | Discharges the 325\text{ V} capacitor safely to 0\text{ V} when mains AC is disconnected. |
| Inrush Current Limiter (NTC) | 10\,\Omega \text{ to } 20\,\Omega cold resistance, 2\text{ A} steady-state | NTC 10D-9 or 10D-11 | 1 | Limits initial charging surge current into the uncharged DC capacitor at power-up. |
| High-Frequency Decoupling Cap | 0.1\,\mu\text{F} (100\text{ nF}), 630\text{ V} - 1000\text{ V} Metallized Polypropylene | Film Capacitor (MKP / Box type) | 1 | Sits directly across the DC+ and DC- inverter rails to absorb high-frequency switching spikes. |
3. 3-Phase Inverter Power Stage
| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
|---|---|---|---|---|
| Power MOSFETs | N-Channel, V_{DSS} \ge 500\text{ V} - 600\text{ V}, I_D \ge 4\text{ A} - 8\text{ A}, TO-220 package | STP4NK60Z, IRFBC30, or IRF840 | 6 | Arranged in 3 half-bridge legs to switch the 325\text{ V} bus and synthesize 3-phase AC. |
| Heatsinks | TO-220 clip-on aluminum heat spreaders | Standard finned TO-220 heatsink | 6 | Dissipates conduction and switching thermal losses (< 0.5\text{ W} per device). |
| Mica Insulators & Bushings | Thermal insulating pads + nylon screw bushings | Standard TO-220 mounting kit | 6 | Electrically isolates the MOSFET drain tabs (connected to DC+) from a shared heatsink. |
4. Gate Driver, Bootstrap & Isolation Board
| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
|---|---|---|---|---|
| Gate Driver ICs | High- and Low-Side Driver, floating high-side channel up to 600\text{ V}, 2\text{ A} sink/source | IR2110 or IR2113 (DIP-14) | 3 | Drives high-side and low-side MOSFETs per inverter leg from logic-level PWM. |
| High-Speed Optocouplers | Single-channel high-speed digital isolator, 10\text{ Mbps}, V_{ISO} \ge 2500\text{ V}_{rms} | 6N137 (or HCPL-2601) | 6 | Provides galvanic isolation between the low-voltage controller and high-voltage power ground. |
| Bootstrap Diodes | Ultra-Fast Recovery, V_{RRM} \ge 600\text{ V} - 1000\text{ V}, I_F \ge 1\text{ A}, t_{rr} \le 75\text{ ns} | UF4007 or MUR160 | 3 | Charges the high-side floating bootstrap capacitors when low-side switches conduct. |
| Bootstrap Capacitors | 1.0\,\mu\text{F} \text{ to } 2.2\,\mu\text{F}, 50\text{ V}, Low ESR | Multilayer Ceramic (MLCC) / Tantalum | 3 | Supplies floating gate drive voltage (V_B - V_S) for high-side MOSFET conduction. |
| Gate Resistors (R_g) | 15\,\Omega \text{ to } 22\,\Omega, 0.25\text{ W}, Metal Film | Standard 1/4W Resistor | 6 | Limits gate peak turn-on current and prevents parasitic gate-source ringing. |
| Fast Turn-Off Diodes | Fast Switching Diode, V_R \ge 75\text{ V}, I_F \ge 150\text{ mA} | 1N4148 | 6 | Placed anti-parallel across R_g to bypass the resistor during turn-off for faster discharge. |
| Gate Pull-Down Resistors | 10\text{ k}\Omega, 0.25\text{ W} | Standard 1/4W Resistor | 6 | Tied between Gate and Source to keep MOSFETs safely OFF during controller reset/startup. |
| Optocoupler Current Limiters | 330\,\Omega \text{ to } 470\,\Omega, 0.25\text{ W} | Standard 1/4W Resistor | 6 | Limits forward current (I_F \approx 10\text{ mA}) from microcontroller pins through optocoupler LEDs. |
| Optocoupler Pull-Up Resistors | 1\text{ k}\Omega \text{ to } 4.7\text{ k}\Omega, 0.25\text{ W} | Standard 1/4W Resistor | 6 | Pulls open-collector outputs of the 6N137 up to the +5\text{ V} driver logic rail. |
| IC Bypass Capacitors | 0.1\,\mu\text{F} (100\text{ nF}), 50\text{ V}, Ceramic | 104 MLCC / Ceramic Disc | 9 | Decouples V_{CC} and V_{DD} supply pins on the IR2110 (3x) and optocouplers (6x). |
5. Controller, Sensing & Feedback
| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
|---|---|---|---|---|
| Microcontroller Board | 16\text{ MHz} AVR, Hardware PWM Timers, External Interrupt Pins | Arduino Uno, Nano, or Mega 2560 | 1 | Executes speed setpoint ramps, discrete PI control loop, and SPWM generation. |
| Rotor Speed Sensor | Incremental Optical / Magnetic Rotary Shaft Encoder, 5\text{ V}, Quadrature AB output | Omron E6B2-CWZ6C (100 - 360\text{ PPR}) or Slotted Optical Disc Module | 1 | Measures physical motor shaft speed to feed real-time RPM back into the closed-loop PI controller. |
| Speed Reference Potentiometer | 10\text{ k}\Omega Linear Rotary Potentiometer | B10K Potentiometer with dial knob | 1 | Provides manual target speed adjustment (0\text{ to } 1440\text{ RPM}) via Arduino ADC input. |
| Pushbuttons (Tactile) | SPST Momentary Tactile Switch (6\times 6\text{ mm}) | Standard PCB Tactile Switch | 2 | Manual START / RUN and STOP commands. |
| Status Indicator LEDs | 3\text{ mm} or 5\text{ mm} LEDs (Red, Green, Yellow) | Standard THT LEDs + 330\,\Omega series resistors | 3 | Displays system states: Power ON (Green), Inverter Running (Yellow), Fault/Trip (Red). |
6. Auxiliary Power Supply & Circuit Protection
| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
|---|---|---|---|---|
| Gate Driver DC Supply | +12\text{ V} \text{ to } +15\text{ V DC}, 1\text{ A} regulated | Regulated SMPS Wall Adapter or Linear Supply | 1 | Powers the IR2110 gate driver output stages (V_{CC}). |
| Logic DC Supply | +5\text{ V DC}, 1\text{ A} regulated | LM7805 Linear Regulator or 5 V Step-Down Module | 1 | Powers Arduino logic, 6N137 optocouplers, and rotary optical encoder. |
| AC Mains Input Fuse | Fast-acting / Slow-blow Glass Fuse, 250\text{ V}, 2\text{ A} | 5\times 20\text{ mm} Cartridge Fuse + Panel/PCB Holder | 1 | Overcurrent and short-circuit protection for mains input. |
| DC Bus Fuse | Fast-acting Ceramic Fuse, 500\text{ V DC}, 2\text{ A} | 6.3\times 32\text{ mm} High-Voltage DC Fuse | 1 | Protects the DC bus against inverter shoot-through or MOSFET breakdown. |
7. OPAL-RT Real-Time HIL Interface Hardware
| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
|---|---|---|---|---|
| DB37 Breakout Board | 37-Pin D-Sub Female connector with screw terminals | Standard DB37-M6 Breakout Module | 1 | Connects physical oscilloscope probes to OPAL-RT Slot 2 Group B analog output pins. |
| Analog Scaling Resistors | Precision Metal Film (0.1\% or 1\%), 0.25\text{ W} | 100\text{ k}\Omega and 3.16\text{ k}\Omega (Divider Ratio \approx 0.03077) | 3 sets | Hardware passive attenuator if scaling high external voltages into OPAL-RT analog inputs. |
| Oscilloscope Interconnect | BNC to Alligator Clip / Minigrabber Test Leads | Standard 50\,\Omega BNC Cable | 3 | Connects DB37 Phase A, B, C channels (Pins 1, 2, 3 and grounds 20, 21, 22) to DSO channels. |
8. Mechanical & Wiring Hardware
| Component | Specification / Rating | Recommended Part | Qty | Function in Circuit |
|---|---|---|---|---|
| PCB / Prototyping Board | Double-sided FR4 copper clad perfboard (10\times 15\text{ cm}) or Custom 2-Layer PCB | FR4 Prototype PCB | 1 | Physical substrate for soldering inverter switches, drivers, and isolation. |
| High-Voltage Screw Terminals | 2-Pin and 3-Pin PCB Terminal Blocks, 300\text{ V} - 600\text{ V}, 10\text{ A} rating, 5.08\text{ mm} pitch | Phoenix Contact / Degson Terminal Blocks | 4 | Wire terminals for AC Mains Input, DC Bus monitor, and 3-Phase Motor Out (U, V, W). |
| Hookup Wire | 18\text{ AWG} stranded wire (600\text{ V} rated) + 24\text{ AWG} wire | UL1007 / UL1015 hookup wire | Assorted | 18\text{ AWG} for AC/DC power tracks and motor phases; 24\text{ AWG} for gate drive and logic signals. |

 |

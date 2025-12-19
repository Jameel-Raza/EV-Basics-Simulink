# 🔌 High-Power EV Fast Charger: Closed-Loop CC-CV Control Module

## 1. Project Objective & Technical Rationale
**Objective:** To design and validate a multi-stage power electronics conversion system that transforms 3-phase AC grid power into a regulated DC current/voltage profile specifically for EV battery charging.

### **The "Why" (Technical Drivers)**
* **Safety & Longevity:** Lithium-ion batteries are chemically sensitive. Charging must follow a strict **Constant Current (CC)** phase to prevent overheating, followed by a **Constant Voltage (CV)** phase to prevent cell overpotential.
* **Grid Stability:** Uncontrolled charging creates massive current spikes that can trip protection breakers or sag the grid voltage.
* **Precision Engineering:** Because a battery's internal resistance and Open Circuit Voltage (OCV) change dynamically during a charge cycle, a fixed Duty Cycle (Open Loop) is insufficient. Closed-loop control is mandatory to maintain stability.

---

## 2. Theoretical Foundation: Plant Modeling & Controller Synthesis
To control a system, one must first understand its "physics" mathematically. This section documents how we transformed a collection of hardware components into a predictable, controllable mathematical model.



### **2.1. The Buck Converter as a First-Order Plant**
Our primary goal is to control the **Inductor Current ($I_L$)**. When the IGBT switch is closed, the circuit is governed by Kirchhoff’s Voltage Law (KVL). The voltage across the inductor is the difference between the input DC Link voltage and the battery voltage, minus the resistive losses:

$$V_{in} - V_{batt} = L\frac{di_L}{dt} + R_L i_L$$

To design a controller, we need the **Control-to-Current Transfer Function**, $G_{id}(s)$. This tells us how the inductor current responds to changes in the Duty Cycle ($d$). In the Laplace domain, the plant is modeled as:

$$G_{id}(s) = \frac{I_L(s)}{D(s)} = \frac{V_{in}}{Ls + R}$$

* **The Pole:** The denominator $Ls + R$ reveals a natural "pole" at $s = -R/L$. This represents the physical limit of how fast the current can change due to the inductor's "inertia." For our values ($L=3\text{mH}, R=0.5\Omega$), the pole is at **-166.67 rad/s**.

### **2.2. PI Controller Design & Pole-Zero Cancellation**
We use a Proportional-Integral (PI) controller to provide high-speed reaction and zero steady-state error. The transfer function is:

$$C(s) = K_p + \frac{K_i}{s} = \frac{K_p s + K_i}{s} = K_p \frac{(s + \frac{K_i}{K_p})}{s}$$

**The Engineering Strategy:**
If we leave the plant as it is, the inductor's natural lag will make the system sluggish. We use **Pole-Zero Cancellation** to mathematically align the "Zero" of our controller to sit exactly on top of the "Pole" of the plant.

1.  **Setting the Ratio:** We set $\frac{K_i}{K_p} = \frac{R}{L}$.
2.  **The Result:** The $(s + R/L)$ term in the controller cancels out the $(Ls + R)$ term in the plant. The complex second-order system collapses into a simple, predictable first-order integrator:
    $$L(s) = C(s) \cdot G_{id}(s) = \frac{K_p \cdot V_{in}}{L \cdot s}$$

### **2.3. Determining Cross-Over Frequency ($f_c$)**
The **Cross-Over Frequency** defines the "speed" of the controller.
* **Target Bandwidth:** With a switching frequency ($f_{sw}$) of $5\text{kHz} - 10\text{kHz}$, we targeted a bandwidth ($\omega_c$) of approximately **1000 rad/s** ($f_c \approx 160\text{Hz}$).

At crossover, the magnitude $|L(j\omega_c)|$ must be 1:
$$1 = \frac{K_p \cdot V_{in}}{L \cdot \omega_c} \implies K_p = \frac{0.003 \cdot 1000}{540} \approx \mathbf{0.006}$$

Using our cancellation ratio, $K_i$ is found:
$$K_i = K_p \cdot \frac{R}{L} = 0.006 \cdot \frac{0.5}{0.003} = \mathbf{1.0}$$
*(Final simulation tuning utilized $K_i = \mathbf{1.85}$ to overcome non-ideal voltage drops).*

---

## 3. AC Grid Side Theory: Rectification & Passive Interfacing
The "Front End" of the charger is responsible for converting the high-voltage 3-Phase AC from the grid into a relatively stable DC Link for the Buck converter.

### **3.1. Why Grid Inductors ($L_g$) are Mandatory**
In this module, we placed **0.5mH inductors** in series with each AC phase. These serve several critical functions:
* **di/dt Limiting:** Without these inductors, the current would change instantaneously every time a diode in the bridge conducts, leading to infinite current spikes in the simulation.
* **Harmonic Filtering:** The 3-phase diode bridge is a non-linear load that naturally creates "rabbit-ear" current spikes. The inductors act as a first-order low-pass filter, smoothing these spikes to reduce the Total Harmonic Distortion (THD) injected back into the grid.
* **Voltage Buffering:** They provide a small voltage drop that protects the diode bridge from sudden grid transients or surges.

### **3.2. The 3-Phase Diode Bridge (Passive Front End)**
We utilized a **Universal Bridge** configured with 6 diodes.
* **The Math:** For a 3-phase system with Line-to-Line voltage ($V_{LL}$), the average DC output ($V_{dc}$) is calculated as:
    $$V_{dc} \approx 1.35 \times V_{LL} = 1.35 \times 400V = 540V$$
* **Ripple Frequency:** Because there are 6 pulses per AC cycle, the ripple frequency on the DC Link is **6 times the grid frequency** ($6 \times 50\text{Hz} = 300\text{Hz}$). The large DC Link capacitor ($C_{dc}$) is sized to absorb this ripple.

---

## 4. Implementation Procedure: From Scratch

### **Step 1: Power Stage Assembly**
1.  **AC Source:** Drag a `Three-Phase Source` (400V L-L, 50Hz).
2.  **Inductive Input:** Place three `Series RLC Branch` blocks (0.5mH) between the Grid and Rectifier.
3.  **Rectification:** Connect a `Universal Bridge` (set to `Diodes`).
4.  **DC Link:** Add a large capacitor (e.g., $2200\mu F$) to stabilize the output of the rectifier.
5.  **Buck Converter:**
    * Place an `IGBT` switch as the high-side controller.
    * Place a `Diode` in anti-parallel as the freewheeling path.
    * Place a $3\text{mH}$ Inductor in series with the battery.

### **Step 2: Feedback & Logic Construction**
1.  **Sensing:** Add `Current Measurement` (after inductor) and `Voltage Measurement` (across battery).
2.  **Signal Delay:** Pass feedback signals through a `Unit Delay` block to prevent algebraic loops and model MCU computation lag.
3.  **The Dual-Loop Brain:**
    * **CC Loop:** $(Ref\_I - Feed\_I) \to \text{PI Controller}$.
    * **CV Loop:** $(Ref\_V - Feed\_V) \to \text{PI Controller}$.
    * **Arbitration:** Feed both outputs into a `Min` block. This ensures that if the battery voltage hits the target, the voltage loop "wins" and throttles current.

---

## 5. Challenges Faced & Engineering Solutions

### **Challenge 1: The "Ghost Current" Phenomenon**
* **Symptom:** Charging occurred even at `0` command.
* **Solution:** Applied **Gain (0.5) and Bias (0.5)** to the Sawtooth to shift it from `-1 to 1` to a strictly unipolar **0 to 1** range output.

### **Challenge 2: Integrator Windup Paralysis**
* **Symptom:** Controller "frozen" at 0 after startup spikes.
* **Solution:** Enabled **Anti-Windup Clamping** in the PI block to stop error accumulation at the 0.95 saturation limit.

### **Challenge 3: The Physics Potential Barrier**
* **Symptom:** PI output positive (0.4), but current at 0A.
* **Root Cause:** $V_{in} \times 0.4 \approx 216V < V_{batt} (350V)$.
* **Solution:** Ensured $V_{out} > V_{batt}$ during operation, confirming that control logic cannot violate Kirchhoff’s Voltage Law.

---

## 6. Key Learnings & Observations
* **Passive vs. Active Front Ends:** We observed that while diodes are simple, they offer no control over the DC Link voltage, making the system entirely dependent on the grid's nominal value.
* **Inductor Sizing:** The input inductors are critical; they prevent simulation instability by providing impedance to limit high-frequency current changes.
* **Digital Realism:** Using Unit Delays and discrete solvers is essential for simulating behavior that can be flashed onto real microcontrollers.

---

## 7. Project Status & Future Scope
* **Module 1 (Complete):** Success in passive rectification and active DC-DC charging.
* **Module 2 (Upcoming):** Replacing the diode bridge with an **Active Front End (AFE)**. This will utilize a **PLL (Phase Locked Loop)** and **DQ Transformation** to achieve unity power factor and bidirectional V2G capability.
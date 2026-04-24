# Automatic Night Light — Analog Electronics Project

A practical and simulation-based analysis of an automatic night light circuit built around the µA741 op-amp configured as an inverting Schmitt trigger comparator. The circuit automatically turns on an LED when ambient light drops below a threshold, controlled by an LDR voltage divider and a 100kΩ potentiometer.


---

## Table of contents

- [Circuit overview](#circuit-overview)
- [Component list](#component-list)
- [Circuit operation](#circuit-operation)
- [Part 1 — Practical implementation](#part-1--practical-implementation)
- [Part 2 — Simulation & analysis](#part-2--simulation--analysis)
- [Threshold voltage analysis](#threshold-voltage-analysis)
- [Key results](#key-results)
- [Files](#files)

---

## Circuit overview

The circuit senses ambient light using an **LDR** and compares the resulting voltage against a reference set by a **100kΩ potentiometer**. The µA741 op-amp acts as a comparator — when it gets dark, its output swings high, driving a **2N2222A NPN BJT** into saturation and turning on the LED.

**Supply voltage:** +9V  
**Simulator used:** LTspice

```
+9V
 │
LDR             ← Resistance increases in darkness
 │
 ├──────── VB (inverting input of µA741)
 │
100kΩ pot       ← Sets reference voltage threshold

µA741 (Schmitt trigger with 270kΩ feedback)
  output → VC → 3× 1N4148 diodes → 4.7kΩ → VD → 2N2222A base → LED
```

---

## Component list

| Component | Value / Part | Function |
|---|---|---|
| Op-amp | µA741 | Schmitt trigger comparator |
| BJT | 2N2222A (NPN) | LED switch driver |
| Diodes | 3× 1N4148 | Signal path from op-amp to transistor base |
| LDR | Variable (light-dependent) | Light sensor — resistance rises in darkness |
| Potentiometer | 100kΩ | Adjustable reference voltage threshold |
| Feedback resistor | 270kΩ | Sets Schmitt trigger hysteresis |
| Resistors | 47kΩ, 10kΩ, 100kΩ, 4.7kΩ, 220Ω | Voltage dividers, current limiting |
| LED | Standard | Output indicator |

---

## Circuit operation

**1. Light detection**  
The LDR and fixed resistors form a voltage divider. In bright light, LDR resistance is low → VB (inverting input) is low. In darkness, LDR resistance increases → VB rises.

**2. Reference voltage**  
The potentiometer sets a reference at the non-inverting input of the op-amp. This threshold determines the light level at which the circuit switches.

**3. Comparator action**  
When VB exceeds the reference (darkness): op-amp output VC swings to +Vsat (~8.7V). When VB drops below reference (bright light): VC swings to −Vsat (~0.4V). The 270kΩ feedback resistor creates **hysteresis** (Schmitt trigger behavior) — preventing oscillation near the threshold.

**4. Transistor switching**  
High VC → current flows through 3× 1N4148 diodes and 4.7kΩ resistor → 2N2222A base driven into saturation → LED turns on.  
Low VC → transistor cuts off → LED off.

---

## Part 1 — Practical implementation

The circuit was built on a breadboard and verified as follows:

- Potentiometer adjusted until LED turns on in darkness
- Potentiometer resistance to ground measured at the calibration point
- Node voltages VA, VB, VC, VD, VE measured with a multimeter in both conditions:

| Condition | LDR state | LED |
|---|---|---|
| Dark | High resistance | ON |
| Light | Low resistance | OFF |

LDR resistance was measured directly in both conditions to quantify the difference.

---

## Part 2 — Simulation & analysis

All simulations performed in LTspice with the LED replaced by a 1N4148 diode for compatibility.

### LDR sweep — node voltages

The circuit was simulated at four LDR values to observe how increasing LDR resistance (darker conditions) affects node voltages and LED state:

| LDR value | VB (mV) | VC (V) | LED state |
|---|---|---|---|
| 10kΩ (bright) | 386.87 mV | ~0V (low) | OFF |
| 100kΩ | 573.03 mV | 8.613V (high) | ON |
| 400kΩ | 150.44 mV | 8.613V (high) | ON |
| 600kΩ | 100.86 mV | 8.613V (high) | ON |

At LDR = 10kΩ (bright), the op-amp output is low and the LED is off. As LDR resistance increases past the threshold, the output switches to +Vsat and the LED turns on.

### VPWL simulation (Fig. 2)

The LDR, potentiometer, and 9V supply are replaced by a triangle VPWL source (0V → 9V → 0V over 20ms) to sweep VA continuously and visualize the Schmitt trigger switching behavior:

- **VC(t)** — square wave: high when VPWL < lower threshold, low when VPWL > upper threshold
- **VE(t)** — complementary square wave (inverted): LED on when VE is high

The VC(t) plot clearly shows the two threshold voltages where the op-amp output switches state.

---

## Threshold voltage analysis

### Estimated from simulation
By reading the VPWL values at the transition points on the VC(t) waveform:

- **Upper threshold V_ut** ≈ 3.624V (VC switches from high to low)
- **Lower threshold V_lt** ≈ 1.23V (VC switches from low to high)

### Hand calculation

Using the voltage divider formed by the 100kΩ and 270kΩ feedback resistors with +Vsat = 8.7V and −Vsat = 0.4V:

```
Upper threshold:
  I = (8.7 - 1.378) / (8.2k + 100k + 270k) = 0.0188 mA
  V_ut = (8.7 - I × 270k) = 3.624V  ✓

Lower threshold:
  I = (0.4 - 1.378) / (8.2k + 100k + 270k) = -3.1 × 10⁻³ mA
  V_lt = (0.4 - I × 270k) = 1.23V  ✓
```

**Simulation and hand calculation match** — the small differences are due to rounding and component model approximations in LTspice.

---

## Key results

- The circuit correctly switches the LED on in darkness and off in light
- The Schmitt trigger hysteresis (270kΩ feedback) prevents oscillation near the switching threshold
- Upper threshold: **3.624V** · Lower threshold: **1.23V** — confirmed by both simulation and hand calculation
- The LDR threshold can be adjusted by changing the potentiometer setting

---

## Files

```
automatic-night-light/
├── project_file.pdf     # Original project specification and circuit diagrams
└── report.pdf           # Full project report with breadboard photo, simulations, and hand calculations
```

> Simulation was performed in **LTspice**. The `.asc` schematic file was not included in the submission — the report PDF contains all simulation screenshots.

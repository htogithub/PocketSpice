# PocketSpice

A mobile-first electrical circuit simulator that runs in the browser. Draw a schematic with your finger, tap **Run**, and the voltage at every node and the current through every component appear right on the drawing.

The app is a single HTML file: nothing to install, no account required. On iPhone, open the website link from the About section in Safari, then tap Share → "Add to Home Screen" to use it like a native app.

> The interface is available in English and French.

---

## Features

### Touch schematic editor
- Place components from the toolbar onto a snapping grid.
- Draw wires by dragging from one terminal to another; right-angle bends are created automatically.
- Connections are detected automatically:
  - a wire ending on a terminal or in the middle of another wire connects to it (T-junction);
  - a black dot marks every junction;
  - wires that cross without a dot are not connected.
- Unconnected terminals and wire ends are flagged with a small hollow circle.
- Drag components to move them: attached wires follow and stay orthogonal.
- Rotate in 90° steps from the component panel or the floating rotate button; mirror transistors.
- Navigation: drag to pan, pinch to zoom, one button to fit the whole circuit.
- Undo / redo (up to 120 steps).
- Component panel: name, value and model parameters. The interpreted value is shown live (for example `4.7k` = 4.7 kΩ).
- Renaming: edit a component's Name field, or tap a wire to give its node a name.
- Net labels: two labels with the same name are connected; `0` or `gnd` means ground.

### Components
| Component | Editable parameters |
|---|---|
| Resistor | R |
| Capacitor | C (open circuit in DC) |
| Inductor | L (short circuit in DC) |
| Voltage source | V |
| Current source | I |
| Diode | Standard, 1N4148, red LED or Zener; Is, n, Zener voltage |
| Bipolar transistor | NPN or PNP; β, Is, Early voltage VAF |
| MOSFET | NMOS or PMOS; \|Vth\|, β = Kp·W/L, λ |
| Ground | — |
| Net label | name |

Two extra elements are available in netlists: E (voltage-controlled voltage source) and G (voltage-controlled current source).

### Simulation: DC operating point (`.op`)
- **Circuit solving.** Modified nodal analysis (MNA) with Newton-Raphson iteration for nonlinear devices.
- **Robust convergence:**
  - junction voltage limiting (SPICE `pnjlim` method);
  - step limiting for MOSFETs;
  - automatic fallback to GMIN stepping, then to source stepping.
- **Blocking errors, explained in plain language:**
  - missing ground;
  - loop of voltage sources or inductors;
  - shorted source;
  - unreadable value or zero resistance;
  - duplicate component name.
- **Warnings:** unconnected pin, node without a DC path to ground, ignored netlist command.

### Results
- **On the schematic:**
  - green, multimeter-style tags show the voltage of each node with respect to ground;
  - orange text shows the current through each component, with an arrow giving its actual direction.
  - Tags are placed automatically to avoid overlaps, and one button hides them.
- **Component panel (tap a component after Run):**
  - voltage, current and power;
  - for a bipolar transistor: IC, IB, IE, VBE, VCE, IC/IB ratio and operating region (cut-off, active, saturation, reverse active);
  - for a MOSFET: ID, VGS, VDS, VGS − Vth and operating region (cut-off, triode, saturation).
- **Results tab:**
  - full list of node voltages and of every component's quantities;
  - one button copies everything as plain text.

### SPICE netlist
- The netlist is generated automatically from the schematic and can be copied into other SPICE simulators.
- You can also paste a standard SPICE netlist and simulate it directly.
- A hand-edited netlist is never overwritten without asking. A dedicated button rebuilds it from the schematic.

### Interface
- Designed for a phone screen, also usable on a computer:
  - mouse wheel to zoom;
  - Delete to remove;
  - R to rotate;
  - Ctrl/Cmd + Z to undo;
  - Esc to deselect.
- Light, dark or automatic theme, and English or French interface, chosen from the menu and remembered.
- The circuit and the netlist are saved automatically in the browser.
- Five ready-to-run examples:
  - voltage divider;
  - LED with series resistor;
  - Zener regulator;
  - common-emitter bipolar stage;
  - common-source MOSFET stage.
- Built-in help: gestures, sign conventions and models.

---

## Quick start
1. Pick a component in the bottom toolbar, then tap the sheet to place it.
2. Tap the component to set its value.
3. With the **Wire** tool, drag from one terminal to another to connect components.
4. Add a **Ground**.
5. Tap **Run**.

---

## Supported netlist syntax

```
* comment                     ; end-of-line comment
R1 in out 4.7k
C1 a 0 100n
L1 a b 10u
V1 in 0 12
V2 a b DC 5
I1 0 a 1m                     ; current flows from 0 to a through the source
D1 anode cathode D            ; built-in models: D, 1N4148, LED, ZENER
Q1 c b e NPN                  ; or PNP; a 4th (substrate) node is accepted and ignored
M1 d g s NMOS                 ; or PMOS; a 4th (bulk) node is accepted
E1 out 0 inp inn 1e5          ; voltage-controlled voltage source
G1 out 0 inp inn 1m           ; voltage-controlled current source
.model DZ D(Is=1e-14 N=1 BV=6.2)
.model Q2N3904 NPN(Is=6.7f Bf=416 Vaf=74)
.model MN NMOS(Vto=0.7 Kp=100u Lambda=0.02)
M2 d g s s MN W=20u L=1u
+ ; a line starting with + continues the previous one
.op
.end
```

- **Suffixes:** `f p n u (µ) m k meg g t`. As in SPICE, `m` = milli and `meg` = mega: `1M` means 1 milli.
- **Sources:** `SINE(...)`, `PULSE(...)`, `EXP(...)` and `PWL(...)` are accepted, using their value at t = 0.
- **Commands ignored with a warning:** `.tran`, `.ac`, `.dc`, `.step`, `.meas`, `.param`, `.include`.
- **`.model` parameters:** only those used by the models are read (Is, N, BV, IBV, Bf, Br, Vaf, Vto, Kp, Lambda). Others (Cjo, Rs, etc.) are silently ignored.

---

## Device models

Thermal voltage: VT = 25.865 mV (27 °C).

**Diode**
```
I = Is · (exp(V / (n·VT)) − 1)
Zener (if BV > 0): I −= IBV · exp(−(V + BV) / VT), with IBV = 1 mA
```

**Bipolar transistor (Ebers-Moll, with Early effect)**
```
IF = Is · (exp(VBE/VT) − 1)        IR = Is · (exp(VBC/VT) − 1)
IC = (IF − IR) · (1 − VBC/VAF) − IR/BR
IB = IF/BF + IR/BR
```
VAF = 0 disables the Early effect.

**MOSFET (level 1, bulk tied to source)**
```
Cut-off     VGS ≤ Vth                ID = 0
Triode      VDS < VGS − Vth          ID = β · ((VGS − Vth)·VDS − VDS²/2) · (1 + λ·VDS)
Saturation  VDS ≥ VGS − Vth          ID = β/2 · (VGS − Vth)² · (1 + λ·VDS)
```
with β = Kp · W/L. PMOS devices use the same equations with VSG, VSD and |Vth|.

**Built-in models**
| Name | Type | Parameters |
|---|---|---|
| `D` | diode | Is = 10 fA, n = 1 |
| `1N4148` | small-signal diode | Is = 2.52 nA, n = 1.752 |
| `LED` | red LED | Is = 1 aA, n = 2 (about 1.9 V at 10 mA) |
| `ZENER` | Zener diode | Is = 10 fA, n = 1, BV = 5.1 V |
| `NPN`, `PNP` | bipolar | Is = 10 fA, β = 100, BR = 1, VAF = ∞ |
| `NMOS`, `PMOS` | MOSFET | \|Vth\| = 1 V, Kp = 2 mA/V², λ = 0 |

A `.model` that omits some parameters uses the standard SPICE defaults (for example IS = 1e-16 for a bipolar transistor, VTO = 0 and KP = 20 µA/V² for a MOSFET).

---

## Sign conventions
- **Voltage source:** "supplied current" is the current leaving the + terminal. A negative value means the source is absorbing energy.
- **Current source:** current leaves through the arrow tip.
- **Resistor, inductor, diode:** current is shown as a positive value with its actual direction (for example "in → out").
- **Power:** dissipated power for passive components, supplied power for sources.

---

## Limitations
- **Analysis:** only the DC operating point is available. No transient (`.tran`), AC (`.ac`) or DC sweep (`.dc`) analysis yet.
- **Models:** these are simplified teaching models. They ignore internal capacitances, series resistances and temperature, and do not replace full manufacturer models.
- **Netlist:** no subcircuits (`X`), no `.param` and no `{...}` expressions. A pasted netlist can be simulated but does not redraw the schematic.
- **Storage:** circuits are saved only in the browser of the device you use.

---

## How it works
- A single `index.html` file of about 100 KB: plain HTML, CSS and JavaScript, with no framework and no build step.
- The schematic is rendered as SVG; gestures (drag, pinch) use Pointer Events.
- The simulation engine is written in JavaScript:
  - modified nodal analysis;
  - Gaussian elimination with partial pivoting;
  - Newton-Raphson with junction limiting;
  - GMIN stepping and source stepping as fallbacks.
- The only external resource is the Barlow font (Google Fonts). Offline, the app falls back to the system font.

## Host your own copy
1. Create a public repository and upload `index.html`.
2. In **Settings → Pages**, choose "Deploy from a branch", branch `main` and folder `/ (root)`.
3. After a minute or two, the app is live at `https://username.github.io/repository-name/`.

## Roadmap
- Transient analysis with waveform plots.
- AC analysis and Bode plots.
- DC sweep.
- Ideal op-amp and controlled sources in the schematic editor.

# The sub-threshold slew-rate enhancement (SRE), explained

[← Back to README](../README.md)

## 1. The problem it solves

The pass transistor **Mp** is very large (m = 600), so its gate (V<sub>G</sub>) behaves like a big capacitor. To change the current Mp delivers, something has to charge or discharge that gate.

That job belongs to the **error amplifier (EA)**, which runs on **nano-amps** to keep the regulator's quiescent current below 1 µA. Charging a large gate with nano-amps is slow, like filling a bucket with an eyedropper. When the load jumps from 100 µA to 100 mA in 500 ns, V<sub>G</sub> cannot follow in time, and V<sub>OUT</sub> collapses. Without the SRE, simulation shows an undershoot of **≈ 710 mV**.

## 2. The idea

Adding current to the EA would break the quiescent-current budget, so a **helper circuit** is added instead. It:

1. sleeps most of the time and draws almost no current (≈ 16 nA here),
2. wakes up only when V<sub>OUT</sub> changes **suddenly**,
3. gives V<sub>G</sub> a strong, fast push in the right direction,
4. then goes back to sleep, leaving the EA to handle accurate regulation.

The EA is the careful driver; the SRE is a reflex.

## 3. Detecting a sudden change: the coupling capacitors

V<sub>OUT</sub> is connected to the SRE inverter gates **through capacitors**. A capacitor passes changes but blocks steady DC: a sudden 100 mV dip on V<sub>OUT</sub> appears almost instantly on the gate, while a constant 1.0 V produces nothing.

Each gate is also held at a bias voltage through a GΩ pseudo-resistor (section 5). The capacitor and resistor together form a **high-pass filter** with τ = RC ≈ 1.7 GΩ × 1 pF ≈ 1.7 ms. Fast load steps pass through and trigger the SRE; slow drift is ignored and left to the EA.

## 4. Amplifying and steering: two inverters

| Stage | Devices | Input | Output |
|---|---|---|---|
| 1 | Mp10 / Mn11 | V<sub>OUT</sub> via capacitors | Stage 2 via capacitors |
| 2 | Mp11 / Mn12 | Stage 1 output | **V<sub>G</sub>** |

For an undershoot: V<sub>OUT</sub> drops ⬇ → stage 1 output rises ⬆ → stage 2 (Mn12) pulls V<sub>G</sub> down hard ⬇ → Mp conducts more → V<sub>OUT</sub> recovers ⬆. For an overshoot every arrow reverses, and Mp11 pulls V<sub>G</sub> up.

**Why two inverters:** each one flips the signal, so two flips make V<sub>G</sub> move in the **same** direction as V<sub>OUT</sub>. That is what the correction needs, because Mp is itself inverting (lower gate → more current). With a single inverter, the SRE would push the wrong way.

## 5. Gate-source-connected (GSC) pseudo-resistors and the bias

| GSC device | Holds the gate of | At |
|---|---|---|
| Mp12 / Mp13 | Mp10 / Mp11 (PMOS) | BIAS1 = 960 mV |
| Mp14 / Mp15 | Mn11 / Mn12 (NMOS) | BIAS2 = 240 mV |

A PMOS with its gate tied to its source has V<sub>GS</sub> = 0. It is essentially off and leaks only a tiny current, so it acts as a **GΩ resistor in a few µm²**. A poly resistor of that value would be larger than the whole regulator.

The bias sets **how sleepy** the SRE is. A 240 mV gate drive is below threshold, so the inverters sit in **sub-threshold**, where current grows roughly 10× for every ~80 mV of extra gate drive:

- **at rest:** nano-amps (asleep),
- **during a 100 mV V<sub>OUT</sub> blip:** current jumps more than 10× (awake).

This is how the SRE gets a tiny idle current and a strong kick at the same time.

## 6. When the helper becomes a problem

The SRE forms its **own feedback loop** (V<sub>OUT</sub> → capacitors → inverters → V<sub>G</sub> → Mp → V<sub>OUT</sub>), parallel to the main loop through the EA. At the original bias (300 mV) this loop was too strong around 1.5 MHz, exactly where the main loop's phase had already reached −180°. The two loops together oscillated.

Lowering the bias to 240 mV reduced the SRE's gain enough to keep the combined system stable. The undershoot rose only from ≈ 265 mV to ≈ 310 mV, still less than half of the ≈ 710 mV without the SRE. The full analysis is in [stability-investigation.md](stability-investigation.md).

# The "Level-Fill" Thermal Sandwich Guide

In a plastic enclosure like the **Titan Case**, the **DeepX DX-M1** can reach 60°C+ quickly without direct contact with the system's airflow. This guide explains how to create a "Thermal Sandwich" to bridge the NPU to the LattePanda’s active cooling system.

---

## 1. The Theory
The goal is to create a solid stack of thermally conductive material that fills the physical gap between the **DX-M1 chip** and the **underside of the LattePanda 3 Delta fan housing**.

* **Bottom Layer:** DeepX DX-M1 NPU.
* **Middle Layer:** 1.5mm or 2.0mm High-Performance Thermal Pad (12.8 W/mK).
* **Top Layer:** The LattePanda's integrated metal heat shield/fan assembly.

---

## 2. Bill of Materials
To replicate this setup, you need:
* **Thermal Pad:** 12.8 W/mK silicone pad (cut to 20mm x 20mm).
* **Kapton Tape:** To insulate surrounding surface-mount components on the DX-M1.
* **90° USB Adapter:** To ensure the **Transcend 430S** external enclosure doesn't block the case's exhaust vents.

---

### 3. Assembly Steps

The "Thermal Sandwich" uses a tiered approach to ensure 100% surface contact while protecting the structural integrity of the M.2 board.

#### Step A: Component Isolation
Apply Kapton tape over the small surface-mount components surrounding the main DeepX silicon. This prevents thermal pad oils from seeping into the traces and provides a basic electrical safety barrier.

#### Step B: The "Valley-Fill" Technique
Do not use a single thick pad, as this can cause air pockets or uneven pressure.
* **Layer 1 (Recesses):** Cut a 1mm thermal pad into small pieces and apply them specifically over the "deeper" areas of the NPU board, avoiding the high points of the chips.
* **Layer 2 (The Cap):** Lay a single 0.5mm long strip of thermal pad across the entire assembly. This acts as a tension-spreader to create a flat contact surface for the heatsink.

#### Step C: Heatsink & Tensioning
Apply the heatsink over the completed pad stack. To prevent the DX-M1 PCB from bowing from the high point of the main chip:
* **Rubber Band Tensioning:** Place two high-temperature rubber bands (or silicone O-rings) on both near-ends of the main chip. 
* **Purpose:** This provides constant downward pressure for thermal transfer while ensuring the M.2 board stays perfectly flat, avoiding stress on the gold finger connectors.

#### Step D: Airflow Clearance
Ensure the 90° "Up" adapter for your Transcend 430S external enclosure is oriented so the drive sits vertically. This prevents the enclosure from acting as a "heat wall" for the Titan Case exhaust vents.

---

## 4. Expected Results
With the "Thermal Sandwich" applied, the DX-M1 should stay within the 55°C - 62°C range during 24/7 detection. Without it, the NPU may shut down during peak summer ambient temperatures.

---

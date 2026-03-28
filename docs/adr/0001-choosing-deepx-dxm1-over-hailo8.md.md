# ADR-0001: Hardware Selection — DeepX DX-M1 vs. Hailo-8

This Architectural Decision Record (ADR) outlines the technical rationale for selecting the **DeepX DX-M1** AI Accelerator over the **Hailo-8 (HM218B1C2FA)** for the **LattePanda 3 Delta** single-board computer (SBC).

---

## 1. Context & Constraints
The **LattePanda 3 Delta (Intel N5105)** is a compact but resource-constrained SBC. To enable high-performance AI vision (25+ TOPS), we must navigate significant hardware limitations:
* **PCIe Topology:** The Intel Jasper Lake platform has limited PCIe lanes. Utilizing the M.2 M-Key slot for an NPU requires a **"SATA Sacrifice"**—disabling the internal SATA controller to reallocate PCIe Root Ports.
* **Storage Conflict:** Disabling the SATA controller renders the internal **M.2 B-Key slot unusable** for SATA SSDs.
* **Thermal Environment:** The project uses the **Titan Case** (ABS plastic), which traps heat more than aluminum enclosures.

---

## 2. Decision: DeepX DX-M1
The **DeepX DX-M1** was selected as the optimal "balanced" partner for this specific SBC.

### 2.1 Storage Strategy: The "Hybrid-External" Boot
Because the DX-M1 occupies the only NVMe-capable slot and the SATA controller is disabled, the primary high-speed storage has moved to a **"Hybrid-External"** configuration:
* **Drive:** Transcend 430S 256GB M.2 SATA (DRAM Cache Enabled).
* **Interface:** Connected via an **external USB 3.2 enclosure** to the 10Gbps port.
* **Benefit:** This offloads thermal stress from the motherboard and reserves the **64GB eMMC** for a lightweight recovery OS.

### 2.2 Memory & Host Relief
* **DX-M1 Advantage:** Features **4GB of dedicated LPDDR5** memory. 
* **Impact:** On an 8GB RAM system, the DX-M1 stores model weights internally. The **Hailo-8** lacks dedicated large-scale memory and must "stream" weights from the host RAM, which would starve the Ubuntu 24.04 OS.

### 2.3 Thermal & Power Stability
* **Efficiency:** The DX-M1 operates between **2W – 5W**. In the plastic **Titan Case**, it maintains a stable temperature (approx. **61°C**), whereas the **Hailo-8** (up to 8.2W) risks thermal throttling.
* **Signal Integrity:** The DX-M1 is validated to run reliably at **PCIe Gen2 x2** speeds on this board, avoiding the DMA header overflows found at Gen1 speeds.

---

## 3. Comparison Summary

| Feature | DeepX DX-M1 | Hailo-8 |
| --- | --- | --- |
| Power Profile	| 2W - 5W (Stable) | Up to 8.2W (Aggressive) |
| Onboard Memory | 4GB LPDDR5 (Host Relief) | Internal SRAM Only |
| Thermal Fit | Ideal for Plastic Case | Requires Metal Heatsink |
| Storage Impact | Requires External USB Boot | High Host RAM Overhead |

---

## 4. Consequences
* **BIOS Requirement:** Must set PCIe speed to **Gen2** and **Disable SATA Controller**.
* **Hardware Mod:** Requires an external enclosure for the **Transcend 430S** storage drive to maintain the "Hybrid-External" boot strategy.
* **Software:** Utilizes the **DXNN SDK** and **DX-RT** runtime.

**Status:** Accepted and Validated.

---

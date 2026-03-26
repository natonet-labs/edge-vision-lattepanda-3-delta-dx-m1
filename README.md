[![DeepX DX-M1](https://img.shields.io/badge/DEEPX%20DX--M1-25%20TOPS-blue?logo=ai&logoColor=white)](https://developer.deepx.ai)
[![LattePanda](https://img.shields.io/badge/LattePanda-3%20Delta-red?logo=intel&logoColor=white)](https://www.lattepanda.com/lattepanda-3-delta)
[![PCIe](https://img.shields.io/badge/Interface-PCIe%20Gen2%20x2-lightgrey?logo=pci-express&logoColor=white)](https://github.com/your-username/your-repo#3-bios-configuration)

# The LattePanda AI Vision Odyssey: DeepX DX-M1 Integration

This repository is a comprehensive log of the adventure to enable high-performance **AI Vision** on the **LattePanda 3 Delta 864**. What started as a hardware challenge-battling the "Jasper Lake" PCIe lane limitations-has evolved into a full-stack edge intelligence project.

## **Why this project exists**

Edge AI on small-form-factor PCs is often plagued by documentation gaps. This repository serves as a "living document" for developers who want to push the LattePanda 3 Delta beyond its standard limits and into the realm of professional AI vision.

## The Vision

The goal is to transform a low-power SBC into a 25 TOPS AI powerhouse capable of real-time vision tasks, specifically a **Custom Driveway Counter Dashboard**. This project bridges the gap between raw silicon and actionable data, utilizing:

  * **Object Detection:** Leveraging the DeepX DX-M1 for high-speed inference.
  * **Edge Analytics:** Processing metrics locally on the N5105 CPU.
  * **Visual Output:** Displaying hourly stats and battery health on a **Pimoroni Inky Frame 7.3"**.
  * **Cloud Synchronization:** Hosting dashboards and metrics via **Cloudflare Workers**.

-----

## The Adventure Roadmap

### **Phase 1: The Hardware War (Complete)**

The initial hurdle: making a 25 TOPS NPU visible to a board that only has 8 PCIe lanes.

  * **The "SATA Sacrifice":** Disabling the SATA controller to reallocate Root Ports.
  * **Signal Integrity:** Navigating BIOS settings (Gen2 locking) to stabilize the PCIe handshake.
  * **Thermal Engineering:** Creating a "Level-Fill" thermal sandwich to keep the DX-M1 cool in a Titan Case.

### **Phase 2: Vision & Inference (In Progress)**

Deploying the vision models and the **DXNN SDK**.

  * Compiling ONNX models to DXNN format via **DX-COM**.
  * Optimizing detection scripts for the driveway counter app.

### **Phase 3: The Dashboard & Ecosystem**

Closing the loop between the camera and the user.

  * **Cloudflare Integration:** Sending hourly metrics to the cloud.
  * **Inky Frame UI:** Designing a battery-optimized e-paper dashboard for real-time alerts.

-----

## Stack Overview

| Layer | Component |
| :--- | :--- |
| **SBC** | LattePanda 3 Delta 864 (Intel N5105) |
| **Accelerator** | DEEPX DX-M1 (25 TOPS INT8) |
| **OS** | Ubuntu 24.04 LTS (on 64GB eMMC) |
| **Frontend** | Pimoroni Inky Frame 7.3" (Deep Sleep Optimized) |
| **Backend** | Cloudflare Workers & Metrics Dashboard |

-----

## Key Documentation

  * **[Setup & Hardware Validation Guide](docs/SETUP_GUIDE.md):** The definitive guide to BIOS, PCIe topology, and driver installation.
  * **Application Logic:** (Coming Soon) Detailed breakdown of the driveway counter Python scripts.
  * **The "Thermal Sandwich":** (Coming Soon) Mechanical blueprints for cooling the DX-M1.

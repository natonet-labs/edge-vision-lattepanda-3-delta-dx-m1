[![DeepX DX-M1](https://img.shields.io/badge/DEEPX%20DX--M1-25%20TOPS-blue?logo=ai&logoColor=white)](https://developer.deepx.ai)
[![LattePanda](https://img.shields.io/badge/LattePanda-3%20Delta-red?logo=intel&logoColor=white)](https://www.lattepanda.com/lattepanda-3-delta)
[![PCIe](https://img.shields.io/badge/Interface-PCIe%20Gen2%20x2-lightgrey?logo=pci-express&logoColor=white)](https://github.com/your-username/your-repo#3-bios-configuration)

# The LattePanda AI Vision Odyssey: DeepX DX-M1 Integration

This repository is a comprehensive log of the adventure to enable high-performance **AI Vision** on the **LattePanda 3 Delta 864**. What started as a hardware challenge-battling the "Jasper Lake" PCIe lane limitations-has evolved into a full-stack edge intelligence project.

## **Why this project exists**

Edge AI on small-form-factor PCs is often plagued by documentation gaps. This repository serves as a "living document" for developers who want to push the LattePanda 3 Delta beyond its standard limits and into the realm of professional AI vision.

## The Vision

The goal of **Panda Vision** is to transform a low-power SBC into a 25 TOPS AI powerhouse. This project bridges the gap between raw silicon and a personalized "smart home" experience through three core pillars:

* **Proactive Edge Intelligence:** Utilizing the DeepX DX-M1 for high-speed **Vehicle Identification** and **Facial Recognition** with total local privacy.
* **Hardware-Level Automation:** Using the LattePanda’s hardware to trigger real-time physical responses, such as "Welcome Home" greetings.
* **The Matrix Ecosystem:** Delivering high-visibility alerts on an Adafruit 16x32 RGB LED Matrix via a wireless network bridge.

-----

## The Adventure Roadmap

### **Phase 1: The Hardware War (Complete)**
* **The "SATA Sacrifice":** Disabling the internal SATA controller in BIOS to reallocate PCIe lanes for the AI module.
* **Vertical Storage Stack:** Implementing the Transcend 430s SSD with an ElecGear NV-2242A adapter and a MOGOOD 90° Up-Angle connector to maintain a slim profile.
* **Thermal Management:** Stabilizing the DX-M1 in a LattePanda 3 Delta Titan Case for 24/7 operation.

### **Phase 2: Vision & Inference (In Progress)**
* **Model Optimization:** Compiling YOLOv8/v9 models via the DXNN SDK for real-time detection.
* **Face ID Trigger:** Developing the Python logic to identify "Nathan" and initiate the greeting sequence.
* **Metrics:** Synchronizing hourly driveway stats to Cloudflare Workers.

### **Phase 3: Panda Vision (The Network Bridge)**
* **Distributed Display:** Using a Raspberry Pi Pico 2 WH in the kitchen to receive signals from the LattePanda over the home network.
* **The Matrix Driver:** Powering the Adafruit 16x32 LED Matrix with a dedicated 5V 4A Switching Power Supply and using a 74AHCT125 Level Shifter to bridge logic levels.
* **Messaging:** Implementing MQTT for near-zero latency communication between the "Brain" (LattePanda) and the "Eyes" (Pico/Matrix).

-----

## Stack Overview

| Layer | Component |
| :--- | :--- |
| **SBC** | LattePanda 3 Delta 864 (Intel N5105) |
| **Accelerator** | DEEPX DX-M1 (25 TOPS INT8) |
| **OS** | Ubuntu 24.04 LTS (on 64GB eMMC) |
| **Frontend** | Adafruit Industries Medium 16x32 RGB LED Matrix Panel |
| **Backend** | Cloudflare Workers & Metrics Dashboard |

-----

## Key Documentation

  * **[Setup & Hardware Validation Guide](docs/SETUP_GUIDE.md):** The definitive guide to BIOS, PCIe topology, and driver installation.
  * **Application Logic:** (Coming Soon) Detailed breakdown of the driveway counter Python scripts.
  * **The "Thermal Sandwich":** (Coming Soon) Mechanical blueprints for cooling the DX-M1.

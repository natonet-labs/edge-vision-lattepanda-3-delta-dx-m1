# DX-M1 AI Accelerator on LattePanda 3 Delta - Complete Setup Guide

**Hardware:** DEEPX DX-M1 M.2 AI Accelerator + LattePanda 3 Delta (Intel N5105)  
**OS:** Ubuntu 24.04 LTS  
**Status:** Fully validated and documented from scratch  

---

## Table of Contents

1. [Hardware Overview](#1-hardware-overview)
2. [The PCIe Topology Problem](#2-the-pcie-topology-problem)
3. [BIOS Configuration](#3-bios-configuration)
4. [OS Installation](#4-os-installation)
5. [Verifying PCIe Enumeration](#5-verifying-pcie-enumeration)
6. [Driver Installation](#6-driver-installation)
7. [DX-RT Runtime Installation](#7-dx-rt-runtime-installation)
8. [DX-COM Compiler Installation](#8-dx-com-compiler-installation)
9. [Permissions Setup](#9-permissions-setup)
10. [Persistent Module Loading](#10-persistent-module-loading)
11. [Verification](#11-verification)
12. [System Summary](#12-system-summary)
13. [Known Limitations](#13-known-limitations)
14. [Troubleshooting Reference](#14-troubleshooting-reference)

---

## 1. Hardware Overview

| Component | Detail |
|-----------|--------|
| SBC | LattePanda 3 Delta |
| CPU | Intel Celeron N5105 (Jasper Lake, 4-core, 10W TDP) |
| RAM | 8GB LPDDR4 |
| Storage (OS) | 64GB onboard eMMC |
| AI Accelerator | DEEPX DX-M1 M.2 2280 |
| NPU Performance | 25 TOPS (INT8) |
| NPU Memory | 4GB LPDDR5 |
| PCIe Interface | Gen3 x4 (runs at Gen2 x2 on this board) |
| BIOS Version | S70JR200-CN51G-8G-A |

---

## 2. The PCIe Topology Problem

This is the most critical section. **Do not skip it.**

The LattePanda 3 Delta has two M.2 slots:
- **M-Key (2280):** PCIe 3.0 x2, intended for NVMe SSDs
- **B-Key (2242/2252/2280):** SATA III by default, optionally PCIe x1 via BIOS flash

### Root Cause of Non-Detection

The Intel N5105 has only **8 PCIe lanes total**. On the LP3 Delta, the M-Key slot shares a PCIe Root Port (`00:1c.0`) with the onboard Intel I225-V 2.5G Ethernet controller. By default the BIOS allocates this Root Port exclusively to Ethernet, leaving no bus for the DX-M1.

Verified via `lspci -tv` - only the Ethernet appears on bus 01, DX-M1 is invisible:

```
+-1c.0-[01]----00.0  Intel Corporation Ethernet Controller I225-V
```

### The Fix

**Disable the SATA Controller in BIOS.** This frees the PCIe lane allocation and causes the BIOS to assign separate Root Ports to both the Ethernet and the M-Key slot:

```
+-1c.0-[01]----00.0  DEEPX Co., Ltd. DX_M1
+-1c.6-[02]----00.0  Intel Corporation Ethernet Controller I225-V
```

### Trade-off

Disabling the SATA controller makes the **M.2 B-Key slot unusable** for SATA SSDs. If you had a SATA SSD in the B-Key slot (e.g. Transcend 430S), it will no longer be recognized. The OS must run from the onboard eMMC or an NVMe SSD in the M-Key slot (which would conflict with the DX-M1).

**Recommended storage configuration:**
- OS → 64GB onboard eMMC
- AI Accelerator → M-Key slot (DX-M1)
- B-Key slot → unused (SATA disabled)

---

## 3. BIOS Configuration

Enter BIOS by pressing **DEL** repeatedly at POST.

### Required Settings

#### 3.1 Disable SATA Controller
```
Chipset → PCH-IO Configuration → SATA Configuration → SATA Controller → Disabled
```

#### 3.2 Set PCIe Speed to Gen2
```
Chipset → PCH-IO Configuration → PCI Express Configuration → Root Port 1 → PCIe Speed → Gen2
```

> **Critical:** Gen1 speed (2.5GT/s) is insufficient for the DX-M1 firmware initialization handshake. The DMA header buffer overflows and firmware times out. Gen2 (5GT/s) resolves this.

#### 3.3 Save and Reboot
Save changes and allow the system to boot fully into the OS before proceeding.

---

## 4. OS Installation

**Recommended OS:** Ubuntu 24.04 LTS

The DEEPX DXNN SDK and DX-RT runtime are tested and packaged specifically for Ubuntu 20.04, 22.04, and 24.04 LTS. Other distributions (including Linux Mint, which is Ubuntu-based) may work but are not officially supported and may cause library dependency issues.

Install Ubuntu 24.04 LTS to the onboard eMMC. Standard installation, no special partitioning required.

---

## 5. Verifying PCIe Enumeration

After BIOS configuration and OS boot, verify the DX-M1 is visible on the PCIe bus:

```bash
lspci -vn | grep 1ff4
```

Expected output:
```
01:00.0 1200: 1ff4:0000 (rev 01)
```

Verify the full bus tree:

```bash
lspci -tv
```

Expected (DX-M1 and Ethernet on separate buses):
```
+-1c.0-[01]----00.0  DEEPX Co., Ltd. DX_M1
+-1c.6-[02]----00.0  Intel Corporation Ethernet Controller I225-V
```

If the DX-M1 does not appear, recheck BIOS settings - specifically that SATA Controller is disabled and PCIe Speed is Gen2.

---

## 6. Driver Installation

### 6.1 Install Prerequisites

```bash
sudo apt update && sudo apt install git build-essential linux-headers-$(uname -r) -y
```

### 6.2 Clone the Driver Repository

```bash
git clone https://github.com/DEEPX-AI/dx_rt_npu_linux_driver.git
cd dx_rt_npu_linux_driver/modules
```

### 6.3 Build the Driver

```bash
make DEVICE=m1 PCIE=deepx 2>&1 | tail -10
```

The `.ko` files are built in the source tree but not automatically installed. Manually install them:

```bash
sudo mkdir -p /lib/modules/$(uname -r)/extra
sudo cp ~/dx_rt_npu_linux_driver/modules/pci_deepx/dx_dma.ko /lib/modules/$(uname -r)/extra/
sudo cp ~/dx_rt_npu_linux_driver/modules/rt/dxrt_driver.ko /lib/modules/$(uname -r)/extra/
sudo depmod -A
```

### 6.4 Install udev Rules

```bash
sudo cp ~/dx_rt_npu_linux_driver/modules/dx_dma.conf /etc/modprobe.d/
sudo bash -c 'echo "SUBSYSTEM==\"dxrt\", GROUP=\"deepx\", MODE=\"0660\"" > /etc/udev/rules.d/51-deepx-udev.rules'
sudo udevadm control --reload-rules
sudo udevadm trigger
```

### 6.5 Load the Driver

```bash
sudo modprobe dx_dma
sudo modprobe dxrt_driver
```

### 6.6 Verify Driver Binding

```bash
lsmod | grep -i dx
lspci -v -s 01:00.0 | grep -i driver
```

Expected:
```
dxrt_driver    53248  0
dx_dma        520192  1 dxrt_driver
Kernel driver in use: dx_dma_pcie
```

---

## 7. DX-RT Runtime Installation

DX-RT is a C++ build - not a Python package. Download `dx_rt_vX.X.X.tar.gz` from the DEEPX Developer Portal: https://developer.deepx.ai

### 7.1 Install Build Dependencies

```bash
sudo apt install cmake ninja-build pkg-config libncurses-dev libncursesw5-dev -y
```

### 7.2 Extract and Build

```bash
tar -xzf ~/Downloads/dx_rt_v*.tar.gz -C ~/dx_rt_npu_linux_driver/modules/
cd ~/dx_rt_npu_linux_driver/modules/dx_rt
sudo ./build.sh --use_ort_off --use_service_off --clean
```

> **Note:** The `--use_ort_off` flag disables the ONNX Runtime dependency which is not required for basic device operation and inference.

### 7.3 Install

The build script installs binaries to `/usr/local/bin` automatically on successful build. Verify:

```bash
which dxrt-cli
```

---

## 8. DX-COM Compiler Installation

DX-COM is the model compiler for converting ONNX/PyTorch/TensorFlow models to DXNN format.

Download `dx_com-X.X.X-*.whl` from the DEEPX Developer Portal, then install:

```bash
pip3 install --break-system-packages ~/Downloads/dx_com-*.whl
```

Add the local bin to PATH if not already set:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Verify:

```bash
dxcom --version
```

Expected:
```
DX-COM (DEEPX Compiler) X.X.X
Copyright (C) 2022-2025 DEEPX Inc.
Target Hardware: M1
```

---

## 9. Permissions Setup

Create a `deepx` group and add your user to it for non-root device access:

```bash
sudo groupadd deepx
sudo chown root:deepx /dev/dxrt0
sudo chmod 660 /dev/dxrt0
sudo usermod -aG deepx $USER
```

Activate group membership in current session:

```bash
newgrp deepx
```

Or log out and back in for permanent effect. Verify non-root access:

```bash
dxrt-cli --status
```

---

## 10. Persistent Module Loading

Ensure drivers load automatically on every boot:

```bash
echo "dx_dma" | sudo tee -a /etc/modules
echo "dxrt_driver" | sudo tee -a /etc/modules
```

Verify:

```bash
cat /etc/modules | grep dx
```

Expected:
```
dx_dma
dxrt_driver
```

---

## 11. Verification

Reboot the system and confirm everything comes up automatically:

```bash
sudo reboot
```

After reboot, run the full verification sequence:

```bash
# PCIe enumeration
lspci -vn | grep 1ff4

# Driver loaded
lsmod | grep dx

# Device fully operational
dxrt-cli --status
```

### Expected Final Output

```
DXRT v3.2.0
=======================================================
 * Device 0: M1, Accelerator type
---------------------   Version   ---------------------
 * RT Driver version   : v2.1.0
 * PCIe Driver version : v2.0.1
-------------------------------------------------------
 * FW version          : v2.1.5
--------------------- Device Info ---------------------
 * Memory : LPDDR5 5200 Mbps, 3.92GiB
 * Board  : M.2, Rev 1.5
 * Chip Offset : 0
 * PCIe   : Gen2 X2 [01:00:00]

NPU 0: voltage 750 mV, clock 1000 MHz, temperature 6X'C
NPU 1: voltage 750 mV, clock 1000 MHz, temperature 6X'C
NPU 2: voltage 750 mV, clock 1000 MHz, temperature 6X'C
=======================================================
```

---

## 12. System Summary

```
delta@lp3d
├── Hardware
│   ├── LattePanda 3 Delta (Intel N5105, 8GB RAM)
│   ├── 64GB eMMC  ←  Ubuntu 24.04 LTS (OS)
│   └── M-Key slot  ←  DEEPX DX-M1
│       ├── PCIe Gen2 x2
│       ├── 3.92GiB LPDDR5 @ 5200Mbps
│       └── 3x NPU cores @ 1000MHz, 25 TOPS
├── BIOS
│   ├── SATA Controller: Disabled
│   └── PCIe Root Port 1: Gen2
├── Drivers
│   ├── dx_dma.ko  (PCIe DMA driver)
│   └── dxrt_driver.ko  (RT interface driver)
└── Software
    ├── dxrt-cli  (device status and management)
    └── dxcom  (model compiler, ONNX → DXNN)
```

---

## 13. Known Limitations

| Limitation | Detail |
|------------|--------|
| No SATA B-Key | Disabling SATA controller to enable DX-M1 makes the B-Key slot unusable for SATA SSDs |
| PCIe x2 only | LP3 Delta M-Key is wired x2, not x4. DX-M1 is designed for x4 but operates at x2 |
| Gen2 required | Gen1 speed causes DMA header overflow during firmware init - must use Gen2 |
| eMMC only for OS | 64GB eMMC is the only practical storage when DX-M1 occupies M-Key slot |
| No wired Ethernet conflict | Ethernet moves to a separate Root Port when SATA is disabled - both work simultaneously |

---

## 14. Troubleshooting Reference

### DX-M1 not appearing in `lspci`
- Verify SATA Controller is **Disabled** in BIOS
- Verify PCIe Speed is **Gen2** on Root Port 1
- Check physical seating of M.2 module and retention screw

### `modprobe: FATAL: Module not found`
- The build script does not auto-install `.ko` files
- Manually copy from source tree: see [Section 6.3](#63-build-the-driver)

### `dxrt-cli: Device not found`
- Drivers not loaded - run `lsmod | grep dx`
- Device node missing - run `ls /dev/dxrt*`
- If device node exists but access denied - check group membership: `groups $USER`

### `Fail to initialize device 0` / PCIe Bus Errors in dmesg
- PCIe speed is Gen1 - change to Gen2 in BIOS
- Symptoms: `HeaderOF`, `NonFatalErr`, `polling_ack: timeout` in dmesg

### Drivers not loading after reboot
- Check `/etc/modules` contains `dx_dma` and `dxrt_driver`
- If missing, re-add: see [Section 10](#10-persistent-module-loading)

---

*Documented from a complete bring-up on LattePanda 3 Delta with BIOS S70JR200-CN51G-8G-A, Ubuntu 24.04 LTS, kernel 6.17, DX-RT v3.2.0, DX-COM v2.2.1.*

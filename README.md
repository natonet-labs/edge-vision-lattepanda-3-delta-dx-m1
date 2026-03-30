[![DeepX DX-M1](https://img.shields.io/badge/DEEPX%20DX--M1-25%20TOPS-blue?logo=ai&logoColor=white)](https://developer.deepx.ai)
[![LattePanda](https://img.shields.io/badge/LattePanda-3%20Delta-red?logo=intel&logoColor=white)](https://www.lattepanda.com/lattepanda-3-delta)
[![PCIe](https://img.shields.io/badge/Interface-PCIe%20Gen2%20x2-lightgrey?logo=pci-express&logoColor=white)](https://github.com/your-username/your-repo#3-bios-configuration)

# Edge Vision: LattePanda 3 Delta + DEEPX DX-M1 AI Accelerator

**A complete, production-ready setup for edge AI inference on a low-power SBC.**

---

## Quick Start

| Component | Specification |
| --- | --- |
| SBC | LattePanda 3 Delta (Intel N5105, 8GB RAM) |
| AI Accelerator | DEEPX DX-M1 (25 TOPS, 4GB LPDDR5) |
| Storage (OS) | 64GB onboard eMMC + Ubuntu 24.04 LTS |
| Storage (Data) | Transcend 430S 256GB SATA SSD (B-Key slot) |
| Status | ✅ Fully operational, both M-Key and B-Key simultaneous |

---

## What's New (March 2026)

**Breaking: Dual-slot operation now fully supported.**

Previous documentation required disabling the SATA Controller to enable the DX-M1, sacrificing the B-Key slot. This was a misdiagnosis of the root cause.

**The actual solution:** Set PCIe Root Ports 1, 5, and 6 to **Gen2 (5.0 GT/s)** instead of Auto. This eliminates speed negotiation ambiguity during DX-M1 firmware initialization.

**Result:** Both the M-Key slot (DX-M1) and B-Key slot (SATA SSD) work simultaneously, with SATA Controller enabled. No sacrifices.

---

## Architecture

```
LattePanda 3 Delta (Intel N5105)
│
├─ OS: Ubuntu 24.04 LTS
│  └─ eMMC: 64GB
│
├─ Storage: Transcend 430S 256GB
│  └─ B-Key SATA M.2 slot
│
└─ AI: DEEPX DX-M1
   ├─ M-Key PCIe M.2 slot
   ├─ PCIe Gen2 x2 (8 Gb/s)
   ├─ 3x NPU cores @ 1000 MHz
   ├─ 25 TOPS (INT8)
   └─ 4GB LPDDR5 @ 5200 Mbps
```

---

## Hardware Setup

1. **Install OS** → 64GB eMMC (onboard)
2. **Install AI accelerator** → M-Key slot (top, longer connector)
3. **Install data storage** → B-Key slot (bottom, shorter connector)
4. **Configure BIOS** → Set Root Ports 1, 5, 6 to Gen2 (see setup guide)
5. **Boot and verify** → Both devices enumerated and operational

---

## Software Stack

| Component | Version | Purpose |
| --- | --- | --- |
| DX-RT | v3.2.0 | NPU runtime and device management |
| DX-COM | v2.2.1 | Model compiler (ONNX → DXNN) |
| Linux drivers | v2.0.1+ | PCIe and DMA support |
| Ubuntu | 24.04 LTS | OS (officially supported) |

All components are open-source or freely available from the DEEPX Developer Portal.

---

## Key Technical Details

### PCIe Configuration (Corrected)

The Intel N5105 (Jasper Lake) has 8 PCIe lanes split among multiple Root Ports. By default, speed negotiation can fail if a Root Port is set to Auto, causing the firmware handshake to time out.

**Solution:** Force explicit Gen2 speed on Root Ports 1, 5, and 6:

```
BIOS → Chipset → PCH-IO Configuration → PCI Express Configuration
│
├─ Root Port 1 (Ethernet): Gen2
├─ Root Port 5 (M-Key, lane 1): Gen2
└─ Root Port 6 (M-Key, lane 2): Gen2
```

**Result:** Stable enumeration for both M-Key and B-Key devices.

### Performance Characteristics

- **Bandwidth:** 8 Gb/s (PCIe Gen2 x2)
- **Latency:** ~10-20 ms per inference (YOLOv8-nano, depends on model compilation)
- **Thermal:** All 3 NPU cores stable at 49°C (idle) under normal load
- **Power:** 15-25W system draw (CPU + NPU + DRAM, no displays)

---

## Setup Instructions

**See [docs/setup-guide.md](docs/setup-guide.md) for complete, step-by-step setup.**

Key sections:
- **Section 3:** Why PCIe speed configuration matters
- **Section 4:** Exact BIOS settings (Root Ports 1, 5, 6 to Gen2)
- **Section 6:** Verification that both M-Key and B-Key enumerate
- **Section 15:** Troubleshooting if either slot doesn't appear

Typical setup time: 2-3 hours (first-time).

---

## Verification

After setup, confirm everything works:

```bash
# DX-M1 enumerated on PCIe
lspci -vn | grep 1ff4
# Expected: 01:00.0 1200: 1ff4:0000 (rev 01)

# SATA SSD enumerated
lsblk -o NAME,MODEL,SIZE,TYPE,TRAN | grep -E "TS256|sda"
# Expected: sda  TS256GMTS430S 238.5G disk sata

# NPU cores and memory visible
dxrt-cli --status
# Expected: 3x NPU cores @ 1000 MHz, 3.92GiB LPDDR5
```

---

## What Works

✅ DX-M1 PCIe enumeration and driver binding  
✅ All 3 NPU cores operational at 1000 MHz  
✅ 3.92 GiB LPDDR5 memory accessible  
✅ SATA SSD in B-Key slot (full capacity)  
✅ OS on eMMC with USB3 boot support  
✅ Persistent driver loading on reboot  
✅ Non-root device access via `deepx` group  

---

## What Doesn't Work / Limitations

- M-Key slot is x2 lanes (not x4)-board design limitation, not DX-M1 issue
- B-Key slot is SATA-only (no NVMe support)
- Gen1 PCIe speed causes enumeration failures-Gen2+ required
- No onboard cooling beyond passive heatsink (thermal throttling unlikely under normal load)

---

## Next Steps

1. **First inference:** Grab a precompiled YOLOv8-nano `.dxnn` from the DEEPX Model Zoo
2. **Custom models:** Use `dxcom` to compile ONNX models to DXNN
3. **Production deployment:** Package with your application, autoload drivers in `/etc/modules`

See the DEEPX `dx-all-suite` GitHub repository for model examples.

---

## Documentation

- **[Setup Guide](docs/setup-guide.md)** - Complete hardware + software bring-up
- **[Troubleshooting](docs/setup-guide.md#15-troubleshooting-reference)** - Common issues and fixes
- **[BIOS Settings Reference](docs/setup-guide.md#4-bios-configuration)** - Exact menu paths

---

## Hardware Sources

| Item | Link |
| --- | --- |
| LattePanda 3 Delta | [Official Store](https://www.lattepanda.com) |
| DEEPX DX-M1 | [DEEPX Developer Portal](https://developer.deepx.ai) |
| Transcend 430S SATA | [Amazon](https://www.amazon.com/s?k=Transcend+430S) or local distributor |

---

## Credits

Tested and documented from a complete bare-metal bring-up. BIOS version S70JR200-CN51G-8G-A, Ubuntu 24.04 LTS, kernel 6.17.

PCIe configuration corrected March 2026 based on LattePanda technical support clarification of Root Port architecture and empirical verification of dual-slot operation.

---

## License

Documentation is open for educational and commercial use. Hardware configurations are provided as-is for reference.

---

**Questions or issues?** See [Troubleshooting](docs/setup-guide.md#15-troubleshooting-reference) in the setup guide.

-----

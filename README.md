# Battery-Identity-Governor
A Linux kernel-level scheduler architecture that dynamically adjusts DVFS/Thermal profiles based on physical battery identity.

![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)
![Kernel](https://img.shields.io/badge/Kernel-Linux%20%7C%20Android-green.svg)
![Status](https://img.shields.io/badge/Status-Research%20PoC-orange.svg)

> **A hardware-aware Linux kernel governor concept that maps physical battery characteristics to dynamic system thermal & DVFS policies.**

---
### ⚠️ Project Status: Conceptual Architecture & Research Prototype
**This repository hosts the architectural design, technical disclosure, and proof-of-concept (PoC) logic for the "Battery Identity Governor" system.** It does not currently contain a production-ready kernel module. Actual implementation requires platform-specific integration with PMIC drivers and SoC thermal HALs.
---

## 📖 Overview

**Battery-Identity-Governor** is a proposed kernel-level resource scheduling mechanism designed for mobile terminals. It aims to solve the latency and safety issues of traditional user-space performance toggles by introducing a direct **Hardware-to-Kernel** link.

When a specific battery is inserted, this governor allows the kernel to identify its physical capability through a hybrid detection logic and adjust the OS resource allocation strategies in real-time.

**Core Philosophy:** *The hardware defines the performance boundary, not the software user interface.*

> **Architectural Note:** This system functions as a **supplementary decision layer**, providing physical constraints to the existing Thermal HAL, rather than replacing the fundamental SoC thermal protection mechanisms.

## 🚀 Key Features (Design Goals)

### 1. Hybrid Verification Logic
* **Resistor ID (Fast Path):** Utilizes an ADC interface to read the ID pin voltage divider. Used for **rapid categorization** (e.g., Standard Type vs. High-Rate Type) rather than unique identification.
* **NFC/RFID Handshake (Secure Path):** Reads metadata from the battery's embedded tag via I2C/1-Wire. Designed to fetch detailed discharge parameters (C-Rate, Max Temp) and verify authenticity (depending on secure element implementation).

### 2. Interrupt-Driven Architecture
* Unlike legacy polling methods, this module proposes registering a hardware interrupt (IRQ) on the battery connector's detection pin.
* System response is triggered immediately upon physical insertion (`Plug_Event`), minimizing mode-switching latency.

### 3. Dynamic Resource Governance
Upon validation of a high-performance power source, the governor instructs the kernel to apply specific overrides (subject to platform support):
* **Thermal Policy Relaxation:** Requests higher current limits from the PMIC and adjusts software thermal throttling thresholds.
* **DVFS Capability Update:** Signals the cpufreq governor to unlock higher frequency tables (Turbo Modes) previously reserved/hidden.
* **Safety Fallback:** Enforces a restrictive "Safe Mode" if an unrecognized or unauthorized battery is detected.

## 🛠 System Architecture

The system bridges the gap between the **Physical Layer** (Battery Pack) and the **Kernel Space** (Scheduler).

### Hardware Topology
*(See `docs/hardware_schematic.png` for circuit details)*

> **Figure 1:** The hybrid detection circuit combining ADC voltage sampling and NFC coil interaction.

### Workflow Logic

1. **Event:** Battery Insertion -> IRQ Triggered.
2. **Driver Space:** `battery_id_driver` reads ADC value & NFC payload.
3. **Policy Space:** Match UID against `policy_map.json`.
4. **Kernel Space:** Write new values to `/sys/class/thermal/` and `/sys/devices/system/cpu/`.

> **Figure 2:** The decision-making flowchart from hardware interrupt to kernel execution.

## 📋 System Prerequisites & Assumptions
This architectural design assumes the following hardware/software environment:
* **Hardware:** Mobile terminal with ADC-readable battery ID pins and/or NFC controller accessible to the kernel.
* **Kernel:** Linux kernel with `sysfs` exposure for Thermal zones and CPU freq control.
* **Integration:** Requires vendor-specific modifications to the Bootloader/DTB (Device Tree Blob) to enable hardware interrupts for the battery connector.

## 📂 Repository Structure

```text
├── docs/
│ ├── hardware_schematic.png # Circuit Diagrams (Wired)
│ ├── hybrid_hardware_layout.png # Hybrid ID Hardware (NFC+ADC)
│ ├── flowchart.png # Logic Flow
│ ├── Technical_Disclosure_CN.pdf # Original Research Document (CN)
│ └── Technical_Disclosure_EN.pdf # Technical Disclosure (EN)
├── src/
│ ├── kernel_driver/ # (Prototype) Kernel module logic
│ ├── hal/ # Android HAL implementation concepts
│ └── scripts/ # Termux/Shell testing scripts
├── LICENSE # Apache License 2.0
└── README.md
```
## ⚠️ Disclaimer
Research Prototype Only.
This project represents the architectural implementation of the patent disclosure "Battery Identity Recognition & System Scheduling Method".
The logic described herein involves modifying kernel thermal parameters. Improper implementation on unsupported hardware may cause permanent battery damage or overheating. The author assumes no responsibility for hardware failure.
## 📜 License
This project is licensed under the Apache License 2.0 - see the LICENSE file for details.
Authored by: Yuan Jiayi (BG2NDR)

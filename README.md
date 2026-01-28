# Battery-Identity-Governor
A Linux kernel-level scheduler that dynamically adjusts DVFS/Thermal profiles based on physical battery identity.

![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)
![Kernel](https://img.shields.io/badge/Kernel-Linux%20%7C%20Android-green.svg)
![Status](https://img.shields.io/badge/Status-Research%20PoC-orange.svg)

> **A hardware-aware Linux kernel governor that maps physical battery characteristics (Resistor/NFC) to dynamic system thermal & DVFS policies.**

## 📖 Overview

**Battery-Identity-Governor** is a kernel-level resource scheduling mechanism designed for mobile terminals. It solves the latency and safety issues of traditional user-space performance toggles by introducing a direct **Hardware-to-Kernel** link.

When a specific battery (e.g., High-Rate Gaming Battery) is inserted, this governor automatically identifies its physical capability through a hybrid detection logic (ADC + NFC) and adjusts the OS resource allocation strategies in real-time.

**Core Philosophy:** *The hardware defines the performance boundary, not the software user interface.*

> **Architectural Note:** This system functions as a **supplementary decision layer**, providing physical constraints to the existing Thermal HAL, rather than replacing the fundamental SoC thermal protection mechanisms.

## 🚀 Key Features

### 1. Hybrid Identity Verification
* **Resistor ID (Fast Path):** Utilizes an ADC interface to read the ID pin voltage divider. Provides instant categorization (e.g., Standard vs. High-Performance) with zero latency.
* **NFC/RFID Handshake (Secure Path):** Reads encrypted metadata from the battery's embedded tag via I2C/1-Wire to verify authenticity and fetch detailed discharge parameters (C-Rate, Max Temp).

### 2. Interrupt-Driven Architecture
* Unlike legacy polling methods, this module registers a hardware interrupt (IRQ) on the battery connector's detection pin.
* System response is triggered immediately upon physical insertion (`Plug_Event`), ensuring seamless mode switching.

### 3. Dynamic Resource Governance
Once a high-performance battery is verified, the governor executes the following kernel overrides:
* **Thermal Wall Relaxation:** Modifies PMIC current limits and raises thermal throttling thresholds.
* **DVFS Unlocking:** Unlocks "Turbo" frequency tables for CPU/GPU (e.g., enabling overclocking frequencies previously hidden).
* **Safety Fallback:** Automatically enforces a restricted power mode if an unauthorized or unknown battery is detected.

## 🛠 System Architecture

The system bridges the gap between the **Physical Layer** (Battery Pack) and the **Kernel Space** (Scheduler).

### Hardware Topology
*(Please refer to `docs/hardware_schematic.png` for pinout details)*

> **Figure 1:** The hybrid detection circuit combining ADC voltage sampling and NFC coil interaction.

### Workflow Logic

1. **Event:** Battery Insertion -> IRQ Triggered.
2. **Driver Space:** `battery_id_driver` reads ADC value & NFC payload.
3. **Policy Space:** Match UID against `policy_map.json`.
4. **Kernel Space:** Write new values to `/sys/class/thermal/` and `/sys/devices/system/cpu/`.

> **Figure 2:** The decision-making flowchart from hardware interrupt to kernel execution.

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
⚠️ Disclaimer & Research Notice
Research Prototype Only.
This project represents the architectural implementation of the patent disclosure "Battery Identity Recognition & System Scheduling Method". It is intended as a Proof of Concept (PoC) for demonstrating the logic of hardware-driven kernel scheduling.
Improper use on unsupported hardware may cause permanent battery damage. The author assumes no responsibility for hardware failure.
📜 License
This project is licensed under the Apache License 2.0 - see the LICENSE file for details.
Authored by: Yuan Jiayi (BG2NDR)

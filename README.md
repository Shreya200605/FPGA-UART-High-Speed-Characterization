# FPGA-UART-High-Speed-Characterization
Development and performance characterization of a high-speed UART communication link between a Tang Nano 9K FPGA (Gowin GW1NR-9) transmitter and an LPC1768 ARM Cortex-M3 receiver. The project investigates baud-rate accuracy, throughput, data integrity, timing limitations and reliable transmission at increasing data rates.
# UART Transceiver with Physical-Layer Characterisation

**Theme:** Theme-2 – Serial Protocol Interface Systems  
**Institution:** Department of Electronics and Communication Engineering, KLE Technological University  
**Academic Year:** 2026–2027  
**Project Guide:** Prof. Renuka Barikar  
**Team:** Team-13  

---

## 1. Project Overview
This mini-project implements a configurable **8-N-1 UART Transceiver** in Verilog HDL on a **Gowin Tang Nano 9K (`GW1NR-9`)** FPGA, interfaced with an **NXP LPC1768 (ARM Cortex-M3)** microcontroller for hardware-in-the-loop (HIL) validation and physical-layer signal characterisation.

The objective is to evaluate asynchronous serial communication reliability across standard and high baud rates (`9,600 bps` to `921,600 bps`) by measuring clock drift tolerance, sampling alignment, framing error behaviour, bit error rate (BER), and physical waveform characteristics.

---

## 2. Current Project Status (Milestone: 32% — Phases I & II Design Complete)

| Stage / Gate | Title | Status |
| :--- | :--- | :--- |
| **S1** | Problem Definition & Requirements Capture | Completed |
| **S2** | Operating Environment & Interface Analysis | Completed |
| **G1** | Requirements Review & Verification Protocol | Completed |
| **S3** | System Architecture & Initial TX RTL Simulation | Completed (Architecture) / Simulation Started |
| **S4** | Constraints, Resources & Specification Limits | Completed |
| **S5 – S10** | Full TX+RX RTL Integration, Gowin Synthesis, LPC1768 Firmware & Physical Characterisation | Planned / In Progress |

---

## 3. System Specifications & Design Parameters

### Hardware & Toolchain
* **FPGA Target:** Sipeed Tang Nano 9K (`GW1NR-9`, `27 MHz` onboard crystal oscillator, `3.3V LVCMOS` I/O)
* **HIL Test Host:** NXP LPC1768 ARM Cortex-M3 Microcontroller
* **Physical Interconnect:** Point-to-point full-duplex cross-wired UART (`FPGA UART_TX` $\leftrightarrow$ `LPC1768 UART_RXD`, `FPGA UART_RX` $\leftrightarrow$ `LPC1768 UART_TXD`, common `GND`)
* **EDA & Verification Tools:** Gowin EDA, Verilog HDL, Icarus Verilog (`iverilog` / `vvp`), GTKWave, MSYS2 UCRT64

### Protocol & Timing Parameters (Baseline @ `19,200 bps`)
* **Frame Format:** `8-N-1` (1 Start bit `0`, 8 Data bits LSB-first `D0–D7`, No Parity, 1 Stop bit `1`, Idle `HIGH`)
* **Transmitter FSM:** 6-state synchronous state machine (`S0: IDLE` $\rightarrow$ `S1: LOAD` $\rightarrow$ `S2: START` $\rightarrow$ `S3: DATA` $\rightarrow$ `S4: STOP` $\rightarrow$ `S5: DONE`)
* **System Clock Frequency ($f_{\text{clk}}$):** `27 MHz` (`37.04 ns` period)
* **Baud-Rate Divisor Formula:** $\text{Divisor} = \frac{f_{\text{clk}}}{\text{Baud Rate}}$
* **Cycles per Bit (`19,200 bps`):** $\frac{27,000,000}{19,200} = 1,406.25\text{ cycles}$ (`1,406` integer cycles, `0.0178%` clock error)
* **Bit Duration:** `52.08 µs` | **10-Bit Frame Duration:** `520.83 µs`
* **Test Sweep Baud Rates:** `9,600`, `19,200`, `38,400`, `57,600`, `115,200`, `230,400`, `460,800`, and `921,600 bps` using known byte patterns (`0xA5`, `0x55`, `0xAA`)

---

## 4. Team-13 & Individual Contributions

| Member Name | USN | Primary Stage Ownership & Contributions |
| :--- | :--- | :--- |
| **Ranjita Nayak** | `01FE24BEC144` | **S1 (Problem Definition & Requirements Capture):** Scoped system problem statement, functional/non-functional requirements, physical-layer error metrics, and initial TX simulation setup. |
| **Shreya H** | `01FE24BEC070` | **S2 (Operating Environment & Interface Analysis):** Designed 8-N-1 UART framing protocol, 6-state TX FSM (`S0–S5`), $27\text{ MHz}$ baud-rate clock divider calculations, and 7 protocol verification scenarios. |
| **Reedhish Kabade** | `01FE24BEC152` | **S3 (System Architecture):** Developed system-level block diagrams, UART TX/RX RTL module partitioning, and datapath/FSM architectural diagrams. |
| **K Sai Varsha** | `01FE24BEC081` | **S4 (Constraints, Resources & Specification Limits):** Defined Tang Nano 9K (`3.3V LVCMOS`) & LPC1768 cross-wiring constraints, electrical/timing limits, and EDA/simulation toolchain setup. |

---

## 5. Repository Structure

```text
├── README.md                          # Project overview, specifications, team roles, and milestone status
├── docs/
│   ├── 01_requirements_and_specs.md   # S1 & S4: Problem statement, hardware constraints, and timing limits
│   ├── 02_protocol_and_fsm_design.md  # S2: 8-N-1 protocol analysis, 6-state TX FSM, and baud-rate math
│   ├── 03_system_architecture.md      # S3: System block diagrams and module descriptions
│   └── 04_test_plan_and_metrics.md    # G1: Verification test cases and 9-step baud-rate sweep protocol
├── rtl/                               # Verilog HDL source files (UART TX/RX modules)
├── tb/                                # Icarus Verilog testbenches
└── evidence/                          # Architecture diagrams (S3.1–S3.3) and GTKWave simulation captures

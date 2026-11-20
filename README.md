# UART Transceiver with Physical-Layer Characterisation

**Theme:** Theme-2 – Serial Protocol Interface Systems  
**Department:** Electronics and Communication Engineering, KLE Technological University  
**Academic Year:** 2026–2027  
**Project Guide:** Prof. Renuka Barikar  
**Team:** Team-13 (`01FE24BEC144`, `01FE24BEC070`, `01FE24BEC081`, `01FE24BEC152`)  

---

## 1. Project Overview
This project implements a configurable **8-N-1 Universal Asynchronous Receiver-Transmitter (UART) Transceiver** in Verilog HDL on a **Sipeed Tang Nano 9K (`GW1NR-9`)** FPGA, interfaced with an **NXP LPC1768 (ARM Cortex-M3)** microcontroller for hardware-in-the-loop (HIL) validation and physical-layer signal characterisation.

The goal is to evaluate asynchronous serial communication reliability across standard and high baud rates (`9,600 bps` to `921,600 bps`) by analysing clock drift tolerance, receiver sampling offset, framing/overrun error behaviour, bit error rate (BER), effective payload throughput, and physical waveform integrity.

---

## 2. Current Implementation Status (Stages S1–S4 & Gate G1 Completed)

| Stage / Gate | Focus Area | Status |
| :--- | :--- | :--- |
| **S1** | Problem Definition & Requirements Capture | Completed |
| **S2** | Operating Environment & Interface Analysis (8-N-1 Protocol, TX FSM, Clock Divider Math) | Completed |
| **G1** | Requirements Review & Baud-Rate Sweep Verification Protocol | Completed |
| **S3** | System Architecture, Module Partitioning & Initial TX RTL Simulation | Architecture Completed / TX Simulation Initiated |
| **S4** | Hardware/Electrical Constraints, Interconnect Mapping & Toolchain Limits | Completed |
| **S5 – S10** | Full TX+RX RTL Integration, Gowin EDA Synthesis, LPC1768 Firmware & Physical Measurements | Planned / In Progress |

---

## 3. Core Technical Specifications

### Hardware & EDA Toolchain
* **FPGA Platform:** Sipeed Tang Nano 9K (`GW1NR-9`, `27 MHz` onboard crystal oscillator, `3.3V LVCMOS` I/O bank)
* **HIL Controller:** NXP LPC1768 (ARM Cortex-M3)
* **Physical Interface:** Point-to-point full-duplex cross-wired UART (`FPGA UART_TX` $\leftrightarrow$ `LPC1768 UART_RXD`, `FPGA UART_RX` $\leftrightarrow$ `LPC1768 UART_TXD`, common `GND`)
* **HDL & Verification Tools:** Verilog HDL, Gowin EDA (synthesis & bitstream), Icarus Verilog (`iverilog` / `vvp`), GTKWave, MSYS2 UCRT64

### Protocol & Baseline Timing (`19,200 bps`)
* **Frame Structure:** `8-N-1` (1 Start bit `0`, 8 Data bits LSB-first `D0–D7`, No Parity, 1 Stop bit `1`, Idle line `HIGH`)
* **Transmitter FSM:** 6-state synchronous FSM (`S0: IDLE` $\rightarrow$ `S1: LOAD` $\rightarrow$ `S2: START` $\rightarrow$ `S3: DATA` $\rightarrow$ `S4: STOP` $\rightarrow$ `S5: DONE`)
* **System Clock ($f_{\text{clk}}$):** `27 MHz` ($T_{\text{clk}} = 37.04\text{ ns}$)
* **Clock Divider Equation:**
  $$\text{Divisor} = \frac{f_{\text{clk}}}{\text{Baud Rate}}$$
* **Baseline Cycles per Bit (`19,200 bps`):** $\frac{27,000,000}{19,200} = 1,406.25\text{ cycles}$ (`1,406` integer cycles, `0.0178%` baud-rate deviation)
* **Bit Period ($T_{\text{bit}}$):** `52.08 µs` | **10-Bit Frame Period ($T_{\text{frame}}$):** `520.83 µs`
* **Characterisation Baud Sweep:** `9,600`, `19,200`, `38,400`, `57,600`, `115,200`, `230,400`, `460,800`, and `921,600 bps` using test vectors `0xA5`, `0x55`, and `0xAA`

---

## 4. Repository Structure

* **`README.md`** — Project overview, specifications, and stage completion status
* **`docs/01_requirements_and_specs.md`** — S1 & S4: Problem statement, requirements, hardware & electrical constraints
* **`docs/02_protocol_and_fsm_design.md`** — S2: 8-N-1 framing, 6-state TX FSM, divisor calculations & test scenarios
* **`docs/03_system_architecture.md`** — S3: System block diagram, TX/RX RTL modules & datapath details
* **`docs/04_test_plan_and_metrics.md`** — G1: 9-step baud-rate sweep procedure & evaluation metrics
* **`rtl/`** — Verilog HDL source files (UART TX/RX)
* **`tb/`** — Icarus Verilog testbenches
* **`evidence/`** — Architecture diagrams (S3.1–S3.3) and GTKWave waveform captures

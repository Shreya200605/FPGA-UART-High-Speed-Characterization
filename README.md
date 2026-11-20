# FPGA-UART-High-Speed-Characterization
Development and performance characterization of a high-speed UART communication link between a Tang Nano 9K FPGA (Gowin GW1NR-9) transmitter and an LPC1768 ARM Cortex-M3 receiver. The project investigates baud-rate accuracy, throughput, data integrity, timing limitations and reliable transmission at increasing data rates.
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

```text
├── README.md                          # Project overview, specifications, and stage completion status
├── docs/
│   ├── 01_requirements_and_specs.md   # S1 & S4: Problem statement, requirements, hardware & electrical constraints
│   ├── 02_protocol_and_fsm_design.md  # S2: 8-N-1 framing, 6-state TX FSM, divisor calculations & test scenarios
│   ├── 03_system_architecture.md      # S3: System block diagram, TX/RX RTL modules & datapath details
│   └── 04_test_plan_and_metrics.md    # G1: 9-step baud-rate sweep procedure & evaluation metrics
├── rtl/                               # Verilog HDL source files (UART TX/RX)
├── tb/                                # Icarus Verilog testbenches
└── evidence/                          # Architecture diagrams (S3.1–S3.3) and GTKWave waveform captures
---

### File 2: `docs/01_requirements_and_specs.md`
**Purpose:** Consolidates your complete **S1 (Problem Definition & Requirements Capture)** and **S4 (Constraints, Resources & Specification Limits)** engineering documentation into one detailed reference file.

To add this on GitHub: click **Add file** $\rightarrow$ **Create new file**, type `docs/01_requirements_and_specs.md` as the filename, and paste the content below:

```markdown
# Stage S1 & S4: Requirements Capture, Hardware Constraints & Specification Limits

## 1. Problem Statement & Engineering Context (S1)
Asynchronous serial communication relies on agreed baud rates and frame structures without a shared clock line. Over physical interconnects—particularly as baud rates scale from standard rates (`9,600 bps`) up to high-speed links (`921,600 bps`)—link reliability is degraded by:
* **Clock Frequency Offset & Drift:** Cumulative phase error between the transmitter and receiver clock domains across a 10-bit frame.
* **Sampling Misalignment:** Deviation from the ideal mid-bit sampling instant causing bit-slip or false edge detection.
* **Signal Integrity & Physical-Layer Effects:** Finite rise/fall times, ringing, crosstalk, and external noise on interconnect wires.
* **Framing & Overrun Errors:** Corrupted start/stop bit transitions or unserviced receive buffers during continuous back-to-back transfers.

This project designs and implements a custom **8-N-1 UART Transceiver** in Verilog HDL on the **Tang Nano 9K (`GW1NR-9`)** FPGA and pairs it with an **LPC1768 ARM Cortex-M3** microcontroller to quantitatively characterise protocol behaviour and physical-layer signal quality across multiple baud rates.

---

## 2. Functional & Performance Requirements

### Functional Requirements
1. **Configurable Baud-Rate Generator:** Parameterised clock divider supporting standard and high-speed baud rates (`9,600` to `921,600 bps`) derived from the `27 MHz` FPGA system clock.
2. **8-N-1 Frame Formatting:** Automatic insertion of start bit (`0`), LSB-first serialisation of 8-bit payload (`D0–D7`), and stop bit (`1`), maintaining `HIGH` idle state.
3. **Synchronous TX & RX State Machines:** Deterministic finite state machines governing parallel-to-serial transmission and oversampled serial-to-parallel reception.
4. **Error Detection Flags:** Hardware detection of framing errors (invalid stop bit) and buffer overrun conditions.
5. **Physical-Layer Characterisation Capability:** Testable electrical interface supporting deterministic byte sequences (`0xA5`, `0x55`, `0xAA`) for BER, throughput, jitter, and waveform measurements.

---

## 3. Hardware & Electrical Interface Constraints (S4)

### Target Platforms & Pin Mapping
* **FPGA Board:** Sipeed Tang Nano 9K (`GW1NR-9` FPGA)
  * **Onboard Clock Source:** `27 MHz` crystal oscillator ($T_{\text{clk}} = 37.04\text{ ns}$)
  * **I/O Voltage Standard:** `3.3V LVCMOS`
* **Microcontroller Host:** NXP LPC1768 (ARM Cortex-M3)
  * **Logic Level:** `3.3V` TTL/CMOS compatible

### Physical Interconnect Wiring
Point-to-point full-duplex cross-coupled configuration with a shared ground reference:

| Tang Nano 9K (FPGA) | Connection | LPC1768 (MCU) | Signal Direction & Role |
| :--- | :---: | :--- | :--- |
| `UART_TX` | $\longrightarrow$ | `UART_RXD` | Serial data from FPGA transmitter to MCU receiver |
| `UART_RX` | $\longleftarrow$ | `UART_TXD` | Serial data from MCU transmitter to FPGA receiver |
| `GND` | $\longleftrightarrow$ | `GND` | Common electrical ground reference |

* **Electrical Constraint:** Both devices operate at `3.3V` logic levels; direct cross-connection is permitted without external level-shifting, provided common ground is established prior to power-up to prevent ground-offset damage.

---

## 4. Clock & Timing Specification Limits (`27 MHz` System Clock)

| Parameter | Symbol | Value / Formula | Notes |
| :--- | :---: | :--- | :--- |
| **System Clock Frequency** | $f_{\text{clk}}$ | `27 MHz` (`27,000,000 Hz`) | Tang Nano 9K onboard oscillator |
| **Clock Period** | $T_{\text{clk}}$ | `37.04 ns` | $1 / 27\text{ MHz}$ |
| **Baseline Baud Rate** | $B$ | `19,200 bps` | Primary reference rate for initial RTL verification |
| **Exact Clock Cycles per Bit** | $N_{\text{exact}}$ | `1,406.25 cycles` | $27,000,000 / 19,200$ |
| **Implemented Integer Divisor** | $N_{\text{int}}$ | `1,406 cycles` | `0.0178%` timing error (well within $\pm 2\%$ UART tolerance) |
| **Single Bit Duration** | $T_{\text{bit}}$ | `52.08 µs` | $1 / 19,200\text{ s}$ |
| **10-Bit Frame Duration (8-N-1)** | $T_{\text{frame}}$ | `520.83 µs` | $10 \times T_{\text{bit}}$ (1 Start + 8 Data + 1 Stop) |

---

## 5. Software & Verification Toolchain
* **HDL Language:** Verilog HDL (`IEEE 1364`)
* **FPGA Synthesis & Place-and-Route:** Gowin EDA
* **Functional RTL Simulation:** Icarus Verilog (`iverilog` compiler & `vvp` runtime)
* **Waveform Inspection:** GTKWave (`.vcd` dump analysis)
* **Build Environment:** MSYS2 UCRT64

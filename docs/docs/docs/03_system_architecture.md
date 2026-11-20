# Stage S3: System Architecture

**Phase:** Phase II — Architecture & Feasibility  
**Scope:** System architecture conceptualization, functional decomposition into subsystems, interface allocation, and justification against architectural alternatives.

---

## S3.1 System Architecture & Conceptualization
* **System Concept:** A full-duplex asynchronous serial communication and physical-layer characterisation testbed pairing a **Sipeed Tang Nano 9K (`GW1NR-9`)** FPGA (`27 MHz` clock) with an **NXP LPC1768 (ARM Cortex-M3)** microcontroller.
* **Primary vs. Bidirectional Architecture:**
  * **Primary Characterisation Path (`FPGA UART_TX` $\rightarrow$ `LPC1768 UART_RXD`):** Configurable high-rate UART transmitter generating deterministic `8-N-1` frames (`0xA5`, `0x55`, `0xAA`) for reliability, BER, and throughput characterisation.
  * **Bidirectional Verification Loop (`LPC1768 UART_TXD` $\rightarrow$ `FPGA UART_RX`):** Return path enabling the FPGA to receive echoed frames, compare against expected patterns, and log framing/mismatch errors.
* **Visual Artifact Reference:** `evidence/architecture/S3_1_system_architecture_conceptualization.png` *(exported from portfolio artifact `Screenshot 2026 11 19 221232`)*

---

## S3.2 Functional Decomposition & Subsystem Architecture
The FPGA design is decomposed into modular, synchronous Verilog HDL subsystems clocked by the `27 MHz` onboard oscillator:

| Subsystem / Module | Functional Responsibility | Inputs / Outputs | Current Status |
| :--- | :--- | :--- | :--- |
| **1. Clock & Baud-Rate Generator** | Divides `27 MHz` system clock (`37.04 ns`) into bit-rate timing ticks (`1,406` cycles/bit at `19,200 bps`; scalable divisors up to `921,600 bps`). | `clk`, `reset`, `baud_sel` $\rightarrow$ `baud_tick` | Implemented at `19,200 bps`; multi-baud scaling planned |
| **2. UART Transmitter (`uart_tx`)** | Latches parallel byte, frames `8-N-1` sequence (`Start 0`, `D0–D7` LSB-first, `Stop 1`), and drives `uart_tx` with `tx_busy`/`tx_done` handshake. | `tx_data[7:0]`, `tx_start`, `baud_tick` $\rightarrow$ `uart_tx`, `tx_busy`, `tx_done` | **Implemented, simulated (`0xA5`), and synthesized in Gowin EDA** |
| **3. UART Receiver (`uart_rx`)** | Detects falling edge on `uart_rx`, samples start/data/stop bits at bit centers, reconstructs parallel byte, and checks stop-bit framing. | `uart_rx`, `baud_tick` $\rightarrow$ `rx_data[7:0]`, `rx_valid`, `frame_err` | Planned (Phase II–III) |
| **4. Verification & Error Counter** | Compares received payload against expected reference pattern and accumulates valid frame counts and bit/frame error counts. | `rx_data[7:0]`, `rx_valid`, `frame_err` $\rightarrow$ `err_count`, `status_flags` | Planned (Phase II–III) |
| **5. Status & Diagnostic Interface** | Drives on-board Tang Nano 9K LEDs for live heartbeat/activity indication and error signaling. | `tx_busy`, `status_flags` $\rightarrow$ `led[5:0]` | LED heartbeat verified on Tang Nano 9K hardware |

* **Visual Artifact Reference:** `evidence/architecture/S3_2_functional_decomposition.png` *(exported from portfolio artifact `Screenshot 2026 11 19 223610`)*

---

## S3.3 Interface Allocation & Architecture Alternatives
* **Hardware Interface Allocation:**
  * **FPGA (`Tang Nano 9K`, `3.3V LVCMOS`):** `clk` (`27 MHz`), `reset`, `uart_tx` (output), `uart_rx` (input), `led` status outputs, shared `GND`.
  * **Microcontroller (`NXP LPC1768`, `3.3V`):** `UART_RXD` (receives from FPGA `uart_tx`), `UART_TXD` (drives FPGA `uart_rx`), shared `GND`.
* **Architectural Choice Justification:**
  * Implementing the UART transceiver in **custom Verilog HDL on an FPGA** (rather than relying solely on a fixed-function MCU peripheral or soft-core processor) provides deterministic cycle-accurate control (`37.04 ns` resolution) over baud-rate divisors, mid-bit sampling offsets, fault injection, and hardware error counting across rates up to `921,600 bps`.
* **Visual Artifact Reference:** `evidence/architecture/S3_3_interface_allocation_alternatives.png` *(exported from portfolio artifact `Screenshot 2026 11 19 224405`)*

---

## Supporting Evidence & Early Verification (S3)
* **UART Transmitter Simulation Waveform:** Simulated in **Icarus Verilog** and viewed in **GTKWave** at `19,200 bps` with test vector `0xA5`, confirming start bit (`0`), LSB-first data (`1-0-1-0-0-1-0-1`), and stop bit (`1`).
* **Hardware Synthesis:** Synthesized in **Gowin EDA** with on-board LED blink verification on the Tang Nano 9K.

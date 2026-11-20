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

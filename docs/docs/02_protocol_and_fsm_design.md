# Stage S2: Operating Environment, Protocol Framing & TX FSM Design

## 1. Asynchronous Serial Interface & 8-N-1 Frame Structure
The transceiver implements a standard **8-N-1** asynchronous serial frame without an external clock line. Both transmitter and receiver operate on pre-configured baud rates derived from the `27 MHz` FPGA system clock.

### Frame Bit Sequence (10 Bits Total)
* **Idle State:** Logic `HIGH` (`1`) when no transmission is active.
* **Start Bit (1 bit):** Logic `LOW` (`0`) transition signaling the beginning of a frame and triggering receiver synchronization.
* **Data Payload (8 bits):** Transmitted **LSB-first** (`D0` $\rightarrow$ `D1` $\rightarrow$ `D2` $\rightarrow$ `D3` $\rightarrow$ `D4` $\rightarrow$ `D5` $\rightarrow$ `D6` $\rightarrow$ `D7`).
* **Parity Bit (0 bits):** None (`N`).
* **Stop Bit (1 bit):** Logic `HIGH` (`1`) returning the line to idle state and verifying frame alignment.

### Example Frame Timing (`0xA5` = `8'b1010_0101` @ `19,200 bps`)
Transmitted LSB-first on `UART_TX`:

| Bit Position | Idle | Start | `D0` | `D1` | `D2` | `D3` | `D4` | `D5` | `D6` | `D7` | Stop | Idle |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Logic Level** | `1` | `0` | `1` | `0` | `1` | `0` | `0` | `1` | `0` | `1` | `1` | `1` |
| **Duration ($\mu\text{s}$)** | — | `52.08` | `52.08` | `52.08` | `52.08` | `52.08` | `52.08` | `52.08` | `52.08` | `52.08` | `52.08` | — |

---

## 2. 6-State UART Transmitter Finite State Machine (FSM)
The transmitter is governed by a deterministic 6-state synchronous FSM clocked at $f_{\text{clk}} = 27\text{ MHz}$ and paced by the baud-rate tick generator:

```text
  +-----------+   tx_start == 1    +-----------+   next_clk    +------------+
  | S0: IDLE  | -----------------> | S1: LOAD  | ------------> | S2: START  |
  | (TX = 1)  |                    | (Latch D) |               | (TX = 0)   |
  +-----------+                    +-----------+               +------------+
        ^                                                            |
        |                                                     baud_tick (1 bit)
        |                                                            v
  +-----------+   baud_tick        +-----------+   bit_idx==7  +------------+
  | S5: DONE  | <----------------- | S4: STOP  | <------------ | S3: DATA   |
  | (tx_done) |                    | (TX = 1)  |   & baud_tick | (TX=D[i])  |
  +-----------+                    +-----------+               +------------+

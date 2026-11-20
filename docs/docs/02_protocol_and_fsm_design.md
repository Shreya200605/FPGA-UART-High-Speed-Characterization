# Stage S2: Operating Environment & Interface Analysis

---

## S2.1 Operating States & Signal Behaviour

### 1. System Overview & Communication Direction
The system is a point-to-point asynchronous serial link between a **Sipeed Tang Nano 9K FPGA** (`27 MHz` clock) and an **NXP LPC1768 microcontroller**:
* **Primary Data Direction (`FPGA TX` $\rightarrow$ `LPC1768 RX`):** The official objective—a high-rate UART transmitter characterised for reliability and throughput.
* **Bidirectional Extension (`LPC1768 TX` $\rightarrow$ `FPGA RX`):** Added on the mentor's instruction to make the link bidirectional.
* **Frame Format:** `8-N-1` (1 start bit, 8 data bits LSB first, no parity, 1 stop bit).

### 2. System-Level Operating States (`S0`–`S5`)

| State | Description | Entry Condition | Exit Condition |
| :--- | :--- | :--- | :--- |
| **`S0: Reset / Initialize`** | FPGA logic and counters cleared, lines driven to idle | Power-up or reset | Reset released |
| **`S1: Idle`** | Both TX and RX lines held high, waiting for activity | After reset or after a completed frame | Transmit request, or a falling edge on RX |
| **`S2: Transmit (FPGA to LPC1768)`** | FPGA serializes a byte, or a stream of back-to-back bytes | Transmit request | Last byte's stop bit finished |
| **`S3: Receive (LPC1768 to FPGA)`** | FPGA detects a start bit and samples the incoming frame | Falling edge on RX | Stop bit sampled |
| **`S4: Verify & Count`** | Received byte compared with the expected value, counters updated | Frame received | Comparison done |
| **`S5: Error`** | Framing error or data mismatch flagged and counted | Invalid stop bit or mismatch | Count recorded, return to Idle |

* **State Transitions:** `S0` $\rightarrow$ `S1` on reset release. `S1` $\rightarrow$ `S2` on a transmit request. `S1` $\rightarrow$ `S3` on an RX falling edge. `S2` $\rightarrow$ `S1` when transmission ends. `S3` $\rightarrow$ `S4` on a valid frame, or `S3` $\rightarrow$ `S5` on a framing error. `S4` $\rightarrow$ `S1` on a match, or `S4` $\rightarrow$ `S5` on a mismatch. `S5` $\rightarrow$ `S1` after logging.
* **Current Status (S2.1):** The Verilog transmitter has been developed and simulated at `19,200 bps` with test byte `0xA5`, synthesized in Gowin EDA, and its operation indicated on the FPGA board’s on-board LEDs. The `uart_tx` output has not yet been connected to the LPC1768 or measured on an external instrument. Receive, verify, and error-counting behaviour on hardware, all LPC1768 interaction, bidirectional operation, and high-speed operation are planned and not yet tested.

### 3. Frame-Level States (Inside One UART Byte)

| Phase | Bits | Line Level | Duration |
| :--- | :---: | :--- | :--- |
| **Idle** | – | High (logic `1`) | Indefinite |
| **Start bit** | `1` | Low (logic `0`) | `1` bit time |
| **Data bits** | `8` (`D0` first to `D7` last) | Follows data | `8` bit times |
| **Stop bit** | `1` | High (logic `1`) | `1` bit time |

One frame is **10 bit times**. For the test byte **`0xA5`** (binary `1010 0101`), the line carries, in order: `Start 0`, then `D0–D7 = 1, 0, 1, 0, 0, 1, 0, 1`, then `Stop 1`.

### 4. Signal Behaviour

| Signal | Type | Direction | Range | Frequency / Timing |
| :--- | :--- | :--- | :--- | :--- |
| **`clk`** | Digital clock | On-board oscillator to FPGA | `3.3 V` logic (to be confirmed from board documentation) | `27 MHz`, period `37.04 ns` |
| **`reset`** | Digital level | Button/logic to FPGA | Logic high/low | Asynchronous, event-driven |
| **`uart_tx` (FPGA)** | Digital serial data | FPGA to LPC1768 RX | `0 V / 3.3 V` (to be confirmed) | Bit rate = baud rate; idle high |
| **`uart_rx` (FPGA)** | Digital serial data | LPC1768 TX to FPGA | `0 V / 3.3 V` (to be confirmed) | Asynchronous; start bit marks timing |
| **`GND`** | Reference | Common | `0 V` | Must be shared by both boards |
| **`Status/error flags`** | Internal digital | Within FPGA | Logic level | Updated once per frame |

### 5. Timing at the Implemented Rate (`19,200 bps`)
* **Bit time:** $\frac{1}{19,200} \approx 52.08\ \mu\text{s}$
* **Frame time:** $10 \times 52.08\ \mu\text{s} \approx 520.8\ \mu\text{s}$
* **Clock cycles per bit:** $\frac{27,000,000}{19,200} = 1,406.25$, so the integer divider uses `1,406` cycles (confirm exact value in Verilog parameter).

### 6. Timing Expectation at Higher Rates (Planned, Not Yet Measured)
The signal on the line is a digital waveform whose fastest transitions occur when alternating bits are sent (such as `0xAA` or `0x55`), at a frequency of $\text{baud} \div 2$. As the baud rate rises, bit time shrinks and the link becomes more sensitive to clock mismatch, edge quality, and wiring. The highest reliable rate is not assumed here; it will be determined experimentally in the verification phase.

---

## S2.2 Operating Environment Assessment

### 1. Intended Operating Environment
The system is developed and characterised as a laboratory bench setup. The Tang Nano 9K FPGA board and the LPC1768 development board are placed on a desk, connected by short jumper wires for `TX`, `RX`, and a common ground, and each board is powered through its USB connection (power source to be confirmed). A PC runs the tools (`Gowin EDA`, `Icarus Verilog`, `GTKWave`, `MSYS2 UCRT64`) and is used to observe results.

| Factor | Expected Condition | Status |
| :--- | :--- | :--- |
| **Location** | Indoor laboratory bench | To be confirmed |
| **Ambient temperature** | Normal room temperature; no thermal stress planned | To be confirmed |
| **Power** | USB-powered boards; supply noise not yet characterised | To be confirmed |
| **Interconnect** | Short jumper wires, point-to-point, shared ground | To be confirmed (wire length to be recorded) |
| **Mounting** | Loose boards on a desk, or breadboard | To be confirmed |
| **Mechanical stress, vibration, humidity** | Not applicable to a bench prototype | Assumption |

### 2. Environmental Factors Affecting High-Rate UART Link Reliability

| Factor | Why It Matters | How It Will Be Handled |
| :--- | :--- | :--- |
| **Wire length and quality** | Longer or loose wires add capacitance and pick up noise, distorting edges at high rates | Keep wires short; record the length used |
| **Common ground** | Without a shared ground the logic levels have no common reference | Connect `GND` between the boards before connecting data lines |
| **Logic-level compatibility** | A mismatch can cause errors or damage | Verify the voltage levels of both boards from their documentation before wiring |
| **Electrical noise and crosstalk** | Nearby wires, power lines, and switching can corrupt bits | Keep `TX` and `RX` wires apart and short; compare error counts when layout changes |
| **Clock accuracy of each board** | A baud-rate mismatch causes sampling errors, worse at higher rates | Analysed in S6 and S7; measured in verification |
| **Power-supply stability** | Supply noise can disturb logic timing | Observe during testing; not characterised yet |
| **Measurement access** | A blink-only LED cannot show a waveform | Use an oscilloscope or logic analyser if available (to be confirmed) |

### 3. Status (S2.2)
* **Completed:** The design was developed and simulated, and the on-board LED on the FPGA blinks while the design runs.
* **Proposed:** The bench setup and the wiring described above.
* **Planned:** Recording the real setup (wire length, power source, measurement tools) when the boards are first connected. No environmental testing has been performed.

---

## S2.3 Interface Characteristics & Operating Scenarios

### 1. System Interfaces

| Interface | Between | Direction | Type | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **`Clock`** | On-board oscillator to FPGA | Input | `27 MHz` digital clock | Time base for all FPGA timing |
| **`Reset`** | Button or logic to FPGA | Input | Digital level | Initialises the design |
| **`uart_tx`** | FPGA to LPC1768 RXD | FPGA to LPC1768 | Asynchronous serial, digital | Primary path: the official objective |
| **`uart_rx`** | LPC1768 TXD to FPGA | LPC1768 to FPGA | Asynchronous serial, digital | Return path: the mentor's bidirectional requirement |
| **`GND`** | Between both boards | Common | Reference | Must be connected |
| **`LEDs`** | FPGA outputs | Output | Digital | Currently a blink indication; error flags proposed |
| **`PC link`** | PC to boards | Programming and monitoring | USB | Programming via Gowin EDA; use for monitoring to be confirmed |

### 2. UART Interface Characteristics

| Parameter | Value | Status |
| :--- | :--- | :--- |
| **Format** | 8 data bits, no parity, 1 stop bit (`8-N-1`), LSB first | Defined; implemented in the transmitter |
| **Idle level** | High (logic `1`) | Defined |
| **Baud rate** | `19,200 bps` in the current design; configurable higher rates proposed | `19,200` simulated; higher rates planned |
| **Timing** | Asynchronous, each frame starts with a falling edge; receiver samples relative to start bit | Defined |
| **Flow control** | None assumed (to be confirmed) | Proposed |
| **Logic level** | `3.3 V` expected, to be verified on both boards | To be verified |
| **Pins** | FPGA pins and LPC1768 UART pins | To be assigned and verified from documentation |
| **LPC1768 UART configuration** | Which UART peripheral, its settings, and baud-rate limits | To be verified from board documentation and LPC1768 user manual |

### 3. Operating Scenarios (Scenarios 1–7)
Scenarios 1 to 3 build sequentially on each other; Scenarios 4 to 7 address the bidirectional requirement and high-rate performance goals:

| # | Scenario | Purpose | Status |
| :---: | :--- | :--- | :--- |
| **1** | **Power-up and reset** | The design starts in a known idle state, with TX held high | Simulated; LED blink observed on the board |
| **2** | **Single byte at `19,200 bps` (`0xA5`)** | Confirm the frame format and timing | Simulated; hardware waveform not yet measured |
| **3** |

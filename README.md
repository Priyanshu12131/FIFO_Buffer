# 64-Stage FIFO Memory with Overflow/Underflow Protection

A **64-stage, 8-bit synchronous FIFO (First-In-First-Out) memory buffer** designed and verified using **SystemVerilog RTL**. The design implements circular-buffer based data storage with independent read/write pointers and integrated protection against **overflow and underflow** conditions.

---

## 📌 Project Overview

A FIFO (First-In-First-Out) is a digital storage structure where the data written first is read first. FIFOs are commonly used for **data buffering, data-rate matching, protocol buffering, and communication interfaces**.

This project implements a **64-stage synchronous FIFO** with:

* 64 storage locations
* 8-bit data width
* Read and write pointer management
* Full and Empty status flags
* Overflow and Underflow protection
* Programmable threshold indication
* FIFO occupancy counter
* Asynchronous active-low reset
* Synthesizable SystemVerilog RTL
* Simulation-based verification

The design is parameterizable and suitable for **FPGA and ASIC RTL design flows**.

---

## ✨ Features

* **64-depth FIFO**
* **8-bit data width**
* Synchronous read/write operation
* Circular buffer architecture
* Independent read and write pointers
* Full detection
* Empty detection
* Overflow detection
* Underflow detection
* Threshold status indication
* FIFO occupancy counter
* Active-low asynchronous reset
* Synthesizable RTL
* Parameterizable architecture
* Simulation testbench for functional verification

---

## 🏗️ FIFO Architecture

The FIFO uses a **circular buffer architecture** consisting of:

```text
             +----------------------+
             |      FIFO Memory     |
             |      64 × 8-bit      |
             +----------+-----------+
                        |
          +-------------+-------------+
          |                           |
     Write Pointer               Read Pointer
          |                           |
          ↓                           ↓
       Write Data                  Read Data
```

The **Write Pointer (wr_ptr)** identifies the next memory location where data will be written.

The **Read Pointer (rd_ptr)** identifies the oldest valid data location that should be read.

Both pointers wrap around when they reach the end of the 64-entry memory, creating a circular buffer.

---

## ⚙️ Working Principle

### Write Operation

When:

```text
wr_en = 1
FIFO is not full
```

the input data is written into the memory location pointed to by `wr_ptr`.

After a successful write:

```text
wr_ptr = wr_ptr + 1
```

The FIFO occupancy counter is also incremented.

---

### Read Operation

When:

```text
rd_en = 1
FIFO is not empty
```

the data at the location pointed to by `rd_ptr` is transferred to the output.

After a successful read:

```text
rd_ptr = rd_ptr + 1
```

The FIFO occupancy counter is decremented.

---

### Simultaneous Read and Write

The FIFO can perform read and write operations in the same clock cycle when both operations are valid.

In this condition, the number of stored elements remains unchanged.

---

## 🔄 FIFO Pointer Management

The design uses two 6-bit pointers:

```text
wr_ptr → Write Pointer
rd_ptr → Read Pointer
```

Since:

```text
Depth = 64
log2(64) = 6
```

the pointers require 6 bits.

When a pointer reaches the maximum address, it automatically wraps back to zero.

Example:

```text
60 → 61 → 62 → 63 → 0 → 1 → 2
```

This provides the circular-buffer behavior required for FIFO operation.

---

## 🚦 Status Flags

The FIFO provides five important status indications.

| Flag        | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| `full`      | Indicates that all 64 FIFO locations are occupied            |
| `empty`     | Indicates that no valid data is available                    |
| `overflow`  | Indicates an attempted write while FIFO is full              |
| `underflow` | Indicates an attempted read while FIFO is empty              |
| `threshold` | Indicates that FIFO occupancy is below the defined threshold |

The threshold value used in the implementation is:

```text
THRESHOLD = 16
```

Therefore:

```text
threshold = 1
```

when the FIFO contains fewer than 16 entries.

---

## 🛡️ Overflow Protection

Overflow occurs when the system attempts to write data while the FIFO is already full.

Condition:

```text
overflow = full && wr_en
```

When the FIFO reaches 64 stored elements, additional writes are prevented and the overflow indication becomes active.

---

## 🛡️ Underflow Protection

Underflow occurs when the system attempts to read data while the FIFO is empty.

Condition:

```text
underflow = empty && rd_en
```

This prevents invalid data from being read when no valid data is available.

---

## 📋 Design Specifications

| Parameter     | Value                                       |
| ------------- | ------------------------------------------- |
| FIFO Type     | Synchronous FIFO                            |
| FIFO Depth    | 64 stages                                   |
| Data Width    | 8 bits                                      |
| Address Width | 6 bits                                      |
| Clock         | Single synchronous clock                    |
| Reset         | Active-low asynchronous                     |
| HDL           | SystemVerilog                               |
| Write Enable  | `wr_en`                                     |
| Read Enable   | `rd_en`                                     |
| Status        | Full, Empty, Overflow, Underflow, Threshold |

---

## 🔌 Module Interface

### Inputs

| Signal   | Width | Description        |
| -------- | ----: | ------------------ |
| `clk`    |     1 | System clock       |
| `rst`    |     1 | Asynchronous reset |
| `wr_en`  |     1 | Write enable       |
| `rd_en`  |     1 | Read enable        |
| `buf_in` |     8 | Input data         |

### Outputs

| Signal         | Width | Description            |
| -------------- | ----: | ---------------------- |
| `buf_out`      |     8 | Output data            |
| `buf_empty`    |     1 | FIFO empty indication  |
| `buf_full`     |     1 | FIFO full indication   |
| `overflow`     |     1 | Overflow indication    |
| `underflow`    |     1 | Underflow indication   |
| `threshold`    |     1 | Threshold indication   |
| `fifo_counter` |     7 | Current FIFO occupancy |

---

## 🧩 RTL Design Structure

The RTL implementation contains the following major logic blocks:

### 1. FIFO Memory

```systemverilog
logic [7:0] buf_mem [63:0];
```

This creates a memory containing **64 locations**, with each location storing **8-bit data**.

### 2. Read/Write Pointers

```systemverilog
logic [5:0] wr_ptr, rd_ptr;
```

These pointers control the memory locations used for write and read operations.

### 3. FIFO Counter

```systemverilog
logic [6:0] fifo_counter;
```

The counter tracks the number of elements currently stored in the FIFO.

### 4. Status Logic

The counter is used to generate:

```text
EMPTY → counter == 0
FULL  → counter == 64
```

### 5. Protection Logic

Overflow and underflow are generated from invalid write/read attempts.

---

## 🧪 Verification / Testbench

A SystemVerilog testbench was developed to verify the FIFO behavior.

The testbench verifies:

* Reset operation
* Write operation
* Read operation
* FIFO ordering
* FIFO occupancy
* Empty condition
* Full condition
* Underflow condition
* Overflow condition
* Boundary behavior

The testbench generates a clock with:

```systemverilog
always #5 clk = ~clk;
```

and performs multiple write/read operations using randomized 8-bit data.

---

## 🔬 Test Scenarios

### Test 1 — Write Operation

Five random data values are written into the FIFO.

Example:

```text
36
129
9
99
13
```

The FIFO counter increases:

```text
1 → 2 → 3 → 4 → 5
```

---

### Test 2 — Read Operation

Three values are read from the FIFO.

The data is retrieved in the same order in which it was written:

```text
36 → 129 → 9
```

This demonstrates the **First-In-First-Out** property.

---

### Test 3 — Underflow

After reading all available data, additional read operations are performed.

When the FIFO is empty:

```text
empty = 1
underflow = 1
```

This confirms that invalid reads are detected.

---

### Test 4 — Overflow

The testbench attempts to write more than 64 entries.

The FIFO reaches:

```text
COUNT = 64
```

Further write attempts produce:

```text
overflow = 1
```

while the FIFO count remains at 64.

---

## 📊 Simulation Result

The simulation successfully demonstrated:

* Correct sequential write behavior
* Correct sequential read behavior
* FIFO ordering
* Pointer progression
* FIFO occupancy tracking
* Underflow detection
* Overflow detection
* Full and empty boundary behavior
* Pointer wraparound

The report records the final testbench output as:

```text
===== FIFO TEST COMPLETED =====
TOTAL PASS = 77
TOTAL FAIL = 2
```

> **Note:** The report's conclusion states that all test cases passed, but the console output shown in the report records `TOTAL FAIL = 2`. For a GitHub README, it is better to report the actual console result rather than claim 100% passing unless those two failures are separately resolved.

---

## 💻 Technologies Used

* **SystemVerilog**
* **RTL Design**
* **Digital Logic Design**
* **FIFO Architecture**
* **Simulation & Verification**
* **FPGA/ASIC-oriented RTL Design**

---

## 📁 Suggested Repository Structure

```text
FIFO-Design-SystemVerilog/
│
├── rtl/
│   └── fifo.s
│
├── testbench/
│   └── tb_fifo.s
│
├── simulation/
│   └── waveform/
│
├── docs/
│   └── FIFO_Project_Report.pdf
│
└── README.md
```

---

## 🚀 Applications

FIFO buffers are widely used in:

* Data-rate matching
* Protocol buffering
* UART communication
* Ethernet interfaces
* USB interfaces
* PCIe data pipelines
* I2C/SPI data pipelines
* Processor-peripheral communication
* Burst data buffering

A FIFO allows a faster producer and slower consumer to exchange data safely by temporarily storing incoming data.

---

## 🎯 Learning Outcomes

Through this project, the following concepts were implemented and studied:

* SystemVerilog RTL coding
* FIFO architecture
* Circular buffer design
* Read/write pointer management
* Memory modeling
* FIFO occupancy tracking
* Full/empty detection
* Overflow/underflow protection
* Synchronous digital design
* RTL simulation and debugging
* Hardware verification

---

## 👨‍💻 Author

**Priyanshu Kumar**

B.Tech – Computer and Communication Engineering
JK Lakshmipat University, Jaipur

**Project:** 64-Stage FIFO Memory with Integrated Overflow/Underflow Protection
**Faculty:** Dr. Gaurav Mani Khanal

---

## 📚 References

1. IEEE Standard for SystemVerilog — IEEE Std 1800-2017
2. Cummings, C. E. — *Simulation and Synthesis Techniques for Asynchronous FIFO Design*
3. Mentor Graphics — *ModelSim SE User's Manual*
4. Ciletti, M. D. — *Advanced Digital Design with the Verilog HDL*

---

## ⭐ Project Highlights

```text
FIFO Depth       : 64
Data Width       : 8-bit
Pointer Width    : 6-bit
Architecture     : Circular Buffer
HDL              : SystemVerilog
Reset            : Asynchronous
Protection       : Overflow + Underflow
Status Flags     : Full + Empty + Threshold
Verification     : SystemVerilog Testbench
```

---

## 📌 Conclusion

The project implements a **parameterizable, synthesizable 64-stage × 8-bit synchronous FIFO** using SystemVerilog RTL. The design manages data using independent read/write pointers and provides protection against invalid read/write operations through full, empty, overflow, and underflow status signals. The implementation demonstrates practical RTL design and verification concepts applicable to FPGA and ASIC development.
Demo Video:-https://drive.google.com/file/d/1WP2M4FGWY1oL9BLpZUCGggLE35EaadBT/view

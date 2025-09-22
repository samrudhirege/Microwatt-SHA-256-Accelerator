# OpenFrame Overview

The OpenFrame Project provides an empty harness chip that differs significantly from the Caravel and Caravan designs. Unlike Caravel and Caravan, which include integrated SoCs and additional features, OpenFrame offers only the essential padframe, providing users with a clean slate for their custom designs.

<img width="256" alt="Screenshot 2024-06-24 at 12 53 39 PM" src="https://github.com/efabless/openframe_timer_example/assets/67271180/ff58b58b-b9c8-4d5e-b9bc-bf344355fa80">

## Key Characteristics of OpenFrame

1. **Minimalist Design:** 
   - No integrated SoC or additional circuitry.
   - Only includes the padframe, a power-on-reset circuit, and a digital ROM containing the 32-bit project ID.

2. **Padframe Compatibility:**
   - The padframe design and pin placements match those of the Caravel and Caravan chips, ensuring compatibility and ease of transition between designs.
   - Pin types are identical, with power and ground pins positioned similarly and the same power domains available.

3. **Flexibility:**
   - Provides full access to all GPIO controls.
   - Maximizes the user project area, allowing for greater customization and integration of alternative SoCs or user-specific projects at the same hierarchy level.

4. **Simplified I/O:**
   - Pins that previously connected to CPU functions (e.g., flash controller interface, SPI interface, UART) are now repurposed as general-purpose I/O, offering flexibility for various applications.

The OpenFrame harness is ideal for those looking to implement custom SoCs or integrate user projects without the constraints of an existing SoC.

## Features

1. 44 configurable GPIOs.
2. User area of approximately 15mm².
3. Supports digital, analog, or mixed-signal designs.

Overview

This project extends the Microwatt POWER ISA CPU core by adding a hardware SHA-256 accelerator. The accelerator offloads cryptographic hashing tasks from the CPU, allowing much faster execution compared to software-only implementations.

By integrating the accelerator as a memory-mapped peripheral, Microwatt can efficiently perform secure hashing operations (commonly used in authentication, blockchain, and data integrity applications).

Objectives

Design and integrate a SHA-256 hardware accelerator into Microwatt.

Provide a memory-mapped interface for communication between CPU and accelerator.

Implement and verify using test vectors from NIST.

Demonstrate improved performance over software hashing.


System Architecture
Block Diagram
 Microwatt Core
       │
       │ Wishbone / Memory-Mapped Bus
       │
 ┌─────┴──────────┐
 │  SHA-256 Core  │
 └─────┬──────────┘
       │
       ├─ MESSAGE_IN Registers (512-bit)
       ├─ CONTROL Register (start/init)
       ├─ STATUS Register (busy/done)
       └─ DIGEST_OUT Registers (256-bit)

| Address Offset | Register           | Description                       |
| -------------- | ------------------ | --------------------------------- |
| 0x00           | CONTROL            | Start / Init bits                 |
| 0x04           | STATUS             | Busy / Digest Valid flags         |
| 0x08–0x44      | MESSAGE\_IN\[0–15] | 16 × 32-bit words (512-bit block) |
| 0x48–0x64      | DIGEST\_OUT\[0–7]  | 8 × 32-bit words (256-bit digest) |


Implementation Plan
Step 1: Core Selection

Use open-source Secworks SHA256
 RTL (iterative design, 64 cycles per block).

Step 2: Wrapper Module

Develop sha_wrapper.v to connect SHA core with Microwatt bus.

Handle CONTROL/STATUS registers, input buffering, and digest storage.

Step 3: Integration

Map SHA accelerator at 0xC000_0000 in Microwatt SoC.

Modify Microwatt top-level design to include the wrapper.

Step 4: Verification

RTL testbench (sha_wrapper_tb.v) using known vectors:

"abc" → ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad

Compare hardware output with Python hashlib.sha256() reference.

Step 5: Demo Program

C program for Microwatt:

Writes 512-bit block to MESSAGE_IN.

Sets CONTROL.start.

Polls STATUS until digest valid.

Reads DIGEST_OUT registers and prints result via UART.


Expected Outcomes

Functional: SHA-256 accelerator integrated into Microwatt.

Correctness: Verified against official NIST test vectors.

Performance: Hardware computation significantly faster than software hashing.


Deliverables:

RTL source (sha_wrapper.v, integration changes).

Testbenches and simulation results.

Demo program in C.

Documentation (block diagram, FSM, memory map).


Impact

Adds cryptographic capability to an open-source POWER CPU.

Demonstrates hardware/software co-design.

Hackathon-ready: small scope, clear verification, measurable performance gain.

 Repository Structure (suggested)
microwatt-sha256-accelerator/
│── rtl/
│   ├── sha_wrapper.v
│   ├── integration_files.vhd
│── tb/
│   ├── sha_wrapper_tb.v
│── sw/
│   ├── sha_test.c
│── docs/
│   ├── architecture.md
│   ├── block_diagram.png
│   ├── fsm_diagram.png
│── README.md
# openframe_timer_example

This example implements a simple timer and connects it to the GPIOs.

## Installation and Setup

First, clone the repository:

```bash
git clone https://github.com/efabless/openframe_timer_example.git
cd openframe_timer_example
```

Then, download all dependencies:

```bash
make setup
```

## Hardening the Design

In this example, we will harden the timer. You will need to harden your own design similarly.

```bash
make user_proj_timer
```

Once you have hardened your design, integrate it into the OpenFrame wrapper:

```bash
make openframe_project_wrapper
```

## Important Notes

1. **Connecting to Power:**
   - Ensure your design is connected to power using the power pins on the wrapper.
   - Use the `vccd1_connection` and `vssd1_connection` macros, which contain the necessary vias and nets for power connections.

2. **Flattening the Design:**
   - If you plan to flatten your design within the `openframe_project_wrapper`, do not buffer the analog pins using standard cells.

3. **Running Custom Steps:**
   - Execute the custom step in OpenLane that copies the power pins from the template DEF. If this step is skipped, the precheck will fail, and your design will not be powered.

# Logism Circuits 

[![Logisim Evolution](https://img.shields.io/badge/Simulator-Logisim%20Evolution-2ea44f?style=for-the-badge)](https://github.com/logisim-evolution/logisim-evolution)
[![Target Hardware](https://img.shields.io/badge/FPGA-Intel%20DE1--SoC-0071C5?style=for-the-badge)](#-overview)
[![Toolchain](https://img.shields.io/badge/Typesetting-LaTeX-008080?style=for-the-badge)](#-overview)

A collection of digital logic designs, circuit schematics, finite state machines, and hardware simulations built across the **CSC258 (Computer Organization)** lab series using **Logisim Evolution**, **74LS TTL integrated circuits**, and **DE1-SoC** hardware synthesis

---

## Overview

This repository documents the progression from basic gate-level boolean minimization up to memory-backed state machines and video output. Across each lab, designs move through truth tables, Karnaugh map minimization, hierarchical subcircuits, test vector verification, timing diagrams, and hardware demonstration.

* **Author:** Austin Sunesh
* **Simulation & Design:** Logisim Evolution (`7404` / `7408` / `7432` TTL chips, standard gates, splitters, bit extenders, registers, RAM, and RGB display)
* **Target Hardware:** Protoboard (breadboard) & DE1-SoC
* **Documentation:** LaTeX reports (`lab*_report.tex` / `.pdf`) and exported circuit schematics

---

### L1: Building Circuits
Focused on translating boolean specifications into physical 7400-series TTL implementations (`74LS04` NOT, `74LS08` AND, `74LS32` OR) with explicit pin mappings and boolean algebra cost reduction.

* **2-to-1 Multiplexer (`Part I` & `Part III`):** Derived truth tables and schematics for $f = x\overline{s} + ys$, then minimized via De Morgan's laws to cut chip usage from 3 TTL chips down to 2 (33% reduction), alongside a gate-level schematic.
* **Boolean Function Optimization (`Part II`):** Implemented and simplified $f = \overline{a+b} + c\overline{b}$ into $\overline{b} \cdot \overline{(a \cdot \overline{c})}$, reducing chip count from 3 to 2 chips (33% cheaper).

### L2: The Design Hierarchy
Introduced modular, hierarchical circuit design by composing smaller subcircuits into larger multi-bit blocks.

* **4-to-1 Multiplexer (`mux4to1`):** Built hierarchically from `mux2to1` subcircuits across 6 inputs ($U, V, W, X, S_0, S_1$, representing $2^6 = 64$ truth table combinations).
* **Hexadecimal 7-Segment Decoder (`HEX0`–`HEX6` / `HEX_DECODER`):** Derived 16-row truth tables (`0`–`F`) and K-map-optimized boolean equations for all seven segments ($S_0$–$S_6$).

### L3: The Arithmetic Logic Unit (ALU)
Expanded into multi-bit bundles, splitters, bit extenders, test vectors, and a 6-operation ALU integrated with hex displays and DE1-SoC synthesis.

* **Full Adder & 4-Bit Ripple-Carry Adder (`full_adder`, `ripple_carry4`):** Constructed from gates and splitters without using Logisim's built-in adder, verified with targeted corner-case test vectors.
* **6-Operation ALU (`op0`–`op5` & `ALU`):** Implemented subcircuits for $A+1$ (using `ripple_carry4`), $A+B$ (using `ripple_carry4` and Logisim's built-in adder), bitwise XOR/OR across nibbles, reduction OR (`00000001` vs. `0`), and nibble packing, selected via a multiplexer.
* **Top-Level Hex & DE1-SoC Integration:** Merged `HEX_DECODER` to drive four 7-segment displays (`HEXA`, `HEXB`, `HEX_ALU1`, `HEX_ALU0`) and mapped outputs for DE1-SoC synthesis.

### L4: Synchronous Sequential Circuits
Transitioned from combinational logic to stateful storage elements, feedback loops, timing diagrams, and custom shift registers.

* **Gated D Latch & Master-Slave D Flip-Flop (`lab4_part1.circ`):** Built a NAND-based gated D latch with 7400-series chips on protoboard and Logisim, then cascaded two opposite-level latches to form a positive edge-triggered master-slave flip-flop.
* **Extended 8-Op ALU & Feedback Register (`lab4_part2.circ`):** Updated `op5` (logical left shift with sign-extended `B`) and added `op6` (arithmetic right shift with sign-extended `B`) and `op7` ($A \times B$ multiplier). Connected an 8-bit `ALUreg` feeding its lower 4 bits back into `B`, verified via timing diagrams (`0x00` $\to$ `0x0f` increment and `0x01` doubling sequence).
* **8-Bit Shift Register (`shifter_bit`, `shift_register` in `lab4_part3.circ`):** Built a 1-bit slice using `mux2to1` and a D flip-flop, then wired 8 stages with active-low parallel load (`Load_n`), shift right (`ShiftRight`), and arithmetic shift right (`ASR`) sign-extension logic.

### L5: Counters and Clocks
Explored multi-bit counters, clock division, lookup tables, and Morse code pattern generation.

* **8-Bit Counter (`counter8`):** Cascaded lower and upper 4-bit incrementers with `HEX_DECODER`s and 7-segment displays, tested with Poke.
* **Rate Divider (`rate_divider`):** Used a `CTR7` counter with a `MUX` to divide clock rates and generate periodic enable pulses, verified with timing simulations.
* **Morse Code Lookup & Main Circuit (`MORSE_LUT`):** Built a 3-to-14-bit multiplexer lookup circuit outputting fixed 14-bit Morse patterns paired with clock/rate control and display output.

### L6: Finite State Machines
Designed Moore FSMs for sequence detection and algorithmic datapath control (plus an optional restoring divider).

* **Robo-Snail Sequence Detector (`part1_FSM`, `part1_state_table` in `lab6_part1.circ`):** Built a 1-bit input (`W`), 1-bit output (`Z`) Moore FSM that smiles (`Z = 1`) on overlapping `1111` or `1101` sequences using non-trivial state encodings and timing verification.
* **Polynomial Datapath Controller (`lab6_part2.circ`):** Designed a control FSM for the provided 8-bit datapath (`R_A`, `R_B`, `R_C`, `R_X`, `R_R`, muxes, and ALU) to evaluate $Cx^2 + Bx + A$ after loading inputs over the first four clock cycles.
* **4-Bit Restoring Divider (`Part III` — Optional):** Modeled shift-and-subtract hardware division with 2's complement subtraction, conditional restoration, quotient (`q0`) updates, and remainder tracking.

### L7: Memory and Video
Combined RAM, counters, and FSM-driven datapaths to drive memory read/write operations and 2D RGB video output.

* **RAM & Counter Memory Test (`Part I`):** Wired a `CTR5` counter to a `32 x 4` RAM module and hex display controlled by write/output enable signals (`OBWE`) and clock pulses.
* **16×16 RGB Square Drawer (`ControlUnit`, `Datapath`, `Next_S0`, `Next_S1` in `lab7_part2`):** Implemented a 3-state FSM (`wait`, `draw`, `complete`) driving an 8-bit counter (`CTR8`) and coordinate adders (`X_in`/`Y_in` to `X_out`/`Y_out`) to draw a $16 \times 16$ square on the RGB display.

---

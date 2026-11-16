# Day 02 – Timing Libraries, Hierarchical Synthesis and Arithmetic Optimization

## Overview

This exercise explores standard-cell timing libraries, hierarchical and flattened synthesis, sequential circuit implementation, technology mapping and arithmetic optimization using Verilog HDL and Yosys.

## Tools Used
- **RTL Design:** Verilog HDL
- **Simulation:** Icarus Verilog
- **Waveform Analysis:** GTKWave
- **Synthesis:** Yosys
- **Technology Library:** SKY130 HD
- **Graph Visualization:** Graphviz

## 1. SKY130 Liberty Library Exploration

Explored the SKY130 HD typical-corner Liberty file:

`sky130_fd_sc_hd__tt_025C_1v80.lib`

Inspected:
- Standard-cell definitions.
- Cell area.
- Input and output pin directions.
- Timing arcs.
- Cell rise and fall timing information.
- Different drive-strength variants.

Examined standard cells including:
- AND2
- OR2
- NAND2
- NOR2
- INV

## 2. Hierarchical RTL Design

Implemented a multi-module design consisting of two submodules:

- `sub_module1`: AND logic.
- `sub_module2`: OR logic.
- `multiple_modules`: Top-level module instantiating both submodules.

**Boolean expression:**

\[
Y=(A\cdot B)+C
\]

Performed hierarchical synthesis using Yosys and examined the module hierarchy.

## 3. Flattened Synthesis

Applied the Yosys flattening process to remove submodule boundaries and combine the logic into the top-level design.

Generated:
- Hierarchical netlist.
- Flattened netlist.
- Hierarchical graphical representation.
- Flattened graphical representation.

Compared the hierarchical and flattened design structures.

## 4. Sequential Circuit Design and Simulation

Implemented and simulated three types of D flip-flops.

### 4.1 DFF with Asynchronous Reset

- Implemented a positive-edge-triggered DFF.
- Reset is active HIGH.
- Output immediately becomes LOW when reset is asserted, independent of the clock.
- Verified using Icarus Verilog and GTKWave.

**Files:**
- `dff_asyncres.v`
- `tb_dff_asyncres.v`
- `tb_dff_asyncres.vcd`

### 4.2 DFF with Asynchronous Set

- Implemented a positive-edge-triggered DFF.
- Set is active HIGH.
- Output immediately becomes HIGH when set is asserted, independent of the clock.
- Verified using GTKWave.

**Files:**
- `dff_async_set.v`
- `tb_dff_async_set.v`
- `tb_dff_async_set.vcd`

### 4.3 DFF with Synchronous Reset

- Implemented a positive-edge-triggered DFF.
- Reset is active HIGH.
- Output becomes LOW only at the rising edge of the clock when reset is asserted.
- Verified the difference between synchronous and asynchronous reset behavior.

**Files:**
- `dff_syncres.v`
- `tb_dff_syncres.v`
- `tb_dff_syncres.vcd`

## 5. SKY130 Technology Mapping of DFFs

Mapped the three sequential designs to SKY130 HD standard cells using Yosys.

| Design | SKY130 Standard Cells |
|---|---|
| Asynchronous Reset | `sky130_fd_sc_hd__clkinv_1`, `sky130_fd_sc_hd__dfrtp_1` |
| Asynchronous Set | `sky130_fd_sc_hd__clkinv_1`, `sky130_fd_sc_hd__dfstp_2` |
| Synchronous Reset | `sky130_fd_sc_hd__dfxtp_1`, `sky130_fd_sc_hd__nor2b_1` |

Generated mapped Verilog netlists and graphical representations for all three designs.

## 6. Arithmetic Operations Using Verilog

Explored constant multiplication and synthesis optimization.

### 6.1 Multiplication by 2

**File:** `mult_2.v`

Implemented multiplication of a 3-bit input by 2.

\[
Y=A\times2
\]

Yosys optimized the multiplication into a one-bit left shift:

\[
Y=\{A,1'b0\}
\]

Generated the synthesized and SKY130-mapped netlists.

### 6.2 Multiplication by 9

**File:** `mult_8.v`

Implemented multiplication of a 3-bit input by 9.

\[
Y=A\times9
\]

Using the identity:

\[
A\times9=A\times(8+1)
\]

\[
Y=(A\ll3)+A
\]

For a 3-bit input, the output is 6 bits wide.

Generated the synthesized netlist and examined the resulting logic implementation.

**Note:** The source filename is `mult_8.v`, but the implemented operation is multiplication by 9 (`a * 9`).

## 7. Key Learnings

- Understanding Liberty timing libraries and standard-cell parameters.
- Exploring standard-cell drive strengths and timing arcs.
- Hierarchical RTL design and module instantiation.
- Hierarchical versus flattened synthesis.
- Sequential circuit design and reset behavior.
- SKY130 standard-cell technology mapping.
- DFF mapping using Yosys `dfflibmap`.
- Combinational technology mapping using ABC.
- Arithmetic optimization and constant multiplication.

## Conclusion

Successfully explored the SKY130 HD standard-cell library, implemented hierarchical RTL designs, simulated sequential circuits, performed technology mapping and investigated arithmetic optimization using Yosys.

# Day 01 – RTL Design, Simulation and Synthesis

## Overview
This exercise focuses on the RTL design and implementation of a 2:1 multiplexer using Verilog HDL. The design was simulated, verified using GTKWave, synthesized using Yosys and mapped to the SKY130 High-Density standard-cell library.

## Tools Used
- **RTL Design:** Verilog HDL
- **Simulation:** Icarus Verilog
- **Waveform Analysis:** GTKWave
- **Synthesis:** Yosys
- **Technology Library:** SKY130 HD
- **Visualization:** Graphviz

## 1. RTL Design – 2:1 Multiplexer

Implemented a 2:1 multiplexer using Verilog.

**Functionality:**

\[
Y = \overline{S}I_0 + SI_1
\]

The output is selected based on the select input.

**Source file:** `good_mux.v`

## 2. Functional Simulation

- Developed a Verilog testbench.
- Simulated the multiplexer using Icarus Verilog.
- Generated a VCD waveform.
- Verified the output for different combinations of select and data inputs using GTKWave.

**Files:**
- `tb_good_mux.v`
- `tb_good_mux.vcd`

## 3. RTL Synthesis Using Yosys

Performed RTL synthesis to convert the behavioral Verilog description into a synthesized netlist.

## 4. SKY130 Technology Mapping

Mapped the synthesized multiplexer to the SKY130 HD standard-cell library using Yosys and ABC.

**Mapped cell:**

`sky130_fd_sc_hd__mux2_1`

**Mapped cell area:** 11.2608 µm²

The synthesis tool selected a dedicated multiplexer standard cell for the implementation.

## 5. Schematic Visualization

Generated a graphical representation of the SKY130-mapped netlist using Yosys and Graphviz.

## Key Learnings
- RTL coding and functional verification.
- Testbench development and waveform analysis.
- Understanding the RTL-to-netlist synthesis process.
- Working with Liberty standard-cell libraries.
- Technology mapping using ABC.
- Understanding standard-cell netlists and schematic representations.

## Repository Structure

```text
Day-01/
├── RTL/
│   ├── good_mux.v
│   └── good_mux_sky130.v
├── Testbench/
│   └── tb_good_mux.v
├── Waveforms/
│   └── tb_good_mux.vcd
└── Screenshots/
    ├── good_mux_sky130.dot
    ├── good_mux_sky130.svg
    ├── good_mux_sky130_corrected.dot
    └── good_mux_sky130_corrected.svg
```

## Conclusion

Successfully implemented and verified a 2:1 multiplexer, performed synthesis using Yosys, and mapped the design to the SKY130 HD standard-cell library.

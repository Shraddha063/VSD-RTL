# Day 01 – RTL Design, Simulation and Synthesis of a 2:1 MUX

## Objective

To design and verify a 2:1 Multiplexer using Verilog HDL, simulate its functionality using Icarus Verilog, inspect the waveform using GTKWave, and perform logic synthesis and SKY130 standard-cell mapping using Yosys.

## Tools Used

- Verilog HDL
- Icarus Verilog
- GTKWave
- Yosys
- SKY130 HD Standard Cell Library
- Graphviz

---

## 1. RTL Design – 2:1 Multiplexer

A 2:1 multiplexer selects one of two input signals based on the select signal.

### Verilog Code – `good_mux.v`

```verilog
module good_mux (
    input i0,
    input i1,
    input sel,
    output y
);

assign y = sel ? i1 : i0;

endmodule
```

### Boolean Expression

\[
Y = \overline{S}I_0 + SI_1
\]

### Truth Table

| Select | I0 | I1 | Output |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 |
| 0 | 1 | 0 | 1 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 1 |
| 1 | 1 | 0 | 0 |
| 1 | 1 | 1 | 1 |

---

## 2. Testbench Development

The testbench applies different combinations of input signals and verifies the MUX output through waveform inspection.

### Verilog Code – `tb_good_mux.v`

```verilog
module tb_good_mux;

reg i0, i1, sel;
wire y;

good_mux uut (
    .i0(i0),
    .i1(i1),
    .sel(sel),
    .y(y)
);

initial begin
    $dumpfile("tb_good_mux.vcd");
    $dumpvars(0, tb_good_mux);

    i0 = 0; i1 = 0; sel = 0;
    #10;

    i0 = 0; i1 = 1; sel = 0;
    #10;

    i0 = 1; i1 = 0; sel = 0;
    #10;

    i0 = 1; i1 = 1; sel = 0;
    #10;

    i0 = 0; i1 = 0; sel = 1;
    #10;

    i0 = 0; i1 = 1; sel = 1;
    #10;

    i0 = 1; i1 = 0; sel = 1;
    #10;

    i0 = 1; i1 = 1; sel = 1;
    #10;

    $finish;
end

endmodule
```

---
![2:1 MUX and Testbench Code in gvim](Screenshots/Screenshot%202026-11-03%20194919.png)
## 3. Simulation Using Icarus Verilog

### Step 1: Navigate to the Day-01 directory

```bash
cd ~/VLSI/VSD-RTL/Day-01
```

### Step 2: Compile the RTL and testbench

```bash
iverilog -o mux_sim RTL/good_mux.v Testbench/tb_good_mux.v
```

### Step 3: Execute the simulation

```bash
vvp mux_sim
```

### Step 4: Open the generated waveform

```bash
mv tb_good_mux.vcd Waveforms/
gtkwave Waveforms/tb_good_mux.vcd
```

### Simulation Output

The waveform confirms that the output follows the selected input:

- When `sel = 0`, output `y` follows `i0`.
- When `sel = 1`, output `y` follows `i1`.

### Waveform Screenshot

Save your GTKWave screenshot as `mux_waveform.png` inside the `Screenshots` folder.

![2:1 MUX simulation waveform]( Screenshots/Screenshot%202026-11-03%20192815.png)

---

## 4. RTL Synthesis Using Yosys

Yosys is used to convert the RTL description into a synthesized logic representation.

### Step 1: Navigate to the RTL directory

```bash
cd ~/VLSI/VSD-RTL/Day-01
```

### Step 2: Run generic synthesis

```bash
yosys
```

Enter the following commands in the Yosys prompt:

```tcl
read_verilog RTL/good_mux.v
hierarchy -top good_mux
synth -top good_mux
stat
write_verilog RTL/good_mux_generic.v
```

Exit Yosys:

```tcl
exit
```

The synthesized generic netlist is saved as `RTL/good_mux_generic.v`.

---

## 5. SKY130 Technology Mapping

The generic logic is mapped to standard cells from the SKY130 HD library.

### Liberty File

```bash
LIB=/home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Run technology mapping

```bash
yosys -p "
read_liberty -lib $LIB
read_verilog RTL/good_mux.v
hierarchy -top good_mux
synth -top good_mux
abc -liberty $LIB
clean
stat -liberty $LIB
write_verilog -noattr RTL/good_mux_sky130.v
" | tee sky130_mapping.log
```

### Mapping Result

The synthesized MUX was mapped to the SKY130 standard cell:

`sky130_fd_sc_hd__mux2_1`

Reported cell area:

**11.2608 µm²**

The dedicated MUX cell was selected by the technology-mapping process.

### Mapped Netlist

The generated mapped netlist is:

`RTL/good_mux_sky130.v`



### Terminal Output Screenshot

Save your Yosys mapping terminal screenshot as `mux_mapping_terminal.png`.



---

## 6. Schematic Generation

The mapped netlist can be visualized using Yosys and Graphviz.

### Commands

```bash
cd ~/VLSI/VSD-RTL/Day-01

LIB=/home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

yosys -p "
read_liberty -lib $LIB
read_verilog RTL/good_mux_sky130.v
hierarchy -top good_mux
show -format dot -prefix Screenshots/good_mux_sky130
"
```

Convert the generated DOT file to SVG:

```bash
dot -Tsvg Screenshots/good_mux_sky130.dot -o Screenshots/good_mux_sky130.svg
```

### Schematic

[View the mapped netlist](Screenshots/Screenshot%202026-11-15%20203752.png) 

---

## 7. Key Learnings

- Developed a 2:1 MUX using Verilog HDL.
- Verified RTL functionality through simulation.
- Inspected signal transitions using GTKWave.
- Understood the RTL-to-generic-netlist synthesis process.
- Performed technology mapping using the SKY130 HD standard-cell library.
- Visualized the mapped design using Yosys and Graphviz.

---

## Directory Structure

```text
Day-01/
├── README.md
├── RTL/
│   ├── good_mux.v
│   ├── good_mux_generic.v
│   └── good_mux_sky130.v
├── Testbench/
│   └── tb_good_mux.v
├── Waveforms/
│   └── tb_good_mux.vcd
└── Screenshots/
    ├── mux_waveform.png
    ├── mux_mapping_terminal.png
    └── good_mux_sky130.svg
```


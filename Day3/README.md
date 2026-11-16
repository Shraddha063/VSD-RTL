
# Day 03 – Introduction to Logic Optimizations

## Objective

The objective of Day 03 is to understand how Yosys performs logic optimization on Verilog RTL designs.

The experiments cover:

- Combinational logic optimization
- Boolean expression simplification
- Hierarchical and multiple-module optimization
- Constant propagation
- Flip-flop optimization
- Sequential logic optimization
- Counter optimization
- Removal of unused logic and redundant hardware

## Tools Used

| Tool | Purpose |
|---|---|
| Yosys | RTL synthesis and logic optimization |
| Icarus Verilog | RTL simulation |
| GTKWave | Waveform visualization |
| Ubuntu | Linux environment |

## Project Structure

```text
Day-03-Introduction-to-Logic-Optimizations/
│
├── RTL/
├── Testbench/
├── Netlists/
├── Waveforms/
├── Screenshots/
└── README.md
```

---

# 1. Combinational Logic Optimization

## 1.1 opt_check

**RTL file:** `RTL/opt_check.v`

The conditional expression:

`assign y = a ? b : 0;`

simplifies to:

`assign y = a & b;`

[![opt_check RTL](./Screenshots/Screenshot%202026-11-16%20161749.png)](./Screenshots/Screenshot%202026-11-16%20161749.png)

[![opt_check synthesis](./Screenshots/Screenshot%202026-11-16%20161856.png)](./Screenshots/Screenshot%202026-11-16%20161856.png)

[![opt_check netlist](./Screenshots/Screenshot%202026-11-16%20162003.png)](./Screenshots/Screenshot%202026-11-16%20162003.png)

## 1.2 opt_check2

**RTL file:** `RTL/opt_check2.v`

`assign y = a ? 1 : b;`

The expression simplifies to:

`assign y = a | b;`

The output is high when either input is high.

[![opt_check2 result](./Screenshots/Screenshot%202026-11-16%20162558.png)](./Screenshots/Screenshot%202026-11-16%20162558.png)

## 1.3 opt_check3

**RTL file:** `RTL/opt_check3.v`

`assign y = a ? (c ? b : 0) : 0;`

The simplified Boolean expression is:

`assign y = a & b & c;`

The output is high only when all three inputs are high.

[![opt_check3 result](./Screenshots/Screenshot%202026-11-16%20162843.png)](./Screenshots/Screenshot%202026-11-16%20162843.png)

## 1.4 opt_check4

**RTL file:** `RTL/opt_check4.v`

`assign y = a ? (b ? (a & c) : c) : (!c);`

The expression simplifies to:

`assign y = ~(a ^ c);`

This is an XNOR operation. Input `b` becomes irrelevant after optimization.

[![opt_check4 result](./Screenshots/Screenshot%202026-11-16%20162929.png)](./Screenshots/Screenshot%202026-11-16%20162929.png)

---

# 2. Multiple Module Optimization

**RTL files:**
- `RTL/multiple_module_opt.v`
- `RTL/multiple_module_opt2.v`

This experiment demonstrates optimization across hierarchical module instances.

Yosys identifies unused submodule outputs and removes logic that does not contribute to the primary output.

For the first design, the resulting Boolean expression is:

`assign y = c | (a & b);`

The unused XOR logic is eliminated.

The second design demonstrates complete logic removal when the output is constant.

### Synthesis observations

- Unused module instances are removed.
- Redundant intermediate signals are eliminated.
- Constant propagation simplifies expressions.
- Only logic contributing to the output is retained.

[![multiple module optimization](./Screenshots/Screenshot%202026-11-16%20163227.png)](./Screenshots/Screenshot%202026-11-16%20163227.png)

[![multiple module optimization result](./Screenshots/Screenshot%202026-11-16%20163311.png)](./Screenshots/Screenshot%202026-11-16%20163311.png)

---

# 3. Flip-Flop Optimization

The `dff_const` experiments demonstrate how Yosys optimizes sequential logic based on reset values and next-state behavior.

| Design | Optimization observation |
|---|---|
| `dff_const1` | Sequential behavior retained |
| `dff_const2` | Constant output; all cells removed |
| `dff_const3` | Sequential dependency retained |
| `dff_const4` | Constant output; all cells removed |
| `dff_const5` | Two flip-flops retained |

## 3.1 dff_const1

**RTL:** `RTL/dff_const1.v`

The output is reset to 0 and assigned 1 during normal operation.

The output depends on the reset and clock behavior.

[![dff_const1 result](./Screenshots/Screenshot%202026-11-16%20163342.png)](./Screenshots/Screenshot%202026-11-16%20163342.png)

## 3.2 dff_const2

**RTL:** `RTL/dff_const2.v`

The output is assigned 1 during both reset and normal operation.

Yosys identifies the output as constant and removes the sequential logic.

**Observed result:** Zero cells remain in the optimized design.

## 3.3 dff_const3

**RTL:** `RTL/dff_const3.v`

This design uses an internal register `q1`, which feeds the output register `q`.

`q1 <= 1'b1;`

`q <= q1;`

Because nonblocking assignments use the previous value of `q1` for the assignment to `q`, sequential behavior must be preserved.

[![dff_const3 result](./Screenshots/Screenshot%202026-11-16%20163916.png)](./Screenshots/Screenshot%202026-11-16%20163916.png)

## 3.4 dff_const4

**RTL:** `RTL/dff_const4.v`

Both registers are assigned 1 during reset and normal operation.

Yosys identifies the output as a constant and removes the flip-flop logic.

**Observed result:** Zero cells remain.

[![dff_const4 result](./Screenshots/Screenshot%202026-11-16%20163935.png)](./Screenshots/Screenshot%202026-11-16%20163935.png)

## 3.5 dff_const5

**RTL:** `RTL/dff_const5.v`

The design uses two registers:

`q1 <= 1'b1;`

`q <= q1;`

The output depends on the previous state of `q1`.

**Observed result:** Two flip-flops are retained in the synthesized design.

[![dff_const5 result](./Screenshots/Screenshot%202026-11-16%20164102.png)](./Screenshots/Screenshot%202026-11-16%20164102.png)

---

# 4. Counter Optimization

## 4.1 counter_opt

**RTL:** `RTL/counter_opt.v`

The design describes a 3-bit counter, but only the least significant bit is connected to the output.

`assign q = count[0];`

The upper two bits do not affect the observable output.

**Observed Yosys result:**

| Cell | Count |
|---|---:|
| `$_DFF_PP0_` | 1 |
| `$_NOT_` | 1 |
| **Total** | **2** |

The 3-bit counter is optimized into one flip-flop and one inverter.

[![counter_opt result](./Screenshots/Screenshot%202026-11-16%20194327.png)](./Screenshots/Screenshot%202026-11-16%20194327.png)

## 4.2 counter_opt2

**RTL:** `RTL/counter_opt2.v`

The output depends on a comparison involving all three counter bits:

`assign q = (count[2:0] == 3'b100);`

All three counter bits are required for this comparison.

**Observed generic Yosys result:**

| Cell | Count |
|---|---:|
| `$add` | 1 |
| `$adff` | 1 |
| `$eq` | 1 |
| **Total** | **3** |

Unlike `counter_opt`, this design retains the complete counter state.

[![counter_opt2 result](./Screenshots/Screenshot%202026-11-16%20194704.png)](./Screenshots/Screenshot%202026-11-16%20194704.png)

---

# 5. Simulation Using Icarus Verilog and GTKWave

The existing testbenches are stored in the `Testbench/` directory.

Example simulation commands:

### opt_check2

```bash
iverilog -g2012 -o Waveforms/opt_check2_sim \
RTL/opt_check2.v Testbench/tb_opt_check2.v

vvp Waveforms/opt_check2_sim
```

### dff_const5

```bash
iverilog -g2012 -o Waveforms/dff_const5_sim \
RTL/dff_const5.v Testbench/tb_dff_const5.v

vvp Waveforms/dff_const5_sim
```

### counter_opt

```bash
iverilog -g2012 -o Waveforms/counter_opt_sim \
RTL/counter_opt.v Testbench/tb_counter_opt.v

vvp Waveforms/counter_opt_sim
```

### counter_opt2

```bash
iverilog -g2012 -o Waveforms/counter_opt2_sim \
RTL/counter_opt2.v Testbench/tb_counter_opt2.v

vvp Waveforms/counter_opt2_sim
```

Open a generated waveform using:

```bash
gtkwave Waveforms/tb_counter_opt2.vcd
```

---

# 6. Yosys Synthesis Commands

Run these commands from the Day-03 project directory.

### Multiple-module optimization

```bash
yosys -p "read_verilog RTL/multiple_module_opt.v; hierarchy -top multiple_module_opt; synth -top multiple_module_opt; opt_clean -purge; stat; write_verilog -noattr Netlists/multiple_module_opt_netlist.v"
```

### Multiple-module optimization 2

```bash
yosys -p "read_verilog RTL/multiple_module_opt2.v; hierarchy -top multiple_module_opt2; synth -top multiple_module_opt2; opt_clean -purge; stat; write_verilog -noattr Netlists/multiple_module_opt2_netlist.v"
```

### dff_const2

```bash
yosys -p "read_verilog RTL/dff_const2.v; hierarchy -top dff_const2; synth -top dff_const2; opt_clean -purge; stat; write_verilog -noattr Netlists/dff_const2_netlist.v"
```

### dff_const3

```bash
yosys -p "read_verilog RTL/dff_const3.v; hierarchy -top dff_const3; synth -top dff_const3; opt_clean -purge; stat; write_verilog -noattr Netlists/dff_const3_netlist.v"
```

### dff_const4

```bash
yosys -p "read_verilog RTL/dff_const4.v; hierarchy -top dff_const4; synth -top dff_const4; opt_clean -purge; stat; write_verilog -noattr Netlists/dff_const4_netlist.v"
```

### dff_const5

```bash
yosys -p "read_verilog RTL/dff_const5.v; hierarchy -top dff_const5; synth -top dff_const5; opt_clean -purge; stat; write_verilog -noattr Netlists/dff_const5_netlist.v"
```

### counter_opt

```bash
yosys -p "read_verilog RTL/counter_opt.v; hierarchy -top counter_opt; synth -top counter_opt; opt_clean -purge; stat; write_verilog -noattr Netlists/counter_opt_netlist.v"
```

### counter_opt2

```bash
yosys -p "read_verilog RTL/counter_opt2.v; hierarchy -top counter_opt2; synth -top counter_opt2; opt_clean -purge; stat; write_verilog -noattr Netlists/counter_opt2_netlist.v"
```

---

# 7. Important Yosys Commands

| Command | Function |
|---|---|
| `read_verilog` | Reads Verilog source |
| `hierarchy -top` | Defines the top module |
| `proc` | Converts procedural blocks into internal logic |
| `opt` | Performs optimization |
| `opt_clean` | Removes unused cells and wires |
| `opt_clean -purge` | Removes unused objects and internal remnants |
| `flatten` | Flattens hierarchical modules |
| `synth` | Performs synthesis |
| `abc` | Performs logic optimization and technology mapping |
| `stat` | Displays synthesis statistics |
| `write_verilog` | Exports the synthesized netlist |
| `show` | Displays the synthesized logic graph |

---

# 8. Key Learnings

1. Yosys simplifies conditional expressions into equivalent Boolean logic.
2. Unused module instances and intermediate signals can be eliminated.
3. Constant propagation can remove flip-flops when their outputs are constant.
4. Sequential dependencies must be preserved when they affect future output values.
5. Counter bits that do not contribute to observable outputs can be removed.
6. Counter bits required for output comparisons must be retained.

## Conclusion

Day 03 demonstrates that synthesis is not simply a direct translation of RTL into hardware. Yosys analyzes the design, performs Boolean simplification, propagates constants, eliminates unused logic, and optimizes sequential elements while preserving the intended behavior.

The experiments highlight the importance of observable outputs and sequential dependencies in determining the final synthesized hardware.

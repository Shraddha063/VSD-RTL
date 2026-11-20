# Day 04 – RTL Simulation, Gate-Level Simulation and Verilog Coding Concepts

## Objective

The objective of Day 04 is to understand RTL simulation, gate-level simulation (GLS), and how Verilog coding styles affect simulation and synthesis results.

Experiments covered:

- 2:1 MUX using the ternary operator
- Incomplete sensitivity lists and simulation-synthesis mismatch
- Correct combinational MUX using `always @(*)`
- Blocking assignments and statement execution order
- RTL simulation and functional gate-level simulation

## Tools Used

| Tool | Purpose |
|---|---|
| Icarus Verilog | RTL and gate-level simulation |
| Yosys | RTL synthesis |
| GTKWave | Waveform visualization |
| Sky130 HD Library | Standard-cell library |
| Ubuntu | Linux development environment |

---

## 1. Ternary Operator MUX

**RTL file:** `ternary_operator_mux.v`  
**Testbench:** `tb_ternary_operator_mux.v`

A 2:1 multiplexer is implemented using the Verilog conditional (ternary) operator.

```verilog
assign y = sel ? i1 : i0;
```

Expected behaviour:

- `sel = 0` → `y = i0`
- `sel = 1` → `y = i1`

The testbench changes `i0`, `i1`, and `sel` at different time intervals to verify the MUX functionality.

### Simulation and synthesis results

![Ternary MUX RTL simulation](Screenshots/Screenshot 2026-11-19 111449.png)

![Ternary MUX synthesis result](Screenshots/Screenshot 2026-11-19 112325.png)

![Ternary MUX gate-level simulation](Screenshots/Screenshot 2026-11-19 220033.png)

---

## 2. Incomplete Sensitivity List – Bad MUX

**RTL file:** `bad_mux.v`  
**Testbench:** `tb_bad_mux.v`

The `bad_mux` design uses an incomplete sensitivity list:

```verilog
always @(sel)
begin
    if (sel)
        y <= i1;
    else
        y <= i0;
end
```

The sensitivity list contains only `sel`. Therefore, changes in `i0` or `i1` do not trigger the `always` block unless `sel` also changes.

As a result, RTL simulation may retain an outdated output value even when the selected input changes.

However, synthesis interprets the intended combinational logic and generates a MUX circuit. This can cause a mismatch between RTL simulation and gate-level simulation.

### Simulation and synthesis results

![Bad MUX RTL simulation](Screenshots/Screenshot 2026-11-19 220258.png)

![Bad MUX synthesis result](Screenshots/Screenshot 2026-11-19 221100.png)

![Bad MUX gate-level simulation](Screenshots/Screenshot 2026-11-19 221258.png)

**Observation:** The RTL output does not consistently respond to changes in the data inputs while the select signal remains unchanged. The synthesized MUX behaves as a combinational circuit, demonstrating the RTL-to-GLS mismatch.

---

## 3. Correct Combinational MUX – Good MUX

**RTL file:** `good_mux.v`  
**Testbench:** `tb_good_mux.v`

The incomplete sensitivity list is corrected using `always @(*)`.

```verilog
always @(*)
begin
    if (sel)
        y <= i1;
    else
        y <= i0;
end
```

Using `always @(*)` automatically includes all signals read inside the combinational block in the sensitivity list.

Therefore, the output responds whenever `i0`, `i1`, or `sel` changes.

### Simulation results

![Good MUX RTL simulation](Screenshots/Screenshot 2026-11-19 223304.png)

![Good MUX gate-level simulation](Screenshots/Screenshot 2026-11-19 223605.png)

**Observation:** The output follows the selected input correctly. RTL and gate-level simulation demonstrate the expected combinational MUX behaviour.

---

## 4. Blocking Assignment Caveat

**RTL file:** `blocking_caveat.v`  
**Testbench:** `tb_blocking_caveat.v`

This experiment demonstrates how the order of blocking assignments affects RTL simulation.

The original RTL code is:

```verilog
reg x;

always @(*)
begin
    d = x & c;
    x = a | b;
end
```

Blocking assignments (`=`) execute sequentially in the order in which they are written.

Here:

1. `d = x & c` executes first and uses the previously available value of `x`.
2. `x = a | b` executes afterward.

Consequently, RTL simulation can exhibit unexpected output behaviour because `d` is evaluated before the updated value of `x` is assigned.

The intended combinational expression is:

\[
d = (a | b) \& c
\]

To avoid the read-before-update issue, the statements should be reordered:

```verilog
always @(*)
begin
    x = a | b;
    d = x & c;
end
```

### RTL source and testbench

![Blocking assignment RTL and testbench](Screenshots/Screenshot 2026-11-20 090502.png)

### Simulation waveforms

![Blocking assignment waveform – View 1](Screenshots/Screenshot 2026-11-19 223845.png)

**Observation:** The experiment demonstrates the effect of procedural statement ordering in a combinational block and why the order of blocking assignments matters.

---

## 5. Key Learnings

1. A 2:1 MUX can be implemented using the ternary operator or an `always` block.
2. An incomplete sensitivity list can cause RTL simulation to retain stale output values.
3. `always @(*)` is preferred for combinational logic because it automatically includes the signals read by the block.
4. Blocking assignments (`=`) execute immediately and follow the order of statements.
5. The order of assignments in a combinational block can affect RTL simulation behaviour.
6. RTL simulation and gate-level simulation help identify functional mismatches between behavioural RTL and synthesized hardware.
7. Functional GLS uses standard-cell Verilog models to simulate the synthesized netlist.

---

## Repository Structure

```text
Day-04/
│
├── verilog_files/
│   ├── ternary_operator_mux.v
│   ├── tb_ternary_operator_mux.v
│   ├── bad_mux.v
│   ├── tb_bad_mux.v
│   ├── good_mux.v
│   ├── tb_good_mux.v
│   ├── blocking_caveat.v
│   └── tb_blocking_caveat.v
│
├── Screenshots/
│   ├── Screenshot 2026-11-19 111449.png
│   ├── Screenshot 2026-11-19 112325.png
│   ├── Screenshot 2026-11-19 220033.png
│   ├── Screenshot 2026-11-19 220258.png
│   ├── Screenshot 2026-11-19 221100.png
│   ├── Screenshot 2026-11-19 221258.png
│   ├── Screenshot 2026-11-19 223304.png
│   ├── Screenshot 2026-11-19 223605.png
│   ├── Screenshot 2026-11-19 223845.png
│   └── Screenshot 2026-11-20 090502.png
│
└── README.md
```

**Note:** Keep the `Screenshots/` folder and `README.md` together in the Day 04 directory. The image paths are relative to that directory.

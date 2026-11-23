# Day 05 – RTL Design and Synthesis

**VLSI System Design | RTL Design | SKY130**

## Overview

Day 05 covers combinational logic coding styles, latch inference, procedural loops, multiplexers, demultiplexers, full adders and ripple-carry adders.

Tools used:
- Icarus Verilog
- GTKWave
- Yosys
- Graphviz

---

## 1. Incomplete and Complete If Statements

An incomplete assignment in a combinational block can infer a latch because the output must retain its previous value when no assignment is executed.

### Incomplete If

```verilog
always @(*) begin
    if (i0)
        y = i1;
end
```

**Synthesis observation:** A latch is inferred because `y` is not assigned when `i0 = 0`.

![Incomplete If waveform](Screenshots/Screenshot%202026-11-22%20104348.png)

![Incomplete If schematic](Screenshots/Screenshot%202026-11-22%20104639.png)

### Complete If-Else

```verilog
always @(*) begin
    if (i0)
        y = i1;
    else
        y = 1'b0;
end
```

**Synthesis observation:** Assigning `y` in both branches avoids latch inference.

![Complete If waveform](Screenshots/Screenshot%202026-11-22%20104812.png)

![Complete If simulation](Screenshots/Screenshot%202026-11-22%20105605.png)

![Complete If schematic](Screenshots/Screenshot%202026-11-22%20110137%281%29.png)

---

## 2. 4:1 MUX Using a Procedural For Loop

### RTL Code

```verilog
module mux_generate (
    input i0, input i1, input i2, input i3,
    input [1:0] sel,
    output reg y
);

    wire [3:0] i_int;
    assign i_int = {i3,i2,i1,i0};

    integer k;

    always @(*) begin
        for(k = 0; k < 4; k = k + 1) begin
            if(k == sel)
                y = i_int[k];
        end
    end

endmodule
```

The select signal determines which input is routed to the output.

| Select | Output |
|---|---|
| 00 | i0 |
| 01 | i1 |
| 10 | i2 |
| 11 | i3 |

### Simulation

![MUX using loop waveform](Screenshots/Screenshot%202026-11-22%20111204.png)

### Synthesis

![MUX using loop schematic](Screenshots/Screenshot%202026-11-22%20111554.png)

---

## 3. 1:8 DEMUX Using a Procedural For Loop

### RTL Code

```verilog
module demux_generate (
    output o0, output o1, output o2, output o3,
    output o4, output o5, output o6, output o7,
    input [2:0] sel,
    input i
);

    reg [7:0] y_int;

    assign {o7,o6,o5,o4,o3,o2,o1,o0} = y_int;

    integer k;

    always @(*) begin
        y_int = 8'b0;

        for(k = 0; k < 8; k++) begin
            if(k == sel)
                y_int[k] = i;
        end
    end

endmodule
```

The 3-bit select input activates one of the eight outputs. All remaining outputs are zero.

### Simulation

![DEMUX using loop waveform](Screenshots/Screenshot%202026-11-22%20214956.png)

### Synthesis

![DEMUX using loop schematic](Screenshots/Screenshot%202026-11-22%20112204.png)

---

## 4. 1:8 DEMUX Using a Case Statement

### RTL Code

```verilog
module demux_case (
    output o0, output o1, output o2, output o3,
    output o4, output o5, output o6, output o7,
    input [2:0] sel,
    input i
);

    reg [7:0] y_int;

    assign {o7,o6,o5,o4,o3,o2,o1,o0} = y_int;

    always @(*) begin
        y_int = 8'b0;

        case(sel)
            3'b000: y_int[0] = i;
            3'b001: y_int[1] = i;
            3'b010: y_int[2] = i;
            3'b011: y_int[3] = i;
            3'b100: y_int[4] = i;
            3'b101: y_int[5] = i;
            3'b110: y_int[6] = i;
            3'b111: y_int[7] = i;
        endcase
    end

endmodule
```

**Observation:** The case-based implementation performs the same DEMUX operation as the loop-based implementation.

### Simulation and Synthesis Results

![DEMUX case waveform](Screenshots/Screenshot%202026-11-22%20220247.png)

![DEMUX case schematic](Screenshots/Screenshot%202026-11-22%20221417.png)

---

## 5. Full Adder

### RTL Code

```verilog
module fa (
    input a,
    input b,
    input c,
    output co,
    output sum
);

    assign {co,sum} = a + b + c;

endmodule
```

A full adder accepts three one-bit inputs and produces a sum and carry output.

---

## 6. 8-bit Ripple Carry Adder (RCA)

The ripple-carry adder is constructed by cascading eight full-adder modules.

### RTL Code

```verilog
module rca (
    input [7:0] num1,
    input [7:0] num2,
    output [8:0] sum
);

    wire [7:0] int_sum;
    wire [7:0] int_co;

    genvar i;

    generate
        for (i = 1; i < 8; i = i + 1) begin
            fa u_fa_1 (
                .a(num1[i]),
                .b(num2[i]),
                .c(int_co[i-1]),
                .co(int_co[i]),
                .sum(int_sum[i])
            );
        end
    endgenerate

    fa u_fa_0 (
        .a(num1[0]),
        .b(num2[0]),
        .c(1'b0),
        .co(int_co[0]),
        .sum(int_sum[0])
    );

    assign sum[7:0] = int_sum;
    assign sum[8] = int_co[7];

endmodule
```

### Working Principle

1. The first full adder receives a carry input of zero.
2. Each subsequent full adder receives the carry from the previous stage.
3. The final carry is connected to `sum[8]`.
4. The 9-bit output accommodates the carry generated by adding two 8-bit numbers.

### Simulation Results

The following waveforms show the changing input operands and corresponding 9-bit sum.

![RCA waveform](Screenshots/Screenshot%202026-11-22%20220247.png)

![RCA waveform – detailed view](Screenshots/Screenshot%202026-11-22%20221747.png)

![RCA waveform – additional view](Screenshots/Screenshot%202026-11-22%20221809.png)

### Synthesis Results

The synthesized RCA hierarchy contains eight full-adder instances.

![RCA synthesis output](Screenshots/Screenshot%202026-11-22%20230521.png)

**Note:** The RCA schematic is intentionally not included.

---

## 7. Key Learnings

- Incomplete combinational assignments can infer latches.
- Complete assignments help prevent unintended latch inference.
- Procedural loops can describe repeated combinational logic.
- A DEMUX can be implemented using either a loop or a case statement.
- Generate loops can instantiate repeated hardware structures.
- A ripple-carry adder is constructed by cascading full-adder stages.
- Yosys synthesis reports help identify the hardware inferred from RTL.

---

## Tools Used

| Tool | Purpose |
|---|---|
| Icarus Verilog | RTL simulation |
| GTKWave | Waveform analysis |
| Yosys | RTL synthesis |
| Graphviz | Synthesis schematic visualization |
| SKY130 | Standard-cell technology library |

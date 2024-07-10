## Design a Pipelined CPU

### Project Overview

A 32-bit pipelined CPU based on the given Instruction Set Architecture (ISA) was designed. The ISA details and project requirements are described below.

### ISA Specifications

- **Register file size**: 64 registers, each register has 32 bits.
- **PC**: 32 bits.
- **2’s complement**: Assumed.
- **Instruction format**: Each instruction is 32-bit wide, consisting of five fields: opcode, rd, rs, rt, and unused. The format is as follows:

  ```
  Opcode (4 bits) | rd (6 bits) | rs (6 bits) | rt (6 bits) | unused (10 bits)
  ```

### Instructions

The following table defines the 13 instructions of the ISA:

| Instruction        | Symbol | Opcode | rd | rs | rt | Function                   |
|--------------------|--------|--------|----|----|----|----------------------------|
| No operation       | NOP    | 0000   |    |    |    | No operation               |
| Save PC            | SVPC   | 1111   | rd | y  |    | xrd <- PC + y              |
| Load               | LD     | 1110   | rd | rs |    | xrd <- M[xrs]              |
| Store              | ST     | 0011   |    | rs | rt | M[xrs] <- xrt              |
| Add                | ADD    | 0100   | rd | rs | rt | xrd <- xrs + xrt           |
| Increment          | INC    | 0101   | rd | rs | y  | xrd <- xrs + y             |
| Negate             | NEG    | 0110   | rd | rs |    | xrd <- -xrs                |
| Subtract           | SUB    | 0111   | rd | rs | rt | xrd <- xrs - xrt           |
| Jump               | Jrs    | 1000   |    | rs |    | PC <- xrs                  |
| Branch if zero     | BRZrs  | 1001   |    | rs |    | PC <- xrs, if Z = 1        |
| Jump memory        | JM     | 1010   |    | rs |    | PC <- M[xrs]               |
| Branch if negative | BRNrs  | 1011   |    | rs |    | PC <- xrs, if N = 1        |
| Minimum            | MIN    | 0001   | rd | rs | rt | xrd <- min(xrs, xrt)       |

### Assembly Program

Write two versions of an assembly program to find the minimum of `n` numbers:
1. **Software Loop**: Does not use the MIN instruction.
2. **Hardware Loop**: Uses the MIN instruction.

### Assumptions

1. The input array has at least one element i.e., the input array A is non-empty.
2. The address of the size of the input array is stored in register x4.
3. The input array is stored in memory location following x4. Memory is named M.
4. When the execution of the program begins, the program counter (PC) points to the first instruction.
5. Register x0 is reserved for the constant 0.

### Assembly Code

#### Software Loop (Without MIN Instruction)

| Register | Description |
|----------|-------------|
| x1       | Current input value |
| x2       | Minimum value |
| x3       | Size of the input array, counter |
| x4       | Address of the size of the array |
| x6       | Start of the loop |
| x8       | Sum of current input value and minimum value |
| x10      | Position of array increment and counter decrement |
| x11      | Position of update minimum value |
| x12      | Position of storing the final minimum number |
| M[x32]   | Memory location where final minimum value is stored |

```assembly
1.  SVPC x40, x0
2.  INC x10, x40, 0x39
3.  INC x11, x40, 0x49
4.  INC x12, x40, 0x59
5.  LD x3, x4
6.  INC x4, x4, 0x4
7.  INC x2, x0, 0x7FFFFFFF
8.  SVPC x6, 0x4
9.  LD x1, x4
10. NEG x7, x2
11. ADD x8, x7, x1
12. BRN x11
13. INC x4, x4, 0x4
14. INC x3, x3, 0xFFFFFFFF // decrement the counter
15. BRZ x12
16. J x6
17. INC x2, x1, 0x0
18. J x10
19. ST x2, x32
```

#### Equivalent C/C++ Code

```cpp
const int n = 5;
int arr[n];
int min = INT_MAX;
for (int i = 0; i < n; i++) {
    if (arr[i] < min) {
        min = arr[i];
    }
}
return min;
```

#### Hardware Loop (With MIN Instruction)

| Register | Description |
|----------|-------------|
| x1       | Minimum value |
| x3       | Size of the input array, counter |
| x4       | Address of the size of the array |
| M[x32]   | Memory location where final minimum value is stored |

```assembly
1.  LD x3, x4
2.  INC x4, x4, 0x4
3.  MIN x1, x4, x3
4.  ST x1, x32
```

### Datapath & Control

#### Truth Table for Control Logic

| Instruction         | RegWrite | MemWrite | MemRead | ALUSrc1 | ALUSrc2 | BRN  | BRZ  | JUMP | ADD  | SUB  | NEG  | MemToReg | SavePC |
|---------------------|----------|----------|---------|---------|---------|------|------|------|------|------|------|----------|--------|
| Save PC             | 1        | 0        | 0       | 0       | 1       | 0    | 0    | 0    | 1    | 0    | 0    | 1        | 1      |
| Load                | 1        | 0        | 1       | X       | X       | 0    | 0    | 0    | 0    | 0    | 0    | 0        | 0      |
| Store               | 0        | 1        | 0       | X       | X       | 0    | 0    | 0    | 0    | 0    | 0    | X        | X      |
| Add                 | 1        | 0        | 0       | 0       | 1       | 0    | 0    | 0    | 1    | 0    | 0    | 1        | 1      |
| Increment           | 1        | 0        | 0       | 1       | 1       | 0    | 0    | 0    | 1    | 0    | 0    | 1        | 1      |
| Negate              | 1        | 0        | 0       | X       | 1       | 0    | 0    | 0    | 0    | 0    | 1    | 1        | 1      |
| Subtract            | 1        | 0        | 0       | 0       | 1       | 0    | 0    | 0    | 0    | 1    | 0    | 1        | 1      |
| Jump                | 0        | 0        | 0       | X       | X       | 0    | 0    | 1    | 0    | 0    | 0    | X        | X      |
| Branch if zero      | 0        | 0        | 0       | X       | X       | 0    | 1    | 0    | 0    | 0    | 0    | X        | X      |
| Branch if negative  | 0        | 0        | 0       | X       | X       | 1    | 0    | 0    | 0    | 0    | 0    | X        | X      |

### Emulation of RISC-V Instructions

| RISC-V               | Equivalent ISA                                   |
|----------------------|------------------------------------------------------|
| ADD x3, x1, x2       | ADD x3, x1, x2                                       |
| SUB x3, x1, x2       | SUB x3, x1

, x2                                       |
| ADDI x3, x1, imm     | INC x3, x1, imm                                      |
| LW x3, imm(x1)       | INC x1, x1, imm<br>LD x3, x1                         |
| SW x3, imm(x1)       | INC x1, x1, imm<br>ST x3, x1                         |
| AND x2, x3, x4       | SUB x2, x2, x2<br>SVPC x10, 16<br>ADD x3, x3, x4<br>BRZ x10<br>INC x2, x3, -1<br>NOP |
| OR x2, x3, x4        | SUB x2, x2, x2<br>SVPC x10, 16<br>ADD x3, x3, x4<br>BRZ x10<br>INC x2, x2, 1<br>NOP  |
| NOR x2, x3, x4       | SUB x2, x2, x2<br>INC x2, x2, 1<br>SVPC x10, 16<br>ADD x3, x3, x4<br>BRZ x10<br>INC x2, x2, -1<br>NOP |
| ANDI x2, x3, imm     | SUB x2, x2, x2<br>SVPC x10, 16<br>INC x3, x3, imm<br>BRZ x10<br>INC x2, x2, -1<br>NOP |
| BEQ x1, x2, imm      | SVPC x4, imm<br>SUB x3, x1, x2<br>BRZ x4             |
| JAL x3, imm          | SVPC x3, 1<br>SVPC x4, imm<br>J x4                  |
| JALR x3, x1, imm     | SVPC x3, 1<br>INC x3, x1, imm<br>J x4               |

### Performance Analysis

#### Instruction Count

To prevent hazards, we can employ forwarding and flushing units. Thus, it is not necessary to include the NOP instruction in the code. After examining the assembly code, we note 9 lines of code (Lines 9 - 15) in the loop and 12 lines of code outside the loop. Hence, the total instruction count will be:

```
Instruction Count = 7n + 12
```

#### Cycle Time

There are five stages in the pipeline, and we know that the delay times for the instruction memory, data memory, register file, and ALU are 2ns, 1.5ns, and 2ns, respectively. Therefore, the clock cycle time should be set at 2ns.

#### CPI

To eliminate the control hazard, a flushing unit could be used. Thus, it will consume 1 more cycle because of the BRN instruction.

```
CPI = Total number of clock cycles / Instruction count
CPI = (K + N + 1 - 1) / N
```

Where,
- K = number of stages
- N = number of instructions

Hence,

```
CPI = (5 + 7n + 12 + 1 – 1) / (7n + 12) = (7n + 17) / (7n + 12)
```

For example, if we consider an array of 5 elements, to calculate the minimum number the loop runs 5 times i.e. n = 5

```
CPI = ((7 x 5) + 17) / ((7 x 5) + 12)
CPI = 1.106
```

When ‘n’ approaches infinity, CPI becomes almost equal to 1.

#### Execution Time

Formula for calculating Execution time/CPU time is:

```
Execution Time = Instruction count x CPI x Clock Cycle Time
               = (7n + 12) x [(7n + 17) / (7n + 12)] x 2ns
```

Now, for the example which we considered earlier,

```
Execution Time = ((7 x 5) + 12) x 1.106 x 2ns = 103.96ns
```


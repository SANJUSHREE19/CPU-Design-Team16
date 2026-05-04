# CMPE 220 – Software CPU Design Project

**Course:** CMPE 220 – System Software  
**Instructor:** Prof. Ishie Eshwar  
**Semester:** Spring 2026 
**Team:** 16

---

## Overview

This project implements a simple **16-bit Software CPU** using C++. The system simulates how a basic processor works, including instruction execution, memory management, and program control flow.

The project includes:

- A CPU emulator with a fetch–decode–execute cycle  
- A custom assembler to convert assembly code into machine instructions  
- A memory subsystem with a defined memory layout and memory-mapped I/O  
- Example assembly programs demonstrating CPU functionality : Hello World and Fibonacci Sequence

  ---

## Features

### CPU Emulator
- Implements the full execution cycle (fetch, decode, execute)
- Supports arithmetic, memory, and control instructions
- Tracks processor state using registers and flags

### Assembler
- Converts assembly code into machine code
- Supports labels, immediate values, and a `.DATA` section
- Uses a two-pass approach (label resolution + encoding)

### Memory System
- 64 KB addressable memory
- Separate regions for code, data, stack, and I/O
- Memory-mapped output for character printing

---

## CPU Architecture
### Register File
- 6 general-purpose registers: R0–R5
- FP (R6) – frame pointer (not heavily used in Part 1 but available)
- SP (R7) – stack pointer (used for basic stack operations / future extensions)
- PC – program counter (16-bit)
- IR – instruction register (16-bit)
- Flags – Zero (Z), Negative (N)
All registers are 16-bit.

---

## Instruction Set

### Instruction Word Layout (16 bits)
|15 ... 12|11 ... 8|7 ... 4|3 ... 0| | OPCODE | RD | RS1 | RS2/IMM4 |

## Supported Instructions

| Mnemonic              | Opcode | Type | Description                              |
|----------------------|--------|------|------------------------------------------|
| LOADI rD, #imm8      | 0x1    | I8   | Load an 8-bit immediate value into rD    |
| LOAD rD, [rA]        | 0x2    | R/M  | Load a word from memory[rA] into rD      |
| STORE rS, [rA]       | 0x3    | R/M  | Store a word from rS into memory[rA]     |
| ADD rD, rS1, rS2     | 0x4    | R    | Add rS1 and rS2, store result in rD      |
| ADDI rD, rS, #imm4   | 0x5    | I4   | Add signed 4-bit immediate to rS → rD    |
| JUMP label           | 0x9    | J    | Jump to specified label unconditionally  |
| CMP rL, rR           | 0xA    | R    | Compare rL and rR, update Z and N flags  |
| JUMPEQ label         | 0xB    | J    | Jump if Zero flag (Z) is set             |
| HALT                 | 0xF    | —    | Terminate program execution              |

## Memory Map

The CPU uses a 16-bit address space (64 KiB) divided into code, data, stack, and MMIO regions.

Address (hex, 64 KiB total)

0xFFFF  +------------------------------------+
        |            MMIO Region             |
0xFF00  +------------------------------------+  ← IO_START (also STACK_START)
        |               STACK                |  grows downward (0xFF00 → …)
        |                 …                  |
0x1000  +------------------------------------+  ← DATA_START
        |                DATA                |  .DATA segment – strings, constants
        |                 …                  |
0x0000  +------------------------------------+  ← CODE_START
        |                CODE                |  Program instructions (~4 KiB)
        +------------------------------------+

- CODE_START = 0x0000
- DATA_START = 0x1000
- STACK_START = 0xFF00
- IO_START = 0xFF00 (memory-mapped output register)
Writing a byte/word to the MMIO address (0xFF00) prints a character (used by Hello World).

## Emulator Components

The implementation is organized inside the `src/` directory.

- `cpu_defs.h`
  - Defines CPU architecture constants such as word size, memory size, and memory addresses
  - Stores register indexes and opcode values

- `Cpu.h` / `Cpu.cpp`
  - Implements the CPU register file, PC, IR, and flags
  - Contains the main CPU functions: `fetch()`, `decode_execute()`, and `run()`
  - Handles ALU operations such as `ADD`, `ADDI`, and `CMP`

- `Memory.h` / `Memory.cpp`
  - Implements a 64 KiB memory array
  - Provides memory operations such as `read_word`, `write_word`, and `load_byte`
  - Includes `dump_memory` for debugging
  - Supports memory-mapped output through `IO_START`

- `Assembler.h` / `Assembler.cpp`
  - Implements a two-pass assembler
  - First pass resolves labels
  - Second pass encodes assembly instructions into machine code
  - Supports labels, `.DATA`, and numeric immediates such as `#5`, `0x1000`, and `#-1`
  - Loads generated machine code and data into memory

- `main.cpp`
  - Provides the command-line interface
  - Supports running assembly files using:
    ```bash
    ./emulator run <file.asm>
    ```
  - Supports stepping through instructions using:
    ```bash
    ./emulator step <file.asm>
    ```
  - Prints the final register state and memory dump after execution
 
## Build & Run

### Build

From the project root directory:

```bash
g++ -std=c++11 -I./src src/*.cpp -o emulator
```

### Run

Hello World Program:

```bash
./emulator run asm/hello_world.asm
```

Fibonacci Program:

```bash
./emulator run asm/fibonacci.asm
```
---

## How It Works

1. Assembly code is parsed and converted into machine instructions
2. Instructions are loaded into memory
3. The CPU executes instructions step-by-step:
- Fetch → Decode → Execute
4. Results are stored in registers or memory
5. Output is produced via memory-mapped I/O

## Team
- Sanjushree Golla
- Gandhi Soumya Atluri
- Vignesh Jetty Ravi

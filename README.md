# Bridge-8 — 8-bit Educational CPU Simulator

> **Bridging hardware concepts and software understanding for Computer Architecture and Digital Logic students.**

[![Java](https://img.shields.io/badge/Language-Java-orange?logo=java)](https://www.java.com)
[![Swing](https://img.shields.io/badge/GUI-Java%20Swing-blue)](https://docs.oracle.com/javase/tutorial/uiswing/)
[![ISA](https://img.shields.io/badge/ISA-v1.2-green)]()
![License](https://img.shields.io/badge/License-Research%2FEducational_Use_Only-blue)

---

## What is Bridge-8?

Bridge-8 is an **8-bit accumulator-based educational CPU simulator** written in Java Swing. It is designed for university-level teaching in Computer Architecture and Digital Logic undergraduate computer science subjects, mapping directly to concepts from Stallings' *Computer Organization and Architecture* textbook.

It bridges:
- **Hardware concepts** — register transfers, datapaths, instruction cycles, condition flags
- **Software concepts** — assembly programming, control flow, I/O handling

---

## Architecture — ISA v1.2

| Component | Description |
|---|---|
| **Accumulator (ACC)** | 8-bit single working register |
| **Program Counter (PC)** | 4-bit; addresses 16 memory cells (0–15) |
| **Instruction Register (IR)** | 8-bit; holds the current instruction |
| **Main Memory** | 16 × 8-bit unified code + data store |
| **Condition Flags** | Zero (Z) and Carry (C) |
| **Instruction Encoding** | `[opcode 4-bit \| operand 4-bit]` |

---

## Instruction Set — 15 Opcodes

| Mnemonic | Hex | Effect | Cycles |
|---|---|---|---|
| `LOAD addr` | 0 | ACC ← M[addr] | F + 3 |
| `STORE addr` | 1 | M[addr] ← ACC | F + 3 |
| `ADD addr` | 2 | ACC ← ACC + M[addr] *(sets Z, C)* | F + 4 |
| `SUB addr` | 3 | ACC ← ACC − M[addr] *(sets Z, C as borrow)* | F + 4 |
| `AND addr` | 4 | ACC ← ACC AND M[addr] | F + 4 |
| `OR addr` | 5 | ACC ← ACC OR M[addr] | F + 4 |
| `NOT` | 6 | ACC ← NOT ACC | F + 2 |
| `JMP addr` | 7 | PC ← addr *(unconditional)* | F + 1 |
| `JZ addr` | 8 | if Z=1: PC ← addr | F + 2 |
| `JC addr` | 9 | if C=1: PC ← addr | F + 2 |
| `IN` | A | ACC ← user input *(programmed I/O)* | F + 1 |
| `OUT` | B | display ACC | F + 1 |
| `LOADI addr` | C | ACC ← M[M[addr]] *(indirect)* | F + I + 2 |
| `STOREI addr` | D | M[M[addr]] ← ACC *(indirect)* | F + I + 2 |
| `HALT` | F | stop execution | F + 1 |

> **F** = 3-cycle fetch · **I** = 3-cycle indirect-resolve

---

## Features

### Educational Visualisation
- **RTL micro-op trace mode** — toggle to expand each instruction into its T1, T2, T3… micro-operations, showing actual register values at every clock tick
- **Phase labels** in the RTL trace: `─── FETCH ───`, `─── INDIRECT ───`, `─── EXECUTE ───`
- **Live register view** — ACC, PC, IR displayed simultaneously in hex, binary, and decimal; flashes green on change
- **Live memory table** — all 16 bytes visible with the PC-pointed row highlighted
- **Status badge** — colour-coded indicator: `READY` / `ASSEMBLED` / `RUNNING` / `WAITING` / `HALTED` / `ERROR`

### Programming Environment
- Two-pass assembler with labels, comments, and hexadecimal/decimal literals
- `DAT` directive for declaring data bytes
- 7 built-in example programs, including an **Indirect Addressing** demonstrator
- `ASSEMBLE` /`STEP` / `RUN` / `RESET` controls with adjustable animation speed slider for debugging machine code
- `CLEAR` control to clear the code editor, allowing users to write their own assembly code 
- `RTL` control for verbose micro-operation tracing
- Smart auto-scrolling trace with 2000-entry history buffer

### I/O Console
- **Output panel** — green-chip display for `OUT` instruction results
- **Input panel** — prompt + text field + submit button for `IN` instruction
- Programmed-I/O model (synchronous; CPU pauses during `IN`)

### Documentation & Onboarding
**Help → Tutorial** covers:
- Quick-start walkthrough
- Interface layout description
- All control buttons explained
- Full instruction set reference
- Instruction cycle phases: FETCH / INDIRECT / EXECUTE
- Programmed I/O vs Interrupt-driven I/O distinction
- Classroom tips & ideas

**Help → About** — credit dialog with author and institution

---

## Pedagogical Mapping to Stallings

Stallings, William. *"Computer Organization and Architecture : Designing for performance"* (2016) — Tenth edition.

| Stallings Concept | Bridge-8 Manifestation |
|---|---|
| Fetch–Decode–Execute cycle | T1–T3 fetch, T4+ execute (visible with RTL on) |
| Direct addressing | `LOAD`, `STORE`, `ADD`, `SUB`, `AND`, `OR` |
| Indirect addressing | `LOADI`, `STOREI` with explicit `─── INDIRECT ───` phase |
| Conditional branching | `JZ` (zero flag), `JC` (carry flag) |
| Programmed I/O | `IN`, `OUT` instructions |
| Interrupt-driven I/O | Not implemented; explicitly contrasted in Tutorial |
| Status word | Z and C flags |
| Register transfer language | Every micro-op shown in `dest ← source` notation |

---

## Intentional Simplifications

Bridge-8 deliberately omits several real-world CPU features. These are **teaching choices**, not limitations — they isolate core instruction-cycle concepts so students master the fundamentals before encountering optimisations.

| Omitted Feature | Reason |
|---|---|
| Interrupt cycle | `IN` uses programmed I/O; the Tutorial explicitly contrasts this with interrupt-driven I/O |
| DMA / memory-mapped I/O | `IN` and `OUT` use dedicated I/O bus lines, not memory addresses |
| Pipelining / caching | Every instruction completes fully before the next begins |

---

## Getting Started

### Requirements
- Java Runtime Environment (JRE) **17 or later** — [Download here](https://www.java.com/en/download/)

### Run
1. Download **Bridge8Simulator.jar** from the [Releases](../../releases) page
2. Double-click the JAR, **or** run from terminal:

```bash
java -jar Bridge8Simulator.jar
```

> **"Unable to launch" or class-version error?** Make sure your Java version is 17+:
> ```bash
> java -version
> ```

---


## Author

© 2026 Ahmad Fariz Ali

Faculty of Computing, Universiti Teknologi Malaysia (UTM)

Developed for:
- SECR1033 / SCSR1033
- Computer Architecture and Organization

Built with Java Swing



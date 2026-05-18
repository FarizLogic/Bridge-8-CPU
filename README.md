# Bridge-8 CPU

> An 8-bit accumulator-based educational CPU simulator that makes the fetch–decode–indirect–execute cycle **visible at the register-transfer level**, bridging the conceptual gap between Digital Logic and Computer Architecture for undergraduate Computer Science students.

**Author:** Ahmad Fariz Ali, Faculty of Computing, Universiti Teknologi Malaysia (UTM)
**Course alignment:** SECR1033 / SCSR1033 — Computer Architecture and Organization
**Textbook anchor:** Stallings, *Computer Organization and Architecture: Designing for Performance* (10th ed., 2016)
**Current specification:** ISA v1.2 (16 May 2026)
**License:** Research and Educational Use Only — © 2026 Ahmad Fariz Ali

---

## What Bridge-8 is

Bridge-8 is a deliberately tiny accumulator CPU implemented as a Java Swing simulator. It exists to do one thing well: **make every micro-operation of the instruction cycle visible to students, with real register values resolved at every clock tick**. The entire machine state fits on one screen, the entire address space fits in one table, and every instruction expands into a labelled sequence of register transfers the student can read directly.

The name reflects the project's purpose: **bridging hardware concepts** (register transfers, datapaths, instruction cycles, condition flags) **and software concepts** (assembly programming, control flow, programmed I/O), within a CPU small enough to be understood completely in a single lecture.

## What makes Bridge-8 distinct

Bridge-8's strongest distinguishing feature is its **execution trace**. Every instruction expands into labelled phase banners (`FETCH`, `INDIRECT`, `EXECUTE`) followed by numbered T-states in register-transfer notation, with the value of each register resolved at the end of every cycle:

```
▶ [PC=0] LOAD 8    (IR=0x08)
  ── FETCH ──
  T1   MAR ← PC           (MAR=0)
  T2   MDR ← M[MAR]       (MDR=0x08)
  T3   IR  ← MDR, PC++    (IR=0x08 PC=1)
  ── EXECUTE ──
  T4   MAR ← IR[3:0]      (MAR=8)
  T5   MDR ← M[MAR]       (MDR=0x05)
  T6   ACC ← MDR          (ACC=0x05=5)

▶ [PC=1] OUT       (IR=0xB0)
  ── FETCH ──
  T1   MAR ← PC           (MAR=1)
  T2   MDR ← M[MAR]       (MDR=0xB0)
  T3   IR  ← MDR, PC++    (IR=0xB0 PC=2)
  ── EXECUTE ──
  T4   OUTPUT ← ACC       (output=5)

▶ [PC=2] SUB 9     (IR=0x39)
  ── FETCH ──
  T1   MAR ← PC           (MAR=2)
  T2   MDR ← M[MAR]       (MDR=0x39)
  T3   IR  ← MDR, PC++    (IR=0x39 PC=3)
  ── EXECUTE ──
  T4   MAR ← IR[3:0]      (MAR=9)
  T5   MDR ← M[MAR]       (MDR=0x01=1)
  T6   ACC ← ACC - MDR    (5-1=4)
  T7   set flags          (Z=0 C/borrow=0)
```

This is the artefact that closes the gap between the textbook RTL tables students read on paper and the wire-level signal traces of digital-logic simulators. It is what the name *Bridge-8* refers to operationally.

## Architecture

### Programmer-visible state

| Component | Width | Role |
|---|---|---|
| **ACC** (Accumulator) | 8-bit | Single working register; implicit source/destination of every arithmetic and logic instruction |
| **PC** (Program Counter) | 4-bit | Addresses 16 memory cells (0–15) |
| **IR** (Instruction Register) | 8-bit | Holds the currently executing instruction |
| **Z** flag | 1-bit | Zero — set when arithmetic result is 0 |
| **C** flag | 1-bit | Carry/borrow — set by ADD overflow or SUB borrow |

### Internal datapath registers (exposed in the trace)

| Component | Width | Role |
|---|---|---|
| **MAR** (Memory Address Register) | 4-bit | Holds the address presented to memory |
| **MDR** (Memory Data Register) | 8-bit | Holds the word being transferred to/from memory |

### Memory

- **16 × 8-bit unified code+data store** (Von Neumann organisation)
- Code and data share the same address space
- Maximum program size: 16 cells total

### Instruction format

Every instruction is exactly one byte:

```
 7   6   5   4   3   2   1   0
┌───┬───┬───┬───┬───┬───┬───┬───┐
│   opcode      │   operand     │
│   (4 bits)    │   (4 bits)    │
└───────────────┴───────────────┘
```

The 4-bit operand is either a direct memory address (0–15) or ignored (for instructions that operate only on ACC or I/O).

## Instruction Set Architecture (v1.2)

The ISA contains **15 instructions**. Opcode `E` is reserved.

| Mnemonic | Opcode | Operation | Cycles | Addressing |
|---|---|---|---|---|
| `LOAD addr` | `0` | ACC ← M[addr] | F + 3 | Direct |
| `STORE addr` | `1` | M[addr] ← ACC | F + 3 | Direct |
| `ADD addr` | `2` | ACC ← ACC + M[addr]; set Z, C | F + 4 | Direct |
| `SUB addr` | `3` | ACC ← ACC − M[addr]; set Z, C(borrow) | F + 4 | Direct |
| `AND addr` | `4` | ACC ← ACC AND M[addr] | F + 4 | Direct |
| `OR addr` | `5` | ACC ← ACC OR M[addr] | F + 4 | Direct |
| `NOT` | `6` | ACC ← NOT ACC | F + 2 | Implied |
| `JMP addr` | `7` | PC ← addr | F + 1 | Direct |
| `JZ addr` | `8` | if Z=1 then PC ← addr | F + 2 | Direct |
| `JC addr` | `9` | if C=1 then PC ← addr | F + 2 | Direct |
| `IN` | `A` | ACC ← input (blocking) | F + 1 | Implied |
| `OUT` | `B` | output ← ACC | F + 1 | Implied |
| `LOADI addr` | `C` | ACC ← M[M[addr]] | F + I + 2 | Indirect |
| `STOREI addr` | `D` | M[M[addr]] ← ACC | F + I + 2 | Indirect |
| `HALT` | `F` | stop execution | F + 1 | Implied |

Where **F = 3** (fetch micro-cycles) and **I = 3** (indirect-resolution micro-cycles).

### Addressing modes

- **Direct** — operand nibble is the memory address (`LOAD`, `STORE`, `ADD`, `SUB`, `AND`, `OR`, jumps)
- **Indirect** — operand nibble points to a cell that contains the real address (`LOADI`, `STOREI`)
- **Implied** — operates on ACC or fixed I/O ports (`NOT`, `IN`, `OUT`, `HALT`)
- **Register-direct** — conditional jumps test the implicit Z or C flag

There is no immediate addressing mode. Constants are placed in memory using the assembler's `DAT` directive.

## The four-phase instruction cycle

Bridge-8 v1.2 implements the textbook four-phase cycle from Stallings:

```
   ┌──────────┐    ┌──────────┐    ┌──────────────┐    ┌──────────┐
   │  FETCH   │──> │  DECODE  │──> │   INDIRECT   │──> │ EXECUTE  │
   │ (3 cyc)  │    │          │    │  (3 cyc, if  │    │ (1-4 cyc)│
   │          │    │          │    │   LOADI/     │    │          │
   │          │    │          │    │   STOREI)    │    │          │
   └──────────┘    └──────────┘    └──────────────┘    └──────────┘
                                            │
                                            └─ skipped for direct-mode
                                               instructions
```

The `INDIRECT` phase was added in v1.2 to make indirect-addressing resolution visible as its own pedagogical step, rather than hiding it inside `EXECUTE`. This matches the four-phase cycle Stallings describes in Chapter 12.

The **interrupt cycle** is deliberately omitted (see *Design choices* below).

## Pedagogical mapping to Stallings (10th ed.)

| Stallings concept | Bridge-8 manifestation |
|---|---|
| Fetch–decode–execute cycle | Labelled phase banners + T-state micro-ops in the trace |
| Direct addressing | `LOAD`, `STORE`, `ADD`, `SUB`, `AND`, `OR` |
| Indirect addressing | `LOADI`, `STOREI` with explicit `INDIRECT` phase |
| Conditional branching | `JZ` (zero flag), `JC` (carry flag) |
| Status word | Z and C flags, updated in their own T-state |
| Register transfer language | Every micro-op shown as `dest ← source` with resolved values |
| Programmed I/O | `IN` (blocking), `OUT` |
| Interrupt-driven I/O | Not implemented — contrasted with programmed I/O in tutorial |
| Memory hierarchy | Single 16-cell unified store (no cache, no DMA) |

## Design lineage and prior art

Bridge-8 sits within the educational accumulator-CPU tradition and acknowledges its influences explicitly:

- **From SAP-1** (Malvino, 1977) — the dimensional envelope: 8-bit word, 4-bit address space, 16-cell memory, 4+4 instruction encoding. SAP-1's footprint was chosen for breadboard buildability; Bridge-8 inherits it for one-screen visualisability.
- **From MARIE** (Null & Lobur, 2003) — the indirect-addressing scheme (`LOADI`/`STOREI` with the `I` suffix), the explicit named `INDIRECT` phase, the programmed-I/O philosophy, and the simulator presentation format with labelled phases.
- **From Mano's Basic Computer** (Mano, 1976) — the MAR/MDR datapath register convention and the use of a status flag (Mano's `E`) for arithmetic conditions.

**Independent design choices** in Bridge-8 that are not inherited from any single predecessor:

- **Flag-based conditional branching** (`JZ`, `JC` testing a Z/C status register) rather than MARIE's `Skipcond` AC-comparison primitive. This convention is closer to real microprocessors (6502, 8080, Z80, x86) and improves transfer of learning to industry ISAs.
- **Full bitwise logic family** (`AND`, `OR`, `NOT`) absent from both MARIE and SAP-1, enabling bitmask exercises that connect Digital Logic gate concepts to assembly programming.
- **Per-cycle resolved-value trace format** showing every register's value at the end of every T-state, more granular than MarieSim's typical step-mode presentation.

Bridge-8 is not a clone of MARIE, SAP-1, or any other prior CPU — instruction encodings, opcodes, word size, and conditional-branching semantics all differ. It is best described as a MARIE-inspired educational CPU in the SAP-1 size class with original synthesis choices around flags, bitwise logic, and trace presentation.

## Design choices — what is deliberately omitted

Bridge-8's omissions are **teaching choices, not limitations**. They isolate core instruction-cycle concepts so students master the fundamentals before encountering optimisations.

- **No interrupt cycle.** `IN` blocks the CPU (synchronous programmed I/O), allowing the tutorial to contrast programmed and interrupt-driven I/O pedagogically.
- **No pipelining, no caching.** Every instruction completes fully before the next begins, preserving the serial fetch-decode-execute mental model.
- **No DMA, no memory-mapped I/O.** `IN`/`OUT` use dedicated I/O bus lines, so I/O does not consume the 16-cell memory.
- **No subroutines, no stack.** Calling conventions and stack discipline are out of scope at this level.
- **No general-purpose register file.** A single accumulator preserves the single-source/single-destination clarity of every operation.
- **No immediate addressing.** Constants live in memory via the `DAT` directive, reinforcing the memory-reference discipline.

## Simulator features

- **Two-pass assembler** with labels, comments, hex (`0x..`) and decimal literals, and the `DAT` directive for data constants
- **Seven built-in example programs**, including an Indirect Addressing demonstrator
- **Step, Run, Reset, Clear, Assemble** controls
- **Adjustable animation speed slider** for live demonstration
- **Live register display** for ACC, PC, IR (and MAR, MDR in the trace) in hex, binary, and decimal simultaneously, flashing green on change
- **Live 16-byte memory table** with the PC-pointed row highlighted
- **Status badge** cycling through `READY` / `ASSEMBLED` / `RUNNING` / `WAITING` / `HALTED` / `ERROR`
- **Execution trace** with phase banners, T-state micro-ops, and resolved register values (auto-scrolling, up to 2000 entries)
- **Programmed I/O panel** — green-chip output display and prompt-based input field
- **Built-in tutorial** covering quick-start, interface, full ISA reference, cycle phases, and the programmed-vs-interrupt I/O contrast

## Getting started

### Requirements

- Java Runtime Environment (JRE) 17 or later

### Running

Download `Bridge8Simulator.jar` from the [Releases](https://github.com/FarizLogic/Bridge-8-CPU/releases) page, then:

```bash
java -jar Bridge8Simulator.jar
```

### A first program

This program loads the value 5, outputs it, subtracts 1, and outputs the result:

```
LOAD  8        ; ACC ← M[8] = 5
OUT            ; output ACC
SUB   9        ; ACC ← ACC − M[9] = 5 − 1 = 4
OUT            ; output ACC
HALT
DAT   5        ; cell 8: the value 5
DAT   1        ; cell 9: the value 1
```

Assemble, then step through with the **Step** button while watching the trace expand each instruction into its T-state micro-operations.

### Trying indirect addressing

```
LOADI 8        ; ACC ← M[M[8]] = M[10] = 7
OUT
HALT
DAT   10       ; cell 8: pointer to cell 10
DAT   0
DAT   7        ; cell 10: the actual value
```

The trace will show the `INDIRECT` phase explicitly between `FETCH` and `EXECUTE`.

## Educational positioning

Bridge-8 is intended as a **first-exposure CPU** for students simultaneously learning Digital Logic and Computer Architecture. Its 16-cell memory and single-screen visualisation are advantages at this stage: students see the whole machine, write programs that exercise the whole machine, and follow every micro-operation. Students who outgrow Bridge-8 within a semester are expected to graduate to a larger educational CPU (such as MARIE) or to a real ISA (such as x86 with Irvine library, or 6502).

The simulator is designed to be used alongside a textbook (Stallings) and complements — rather than replaces — hands-on digital-logic experiments with gate-level chips (7400-series, breadboards) and assembly-language programming on real or near-real ISAs.

## Citation

If you use Bridge-8 in teaching or research, please cite:

> Ahmad Fariz Ali, *Bridge-8 CPU: An 8-bit Accumulator Educational CPU Simulator*, Faculty of Computing, Universiti Teknologi Malaysia, 2026. Available at: https://github.com/FarizLogic/Bridge-8-CPU

## License

Copyright © 2026 Ahmad Fariz Ali. **Research and Educational Use Only.** See [LICENSE](LICENSE) for full terms.

## Acknowledgements

Bridge-8 draws conceptual inspiration from the educational CPU tradition established by Albert Paul Malvino (SAP-1), Linda Null and Julia Lobur (MARIE), and M. Morris Mano (Basic Computer). The textbook anchor is William Stallings, *Computer Organization and Architecture: Designing for Performance*, 10th edition (2016).

---

*Bridge-8 — making the instruction cycle visible, one micro-operation at a time.*

---
layout: post
published: true
date: 2026-01-22 00:00:00 -0500
title: 6502 Assembly on the C64 with VICE
tags: ["6502", "Commodore 64"]
---

One of the most effective ways to learn 6502 assembly is to work directly at the machine level: assembling instructions by hand, stepping through execution, and inspecting registers and memory as the CPU runs. The **VICE Monitor** provides an excellent interactive environment for doing exactly that on the Commodore 64.

This post is a practical guide to the subset of monitor commands you will actually use when practicing 6502 programming on the C64. It deliberately ignores advanced or peripheral features and concentrates on assembling code, disassembling it, stepping through instructions, inspecting state, and managing breakpoints.

## Top 10 Command Cheatsheet

```txt
a      assemble
d      disassemble
r      registers
z      step into
n      step over
g      run
m      memory dump
>      write bytes
break  breakpoint
watch  watch memory
```

## Entering the Monitor

From within VICE, open the monitor with:

```txt
Alt + H
```

(or from the menu: *Debug → Monitor*).

You will see a prompt similar to:

```txt
(C:$0801) >
```

This tells you:

* You are in the **computer CPU address space** (`C:`)
* The current “dot” address is `$0801` (used as a default by many commands)

For nearly all C64 assembly practice, you will stay in the `C:` address space.

## Memory Layout for the Commodore 64

A few practical conventions for learning:

* `$0801`–`$9FFF` is generally safe RAM (BASIC program area upward)
* `$C000`–`$CFFF` is commonly used as scratch RAM for machine-code
* `$D000`–`$DFFF` is I/O (VIC-II, SID, CIA)
* `$E000`–`$FFFF` is KERNAL ROM (read-only unless banked out)

For practice, `$C000` is a good default code location.

## Disassembling Code

### Basic Disassembly

To disassemble memory into 6502 instructions:

```txt
d $C000
```

This disassembles forward from `$C000`.

To disassemble a range:

```txt
d $C000 $C050
```

If you omit arguments:

```txt
d
```

the monitor disassembles starting from the current “dot” address.

Disassembly respects labels if you define them, which becomes very useful later.

## Assembling Instructions

### Interactive Assembly Mode

To assemble code starting at a given address:

```txt
a $C000
```

You can now type one instruction per line:

```txt
LDA #$06
STA $D020
RTS
```

Press **Enter on a blank line** to exit assembly mode.

The monitor automatically advances the address as instructions are entered.

## Viewing and Modifying Registers

### Display Registers

To display the CPU registers:

```txt
r
```

You will see output similar to:

```txt
A:00 X:00 Y:00 SP:FF PC:C000 FL:24
```

### Modifying Registers

You can directly set registers:

```txt
r PC=$C000
```

Or multiple registers at once:

```txt
r A=$10, X=$00, Y=$00
```

The status register is referred to as `FL`.

## Stepping Through Code

### Step Into (Execute One Instruction)

```txt
z
```

Each `z` executes exactly one instruction and then stops.

To step multiple instructions:

```txt
z 10
```

### Step Over Subroutines

If you encounter a `JSR` and want to execute it as a single step:

```txt
n
```

This runs the subroutine internally and stops at the instruction after the `JSR`.

### Step Out of a Subroutine

If you are already inside a subroutine:

```txt
ret
```

Execution continues until the matching `RTS` or `RTI`.

## Running and Continuing Execution

To resume execution from the current PC:

```txt
g
```

To set the PC and immediately run:

```txt
g $C000
```

This is often used after assembling new code.

## Inspecting Memory

### Hex Dump

To view memory as hexadecimal bytes:

```txt
m $C000
```

To view a range:

```txt
m $C000 $C0FF
```

### Viewing Text

To view memory as PETSCII characters:

```txt
i $0400 $07E7
```

To view screen codes:

```txt
ii $0400 $07E7
```

These are helpful when working with screen memory or character data.

## Writing Bytes Directly to Memory

You can write raw bytes using the `>` command:

```txt
> $C000 A9 06 8D 20 D0 60
```

This is equivalent to assembling:

```txt
LDA #$06
STA $D020
RTS
```

If you omit the address, bytes are written at the current “dot” address.

## Breakpoints and Watchpoints

VICE uses a unified concept called **checkpoints**, which includes breakpoints, watchpoints, and tracepoints.

### Execution Breakpoint

Stop when code at an address executes:

```txt
break exec $C000
```

Start execution:

```txt
g
```

Execution halts when the PC reaches `$C000`.

### Temporary Run-Until Break

To run until a specific address *once*:

```txt
un $C050
```

The temporary breakpoint deletes itself after being hit.

### Watch Memory Writes

Break when memory is written to:

```txt
watch store $D020
```

This is extremely useful when learning how code affects VIC-II registers.

### Managing Breakpoints

List checkpoints:

```txt
break
```

Delete one:

```txt
del 2
```

Disable / enable:

```txt
disable 2
enable 2
```

## Conditional Breakpoints

You can attach conditions to breakpoints:

```txt
cond 2 if A==$00
```

Conditions can reference:

* Registers (`A`, `X`, `Y`, `PC`, `SP`, `FL`)
* Raster line (`RL`)
* Cycle count (`CY`)
* Memory via banked dereferencing

This allows very precise debugging even in tight loops.

## Using Labels

Labels make disassembly and debugging far more readable.

Add a label:

```txt
al $C000 .main
```

Show labels:

```txt
shl
```

Delete a label:

```txt
dl .main
```

Clear all labels:

```txt
cl
```

You can also load labels generated by assemblers such as ACME or cc65 later, but even manually added labels greatly improve clarity.

## A Simple Practice Workflow

1. Assemble code at `$C000`
2. Disassemble to verify correctness
3. Set the PC manually
4. Step through instructions with `z`
5. Inspect registers and memory after each step
6. Add breakpoints or watchpoints as needed
7. Iterate

## Closing Thoughts

The VICE Monitor is not just a debugger; it is an interactive lab for learning the 6502 at the instruction-level. By assembling directly into memory, stepping instruction by instruction, and watching registers and I/O change in real time, you gain an intuition that is hard to replicate with higher-level tooling.

Once this workflow becomes second nature, integrating an external assembler feels like an optimization—not a dependency.

---
layout: post
published: true
date: 2023-05-20 00:00:00 -0500
title: Learning Z80 Assembly on the RC2014
tags: [Z80, Assembly, RC2014, "CP/M"]
---
I am reading [Programming the Z80](http://www.z80.info/zip/zaks_book.pdf) by
Rodnay Zaks to refresh my knowledge of Z80 assembly. I usually develop and
compile the code on my laptop, but this time I wanted to experience writing and
testing the code on my RC2014 computer running CP/M.

The assembler that ships with CP/M uses 8080 instructions, so instead I am using
[Z80ASM](http://www.s100computers.com/My%20System%20Pages/IBM%20Keyboard/z80asm%20(SLR%20Systems).pdf) by SLR Systems. I edit the files using [ZDE 1.6](https://sites.google.com/site/vdeeditor/Home/vde-files/zde-1-0-manual).

The book provides sample code like this for 8-bit addition,

```asm
LD  A,(ADR1)   ; LOAD OP1 INTO A
LD  HL,ADR2    ; LOAD ADDRESS OF OP2 INTO HL
ADD A,(HL)     ; ADD OP2 TO OP1
LD  (ADR3),A   ; SAVE THE RESULT RES AT ADR3
```

The problem with this code is that it isn't complete and does not output to the
console. I like to step through the code and inspect the registers in each step.
In order to do so, I rewrite the code as follows on the RC2014 including labels for the memory locations.

```asm
        LD A,(OP1)     ; LOAD OP1 INTO A
        LD  HL,OP2     ; LOAD ADDRESS OF OP2 INTO HL
        ADD A,(HL)     ; ADD OP2 TO OP1
        LD  (RES),A    ; SAVE THE RESULT RES AT ADR3
        RET
OP1:    DB 32
OP2:    DB 10
RES:    DB 0
```

I then compile the program in `8BITADD.Z80`. The assembler expects the `Z80`
file extension and in this case my code is on drive `E:` and the assembler is on
drive `I:`. I use the `/F` option so that I get a program listing to help me
understand the memory layout.

```txt
E>I:Z80ASM 8BITADD/F

Z80 ASSEMBLER Copyright (C) 1983 by SLR Systems Rel. 1.30 #F10268

 8BITADD/F
End of file Pass 1
End of file Pass 2
 0 Error(s) Detected.
 14 Absolute Bytes. 3 Symbols Detected.
```

The program listing that is produced contains,

```txt
Z80ASM SuperFast Relocating Macro Assembler                 Z80ASM 1.30 Page   1
8BITADD Z80

    1 0100  3A 010B             LD A,(OP1)     ; LOAD OP1 INTO A
    2 0103  21 010C             LD  HL,OP2     ; LOAD ADDRESS OF OP2 INTO HL
    3 0106  86                  ADD A,(HL)     ; ADD OP2 TO OP1
    4 0107  32 010D             LD  (RES),A    ; SAVE THE RESULT RES AT ADR3
    5 010A  C9                  RET
    6 010B  20          OP1:    DB 32
    7 010C  0A          OP2:    DB 10
    8 010D  00          RES:    DB 0
 0 Error(s) Detected.
 14 Absolute Bytes. 3 Symbols Detected.
```

I can run this and it works, but it doesn't print anything to the console and I
can't see what it is doing. In order to do that, on CP/M, you can use the program
[DDT](http://www.gaby.de/cpm/manuals/archive/cpm22htm/ch4.htm), the Dynamic
Debugging Tool. The following is an example session. (Note that I am running
Z-System, a Z80 CP/M clone, so I am using the command `DDTZ` but it operates the
same.)

```txt
E>A:DDTZ 8BITADD.COM
DDTZ v2.7M by CB Falconer. CPU=Z80
Next  PC  Save
0180 0100 1
```

This loads `8BITADD.COM` into memory. Notice that the program counter PC is `0x0100`
which is the start of our program on line 1 of the listing. We can see our program
in memory with the `D` command which will print 16 lines of memory starting at
`0x0100`. You can also give it ranges of memory. This program is 14 bytes, so we
can just look at the first line.

```txt
-D0100,010F
0100 3A 0B 01 21 0C 01 86 32  0D 01 C9 20 0A 00 00 00 :..!...2... ....
```

Notice that the bytes on the first line match the bytes from our listing.

To step through the program one instruction at a time, use the command `T`.

```txt
-T
C0Z0M0E0I0 A=00 B=0000 D=0000 H=0000>03C3 S=D7B5>CC10 P=0100 LDA  010B*0103
C0Z0M0E0I0 A=20 B=0000 D=0000 H=0000>03C3 S=D7B5>CC10 P=0103 LXI  H,010C
```

Notice that the `A` register changed to `0x20` (hex for 32) and the program counter
is now `0x0103` which is our second instruction. At the end of each line, we also
see the instruction at the program counter disassembled in 8080 format.

Stepping again,

```txt
-T
C0Z0M0E0I0 A=20 B=0000 D=0000 H=0000>03C3 S=D7B5>CC10 P=0103 LXI  H,010C*0106
C0Z0M0E0I0 A=20 B=0000 D=0000 H=010C>000A S=D7B5>CC10 P=0106 ADD  M
```

The `H` register now contains `010C` which is the memory address of `OP2`. You
can also see that after the `>`, the memory that the `HL` register is pointing
to `0x000A`. It shows two bytes at that location in little endian order.

```txt
-T
C0Z0M0E0I0 A=20 B=0000 D=0000 H=010C>000A S=D7B5>CC10 P=0106 ADD  M*0107
C0Z0M0E0I0 A=2A B=0000 D=0000 H=010C>000A S=D7B5>CC10 P=0107 STA  010D
```

This executed `ADD A,(HL)` which is add to `A` what is stored at the address
pointed to by `HL`. `HL` is `0x010C` and that memory location contains `0x0A` so
we expect the `A` register to switch from `0x20` to `0x2A` which it did.

Next we store `A` to memory,

```txt
-T
C0Z0M0E0I0 A=2A B=0000 D=0000 H=010C>000A S=D7B5>CC10 P=0107 STA  010D*010A
C0Z0M0E0I0 A=2A B=0000 D=0000 H=010C>2A0A S=D7B5>CC10 P=010A RET
```

Notice that the high byte of memory where `HL` is pointing is now `0x2A` which is
what we stored. We can also confirm the result by dumping memory once again.

```txt
-D0100,010F
0100 3A 0B 01 21 0C 01 86 32  0D 01 C9 20 0A 2A 00 00 :..!...2... .*..
```

The third last byte now contains the result of the addition.

You may have noticed the `C0Z0M0E0I0` at the start of each line. These are the
flags. Addition didn't overflow in our program, so the flags didn't change. To
see them change, let's update the assembly so that the result of the addition
is greater than 255.

```asm
        LD A,(OP1)     ; LOAD OP1 INTO A
        LD  HL,OP2     ; LOAD ADDRESS OF OP2 INTO HL
        ADD A,(HL)     ; ADD OP2 TO OP1
        LD  (RES),A    ; SAVE THE RESULT RES AT ADR3
        RET
OP1:    DB 222
OP2:    DB 81
RES:    DB 0
```

This is the debugging session up to the `ADD`.

```txt
E>A:DDTZ 8BITADD.COM
DDTZ v2.7M by CB Falconer. CPU=Z80
Next  PC  Save
0180 0100 1
-D0100,010F
0100 3A 0B 01 21 0C 01 86 32  0D 01 C9 DE 51 00 00 00 :..!...2....Q...
-t
C0Z0M0E0I0 A=00 B=0000 D=0000 H=0000>03C3 S=D7B5>CC10 P=0100 LDA  010B*0103
C0Z0M0E0I0 A=DE B=0000 D=0000 H=0000>03C3 S=D7B5>CC10 P=0103 LXI  H,010C
-T
C0Z0M0E0I0 A=DE B=0000 D=0000 H=0000>03C3 S=D7B5>CC10 P=0103 LXI  H,010C*0106
C0Z0M0E0I0 A=DE B=0000 D=0000 H=010C>0051 S=D7B5>CC10 P=0106 ADD  M
-T
C0Z0M0E0I0 A=DE B=0000 D=0000 H=010C>0051 S=D7B5>CC10 P=0106 ADD  M*0107
C1Z0M0E0I0 A=2F B=0000 D=0000 H=010C>0051 S=D7B5>CC10 P=0107 STA  010D
-
```

`0xDE` plus `0x51` is `0x012F` which has overflowed, so the result is the low
byte `0x2F` in the `A` register. Also notice that the flags changed from
`C0Z0M0E0I0` to `C1Z0M0E0I0`, the Carry flag switched from `0` to `1` as
expected.

Continuing the run, we see that `0x2F` is then stored in memory.

```txt
-T
C1Z0M0E0I0 A=2F B=0000 D=0000 H=010C>0051 S=D7B5>CC10 P=0107 STA  010D*010A
C1Z0M0E0I0 A=2F B=0000 D=0000 H=010C>2F51 S=D7B5>CC10 P=010A RET
-D0100,010F
0100 3A 0B 01 21 0C 01 86 32  0D 01 C9 DE 51 2F 00 00 :..!...2....Q/..
```
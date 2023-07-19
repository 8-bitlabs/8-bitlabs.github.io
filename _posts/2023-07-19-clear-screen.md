---
layout: post
published: true
date: 2023-07-19 00:00:00 -0500
title: Clear Screen Utility for CP/M
tags: [Z80, RC2014, RomWBW, Programming, "CP/M", Assembly]
---
Occasionally after playing a game or running a program that uses color in CP/M,
my terminal isn't reset and isn't using my default colors. Even worse is when it
leaves the terminal with dark text on a black background!

CP/M has the `CLS` command, but it does not reset the default colors, so I
decided to write a quick assembly program to reset the terminal using
[ANSI Escape Sequences](https://gist.github.com/fnky/458719343aabd01cfb17a3a4f7296797).

```asm
; Clear the screen and reset colors
  org 100H                  ; CPM Program start address

; =============================================================================
; Definitions and methods for CP/M development
bdos        equ $5          ; CP/M BDOS vector address
c_writestr  equ $09         ; output string, '$' terminated, to console output
esc         equ $1B

main:
  ld de, cls
  ld c, c_writestr
  call bdos
  ret

; =============================================================================
; Datamake
cls:        db esc, "[0m"   ; reset all modes (styles and colors)
            db esc, "[2J"   ; erase entire screen
            db esc, "[H"    ; moves cursor to home position (0, 0)
            db "$"          ; character that terminates c_writestr
```

The code is simple, it uses the `BDOS` function `c_writestr` which writes
a `$` terminated string pointed to by the `DE` register to the console. It
resets all styles and colors, erases the screen using the default colors then
moves the cursor to the top left.

I've tested the code on my RC2014 using a VT100 terminal (Tera Term). You can
download the source from GitHub,
[clear.asm](https://github.com/rprouse/RC2014/blob/main/asm/cpm/clear.asm)
or the compiled binary from this website, [CLEAR.COM](/assets/files/clear.com).

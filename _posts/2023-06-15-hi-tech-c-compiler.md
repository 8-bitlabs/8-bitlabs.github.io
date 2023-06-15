---
layout: post
published: true
date: 2023-06-15 00:00:00 -0500
title: Using the HI-TECH C II Compiler in CP/M
tags: [Z80, RC2014, RomWBW, Programming, "CP/M", C]
---
The HI-TECH C is a nearly ANSI C compliant compiler that runs on Z80 based CP/M
machines. Version 3.09 was last released in March 1989 but the company has since
made it freely available for both commercial and non-commercial use. Tony
Nicholson has picked up development of the compiler and has released 17 patched
versions of HI-TECH C on GitHub at [agn453/HI-TECH-Z0-C](https://github.com/agn453/HI-TECH-Z80-C).

You can download the latest binary distrubtion from
[htc-bin.lbr](https://raw.githubusercontent.com/agn453/HI-TECH-Z80-C/master/htc-bin.lbr).
A Z280 version is also available,
[z280bin.lbr](https://raw.githubusercontent.com/agn453/HI-TECH-Z80-C/master/z280bin.lbr).
Each of these are uncompressed LBR files, so you should be able to copy them to
a disk and extract them using `DELBR.COM`. If you are using
[RomWBW](https://github.com/wwarthen/RomWBW), I have submitted a [pull request to
include it as a disk](https://github.com/wwarthen/RomWBW/pull/348) so it will be
available in the next release or now in the `dev` branch.

## Setup
By default, HI-TECH expects it's files to be in drive `A` in user area `0`. If
like me drive `A` is your CP/M drive then you will probably see the error
`Can't execute $EXEC` you attempt to compile a C file.

To fix this, HI-TECH C uses environment variables to set the default drive to
search for it's files. You set the variables in a file `ENVIRON` on the `A`
drive. I mount HI-TECH on the `I` drive in user area `0`, so I add the following
to the file,

```txt
HITECH=0:I:
```

You can also set a `TMP` variable for the drive to use for all the temporary
files used during compilation. I am running RomWBW which has a fast RAM drive,
so I set the variable to that drive,

```txt
TMP=B:
```

If you downloaded the `LBR` binaries, you will want to rename `C309-17.COM` to
`C.COM`.

## Hello World

Let's write and compile Hello World to understand how it works. First create
`hello.c` in your prefered editor, ZDE for example,

```c
main()
{
     printf("Hello world\n");
}
```

This is pretty much as expected except that you do not need the
`#include <stdio.h>` for `printf` and the return type for `main` is also not
required.

You can then compile with,

```cmd
I:C -V HELLO.C
```

`-V` is for verbose output and displays the following,

```txt
E>I:C -V HELLO.C
Hi-Tech Z80 C Compiler (CP/M-80) V3.09-17
Copyright (C) 1984-87 HI-TECH SOFTWARE
Updated from https://github.com/agn453/HI-TECH-Z80-C
0:I:CPP -DCPM -DHI_TECH_C -Dz80 -I0:I: HELLO.C B$CTMP1.$$$
0:I:P1 B$CTMP1.$$$ B$CTMP2.$$$ B$CTMP3.$$$
0:I:CGEN B$CTMP2.$$$ B$CTMP1.$$$
0:I:ZAS -N -OHELLO.OBJ B$CTMP1.$$$
ERA B$CTMP1.$$$
ERA B$CTMP2.$$$
ERA B$CTMP3.$$$
ERA B$CTMP5.$$$
0:I:LINQ -Z -Ptext=0,data,bss -C100H -OHELLO.COM 0:I:CRTCPM.OBJ HELLO.OBJ 0:I:LIBC.LIB
ERA HELLO.OBJ
ERA B$$EXEC.$$$

E>HELLO
Hello world
```

This created `HELLO.COM` in the same drive. Running it outputed the expected
`Hello world`.

If I look at the program, it is a massive (for CP/M) 16K. If you don't need
wildcard argument support and I/O redirection using pipes, you can use the `-N`
option to link with the smaller `NRTCPM.OBJ` file instead of `CRTCPM.OBJ`. Doing
so results in a 4K file.

You can also run with the `-Mfile.map` option to generate a linker map showing
the library modules included and their sizes.


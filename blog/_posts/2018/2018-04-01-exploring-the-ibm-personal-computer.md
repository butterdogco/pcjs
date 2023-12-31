---
layout: post
title: Exploring the IBM Personal Computer
date: 2018-04-01 10:00:00
permalink: /blog/2018/04/01/
---

A rather ~~ghastly~~ quaint piece of early IBM PC software was a program called
[Exploring The IBM Personal Computer](/software/pcx86/demo/ibm/exploring).  Created by Digital Learning Systems, Inc.
and marketed by IBM, it was designed to walk you through the "ins and outs" of your shiny new IBM PC, and it came in
two flavors: Monochrome and Color.

![Exploring the IBM PC (Intro](/blog/images/exploring-the-ibm-pc-intro.jpg)

After playing some truly awful "music" through the PC's speaker -- for which I must also apologize, because the limited
PC speaker support in PCjs manages to make it sound even worse -- the software begins with a tour of your IBM PC's
keyboard.

In the screenshot below, notice the painstaking detail with which even the coiled cord of the PC's keyboard is lovingly
rendered.

![Exploring the IBM PC (Keyboard)](/blog/images/exploring-the-ibm-pc-keyboard.gif)

Amazingly, this software was even updated for the IBM PC AT -- not once but twice!  On the bright side though,
the second version did eliminate the "music".

Unfortunately, when I first ran this program on PCjs, the introductory screen wasn't fully erased, leaving most of
the block characters around the border of the screen.

![Exploring the IBM PC (Bug)](/blog/images/exploring-the-ibm-pc-intro-bug.png)

This was due to a bug in their call to the ROM's INT 10h "Scroll Up" function:

    AX=0600 BX=0707 CX=184F DX=0000 SP=0100 BP=00FA SI=1E39 DI=01F6 
    SS=0FDF DS=0439 ES=0439 PS=F006 V0 D0 I1 T0 S0 Z0 A0 P1 C0 
    &02C1:06A4 CD10             INT      10

which *should* have passed the coordinates of the top left corner of the scroll operation in CX and the bottom right
coordinates in DX.  However, as you can see above, they passed the top values in DX and the bottom values in CX, so when
the ROM subtracts CX from DX to calculate the number of rows and columns to scroll (or in this case, to clear), it ends up
with negative values, which it then treats as large positive numbers instead, and proceeds to erase a large swath of memory
past the bottom of the screen.

You can debug this yourself using the [PCjs Debugger](/software/pcx86/demo/ibm/exploring/1.00-MDA/?debugger=true)
and setting a breakpoint on the above instruction (`BP 2C1:6A4`).  Here are the relevant instructions leading up to that point:

    &02C1:068C 58               POP      AX
    &02C1:068D 8AE0             MOV      AH,AL
    &02C1:068F 80C406           ADD      AH,06
    &02C1:0692 59               POP      CX
    &02C1:0693 8AC1             MOV      AL,CL
    &02C1:0695 5B               POP      BX
    &02C1:0696 8ACB             MOV      CL,BL      ; should be MOV DL,BL (8A D3)
    &02C1:0698 5B               POP      BX
    &02C1:0699 8AEB             MOV      CH,BL      ; should be MOV DH,BL (8A F3)
    &02C1:069B 5B               POP      BX
    &02C1:069C 8AD3             MOV      DL,BL      ; should be MOV CL,BL (8A CB)
    &02C1:069E 5B               POP      BX
    &02C1:069F 8AF3             MOV      DH,BL      ; should be MOV CH,BL (8A EB)
    &02C1:06A1 5B               POP      BX
    &02C1:06A2 8AFB             MOV      BH,BL
    &02C1:06A4 CD10             INT      10

So why, on a *real* PC, was the screen still being successfully erased?  Because the monochrome video card's 4K buffer
can be addressed not only at B000:0000, but also at B000:1000, B000:2000, and so on, all the way up to B000:7000.

This was an easy fix for the PCjs Video component, because the Bus component already supports aliasing blocks of memory
to other addresses.  Unfortunately, this required the Bus default block size to be no larger than 4K, and for machines
with 24-bit or 32-bit buses, the Bus component preferred to divide the address space up into 16K or 32K blocks.

So I changed the block size to 4K for *all* machines, irrespective of bus width.  Unfortunately, machines with saved states
could still attempt to load states with an older (larger) block size, so I also had to add code to "fix up" blocks whenever
a non-4K block was detected.

The good news is that, after dealing with all these ripple effects, [Exploring The IBM Personal Computer](/software/pcx86/demo/ibm/exploring/1.00-MDA/)
now runs as intended.  Well, except for the awful sound it makes, which I'll be looking into at a later date.

For now, turn your volume down (and appreciate the fact that you *can* turn your volume down, because the speaker on
an IBM PC had no such feature), and enjoy exploring the IBM Personal Computer.

# PIC32MX Blink

Introductory project for [Microstick II][PIC Microstick II] 
with [PIC32MX250F128B SPDIP][PIC32MX250F128B] (included with Microstick II in my
case - not guaranteed - Microchip may change included chips).

It does this:
- blinks on-board LED D6 (ON RA0, PIN2) using TMR1 interrupt. Interrupt rate is 10 Hz,
  LED blinking rate is 10/2 = 5 Hz (in my case measured rate was 5.0165 Hz)
- toggles RA1, PIN3 every 1ms (1000 Hz) using Core Timer interrupt. So output frequency
  on RA1 should be 1000/2 = 500 Hz (measured 501.61 Hz)
  - Core Timer ticks every 2nd CPU cycle. System clock is 48 MHz, so Core Timer increments at
    24 MHz. Interrupt is called when SYSTICK=24000 (see
    `CORE_TIMER_INTERRUPT_PERIOD_VALUE=0x5dc0 (24000 decimal)`
    in `firmware\src\config\default\peripheral\coretimer\plib_coretimer.h`)
  - look in `firmware\src\config\default\peripheral\coretimer\plib_coretimer.c` to 
    understand implementation
  - NOTE: ARM has similar register called SYSTICK.

If you never used PIC32MX and/or Harmony code generator
please look at this nice guide:
- https://ww1.microchip.com/downloads/en/DeviceDoc/Creating_Hello_World_Applicatio_%20on_PIC32_Microcontrollers_Using_MPLAB_Harmonyv3_MHC_DS90003259A.pdf
However it is for different PIC32 CPU and also software versions are outdated.

# Requirements

## Hardware requirements

* [Microstick II][PIC Microstick II]  demo board
* [PIC32MX250F128B SPDIP][PIC32MX250F128B] inserted into socket U5
  (included with board, should be default)
* programming pins set `1` (Pin4 & Pin5): switch `S1` in position `A`

NOTE: PIC32MX is `MIPS32 M4K` based CPU Core with peripherals from Microchip. Please see
- datasheet: [PIC32MX250F128B][PIC32MX250F128B]
- official splash page: https://www.microchip.com/en-us/products/microcontrollers-and-microprocessors/32-bit-mcus/pic32-32-bit-mcus/pic32mx
- [MIPS32 M4K Manual][MIPS32 M4K Manual] from mips.com
- [MIPS32 M4K Core Datasheet][MIPS32 M4K DTS] from mips.com
- [MIPS32 Instruction Set][MIPS32 BIS] from mips.com
- [MIPS32 Instruction Set Quick Reference][MIPS32 QRC] from mips.com
- it is quite similar to ARM (you can even buy ARM from Microchip - it is SAM series coming from Atmel
  that was acquired by Microchip).
- MIP32 has 32-registers, they are called either `r0-r31` or using preferred names.
  For example `r0` has always 0 value and is thus also called `zero` in assembler.
- please see [MIPS32 Instruction Set Quick Reference][MIPS32 QRC]  for quick overview.

Generally we can say that MIPS32 Assembler has similar attributes as ARM:
- designed for compilers, not for manual coding
- all instructions (at least those generated with XC32) are 32-bit wide (unless you use
  MIPS16e instructions set - possible but not default)
- this causes some issues - because CPU is 32-bit, so loading 32-bit literal
  often means that there must be used 2 instructions (1st one loads 16-bit high value, and
  2nd OR it with low 16-bit value)
- assembler provides macro-instructions that accepts 32-bit literals and generate appropriate
  2 instructions.
- also literals are internally signed, so there is trickery when assembler load full 32-bit 
- please note that there is unfortunately different meaning for Word and Double-word sizes (because
  dsPIC is 16-bit but MIPS32 is 32-bit):
  - 16-bit value: dsPIC WORD = MIP32 half-word (`H` suffix)
  - 32-bit value: dsPIC DWORD (D) = MIPS32 word (default suffix)
  - at least Byte is same :-) (using `B` suffix on MIPS)

WARNING! Generated code for MIPS32 interrupts in "Software mode" may be quite inefficient when
compared to these on dsPIC! Here is example of `void __ISR(_TIMER_1_VECTOR, ipl1SOFT) TIMER_1_Handler (void)`
(using `xc32-objdump -S output.elf` command):

This example is using so called `software instructions` mode (`ipl-x-SOFT` keyword):
- please see `DS50002799C - 188` of `c:\Program Files\Microchip\xc32\v4.30\docs\MPLAB-XC32-Compiler-UG-PIC32M-DS-50002799.pdf`

```
Disassembly of section .text.TIMER_1_Handler:

9d000120 <TIMER_1_Handler>:

void __ISR(_TIMER_1_VECTOR, ipl1SOFT) TIMER_1_Handler (void)
{
; copy SP from previous register set (if used)
9d000120:	415de800 	rdpgpr	sp,sp
; move from Coprocessor C0: k0 := EPC
9d000124:	401a7000 	mfc0	k0,c0_epc
9d000128:	401b6000 	mfc0	k1,c0_status
; add immediate "unchecked": sp -= 120 ; no exception on overflow
9d00012c:	27bdff88 	addiu	sp,sp,-120
; store word:  [sp+116] := k0
9d000130:	afba0074 	sw	k0,116(sp)
9d000134:	401a6002 	mfc0	k0,c0_srsctl
9d000138:	afbb0070 	sw	k1,112(sp)
9d00013c:	afba006c 	sw	k0,108(sp)
; inserts bits from 0 reg to pos 1 and length 15
; this will zere-out bits 15 to 1 of k1 register
9d000140:	7c1b7844 	ins	k1,zero,0x1,0xf
; or immediate (set bit mask 0x400 of k1)
9d000144:	377b0400 	ori	k1,k1,0x400
; move to Coprocessor C0: C0 Status := k1
9d000148:	409b6000 	mtc0	k1,c0_status
; saving general registers on stack
; ra = return address (MIPS and ARM use dedicated register on call, not stack)
9d00014c:	afbf005c 	sw	ra,92(sp)
9d000150:	afbe0058 	sw	s8,88(sp)
9d000154:	afb90054 	sw	t9,84(sp)
9d000158:	afb80050 	sw	t8,80(sp)
9d00015c:	afaf004c 	sw	t7,76(sp)
9d000160:	afae0048 	sw	t6,72(sp)
9d000164:	afad0044 	sw	t5,68(sp)
9d000168:	afac0040 	sw	t4,64(sp)
9d00016c:	afab003c 	sw	t3,60(sp)
9d000170:	afaa0038 	sw	t2,56(sp)
9d000174:	afa90034 	sw	t1,52(sp)
9d000178:	afa80030 	sw	t0,48(sp)
9d00017c:	afa7002c 	sw	a3,44(sp)
9d000180:	afa60028 	sw	a2,40(sp)
9d000184:	afa50024 	sw	a1,36(sp)
9d000188:	afa40020 	sw	a0,32(sp)
9d00018c:	afa3001c 	sw	v1,28(sp)
9d000190:	afa20018 	sw	v0,24(sp)
9d000194:	afa10014 	sw	at,20(sp)
; move from LO (LO is lower 32-bit result of multiply:
; v0 := LO
9d000198:	00001012 	mflo	v0
9d00019c:	afa20064 	sw	v0,100(sp)
; move from HI (HI is higher 32-bit result of multiply)
9d0001a0:	00001810 	mfhi	v1
9d0001a4:	afa30060 	sw	v1,96(sp)
9d0001a8:	03a0f025 	move	s8,sp
    TIMER_1_InterruptHandler();
; "jump and link" equivalent of Call instruction
; However return address is not pushed to stack, but stored to ra register
9d0001ac:	0f400176 	jal	9d0005d8 <TIMER_1_InterruptHandler>
9d0001b0:	00000000 	nop
}
; SP := S8
9d0001b4:	03c0e825 	move	sp,s8
; Load Word: v0 := [sp+100]
9d0001b8:	8fa20064 	lw	v0,100(sp)
; move to LO: LO := v0
9d0001bc:	00400013 	mtlo	v0
9d0001c0:	8fa30060 	lw	v1,96(sp)
; move to HI: HI := v1
9d0001c4:	00600011 	mthi	v1
; restoring general registers from stack
9d0001c8:	8fbf005c 	lw	ra,92(sp)
9d0001cc:	8fbe0058 	lw	s8,88(sp)
9d0001d0:	8fb90054 	lw	t9,84(sp)
9d0001d4:	8fb80050 	lw	t8,80(sp)
9d0001d8:	8faf004c 	lw	t7,76(sp)
9d0001dc:	8fae0048 	lw	t6,72(sp)
9d0001e0:	8fad0044 	lw	t5,68(sp)
9d0001e4:	8fac0040 	lw	t4,64(sp)
9d0001e8:	8fab003c 	lw	t3,60(sp)
9d0001ec:	8faa0038 	lw	t2,56(sp)
9d0001f0:	8fa90034 	lw	t1,52(sp)
9d0001f4:	8fa80030 	lw	t0,48(sp)
9d0001f8:	8fa7002c 	lw	a3,44(sp)
9d0001fc:	8fa60028 	lw	a2,40(sp)
9d000200:	8fa50024 	lw	a1,36(sp)
9d000204:	8fa40020 	lw	a0,32(sp)
9d000208:	8fa3001c 	lw	v1,28(sp)
9d00020c:	8fa20018 	lw	v0,24(sp)
9d000210:	8fa10014 	lw	at,20(sp)
; disable interrupts
9d000214:	41606000 	di
; Execute Hazard Barrier
; this instructions ensures that CPU waits until
; all core changes are propagated and visible to next instructions
9d000218:	000000c0 	ehb
9d00021c:	8fba0074 	lw	k0,116(sp)
9d000220:	8fbb0070 	lw	k1,112(sp)
9d000224:	409a7000 	mtc0	k0,c0_epc
9d000228:	8fba006c 	lw	k0,108(sp)
9d00022c:	27bd0078 	addiu	sp,sp,120
9d000230:	409a6002 	mtc0	k0,c0_srsctl
;  Write to GPR in Previous Shadow Set
9d000234:	41dde800 	wrpgpr	sp,sp
9d000238:	409b6000 	mtc0	k1,c0_status
; Exception Return (applies also for Interrupts)
9d00023c:	42000018 	eret
```

There are two ways how to handle more interrupts:
- using `Shadow Set` - quickly switching to other general registers
  set for interrupt (similar to some dsPIC chips) - ISR using `SRS` instead of `SOFT` clause.
- Temporal proximity interrupt coalescing - interrupts are grouped and processed by single
  ISR (Interrupt Service Routine) - introduction is on `DS61108B-page 8-31`
- please see [PIC32MX Section 11 - Interrutps][PIC32MX S11 INT] manual for many fine details
  including brief descritpion of Interrupt Service Routines (ISRs)

Here are MIPS32 resources I plan to study:
- https://www.cs.unibo.it/~solmi/teaching/arch_2002-2003/AssemblyLanguageProgDoc.pdf
- [MIPS32 M4K Core Datasheet][MIPS32 M4K DTS]
- [MIPS32 Instruction Set][MIPS32 BIS]

## Software requirements

* [XC32 compiler][XC compilers] - tested version v4.30
* [MPLAB X IDE][MPLAB X IDE] - tested version v6.15
  - using `PIC32MX_DFP` v1.5.259
  - using default MCC Plugin 5 that now also contains Harmony
    (it used Harmony 3)
  - official migration did not work well - manually recreated TMR1 object in MCC
  - used this guide: https://microchipdeveloper.com/harmony3:update-and-configure-existing-mhc-proj-to-mcc-proj1

NOTE: To make MPLAB X IDE at least partially usable please uncheck this:
- Tools -> Options -> Miscellaneous -> Files -> Enable auto-scanning of sources
Otherwise it will hog CPU significantly...

Once you build this project there is also generated assembler listing
using `xc32-objdump -S` in
```
firmware\mplabproj.X\dist\default\TARGET\mplabproj.X.TARGET.lst
```
[PIC32MX S11 INT]: http://ww1.microchip.com/downloads/en/DeviceDoc/61108B.pdf
[MIPS32 M4K Manual]: https://s3-eu-west-1.amazonaws.com/downloads-mips/documents/MD00249-2B-M4K-SUM-02.03.pdf
[MIPS32 M4K DTS]: https://s3-eu-west-1.amazonaws.com/downloads-mips/documents/MD00247-2B-M4K-DTS-02.01.pdf
[MIPS32 BIS]: https://s3-eu-west-1.amazonaws.com/downloads-mips/documents/MD00086-2B-MIPS32BIS-AFP-05.04.pdf
[MIPS32 QRC]: https://s3-eu-west-1.amazonaws.com/downloads-mips/documents/MD00565-2B-MIPS32-QRC-01.01.pdf 
[Harmony]: https://www.microchip.com/mplab/mplab-harmony
[XC compilers]: https://www.microchip.com/mplab/compilers
[MPLAB X IDE]: https://www.microchip.com/mplab/mplab-x-ide
[PIC32MX250F128B]: https://www.microchip.com/wwwproducts/en/PIC32MX250F128B
[PIC Microstick II]: https://www.microchip.com/DevelopmentTools/ProductDetails/dm330013-2

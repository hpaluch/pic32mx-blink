# PIC32MX Blink

Introductory project for [Microstick II][PIC Microstick II] 
with PIC32MX. It just blinks on-board LED D6 (ON RA0, PIN2) using
TMR1 interrupt.

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

[Harmony]: https://www.microchip.com/mplab/mplab-harmony
[XC compilers]: https://www.microchip.com/mplab/compilers
[MPLAB X IDE]: https://www.microchip.com/mplab/mplab-x-ide
[PIC32MX250F128B]: https://www.microchip.com/wwwproducts/en/PIC32MX250F128B
[PIC Microstick II]: https://www.microchip.com/DevelopmentTools/ProductDetails/dm330013-2

# PIC32MX Blink

Introductory project for [Microstick II][PIC Microstick II] 
with PIC32MX. It just blinks on-board LED D6 (ON RA0, PIN2) using
TMR1 interrupt.

# Requirements

## Hardware requirements

* [Microstick II][PIC Microstick II]  demo board
* [PIC32MX250F128B SPDIP][PIC32MX250F128B] inserted into socket U5
  (included with board, should be default)
* programming pins switch S1 in position `A`

## Software requirements

* [XC32 compiler][XC compilers] - tested version v2.30
* [MPLAB X IDE][MPLAB X IDE] - tested version v5.25
  - installed  [MPLAB Harmony Configurator 3 Plugin][Harmony] - tested
    version 3.3.0.1


[Harmony]: https://www.microchip.com/mplab/mplab-harmony
[XC compilers]: https://www.microchip.com/mplab/compilers
[MPLAB X IDE]: https://www.microchip.com/mplab/mplab-x-ide
[PIC32MX250F128B]: https://www.microchip.com/wwwproducts/en/PIC32MX250F128B
[PIC Microstick II]: https://www.microchip.com/DevelopmentTools/ProductDetails/dm330013-2

Hard Fault Demo on STM32L452 Nucleo-64 Board
====

Demo program to show problem with STM32L452 running code in SRAM2. This
demo is designed to run on an STM32L452RE Nucleo-64 board.

Overview
====

This is an Eclipse project generated via STM32CubeMX.  It uses FreeRTOS to 
spawn two threads.  One thread blinks the green LED on the PCB at 1Hz.  The
other thread starts the ADC DMA process reading into a circular buffer at
26400 sps.  These are placed on a message queue in the DMA interrupt.  The
thread then reads each block of these samples from the queue and sends them
to a FIR filter running in SRAM2.  The filter results are discarded.

There are 3 GPIOs defined:

 * ADC_STATUS (PA0)
 * FIR_STATUS (PA1)
 * ERROR (PC0)
 
ADC_STATUS will flip between high and low as the DMA moves between
half-complete and complete.

FIR_STATUS will go high during FIR processing and low otherwise.

ERROR is raised in the hard fault handler.

This code will eventually end up in the hard fault handler, at which point
ERROR is raised and the LED will blink at about 5Hz.

Versions of this code seems to run fine on an STM32L432KC Nucleo32 board and
on a custom PCB using an STM32L433CCU6 chip.  That same board will fail in
the same manner when using an STM32L452CEU6.  The code will run indefinitely
when the FIR filter code is placed in Flash.

Placing Code in SRAM2
====

1. The section "bss2" (yes, poorly named) is defined in the
STM32L452RE_FLASH.ld linker script.
2. Code and data in bss2 is copied to SRAM2 by the
./Src/startup_stm32l452xx.S startup code.
3. The FIR filter code is placed in the bss2 section by modifying
./Drivers/CMSIS/Include/arm_math.h:

```c++
    void arm_fir_f32(
      const arm_fir_instance_f32 * S,
      float32_t * pSrc,
      float32_t * pDst,
      uint32_t blockSize) __attribute__((section(".bss2")));
```

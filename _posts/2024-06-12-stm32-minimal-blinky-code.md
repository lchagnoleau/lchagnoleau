---
title: STM32 minimal blinky code
date: 2024-06-12
categories: [Programming]
tags: [low-level, debug, arm, stm32]
---

## Introduction

This post will show you how to write the minimal blinky code for STM32 microcontrollers.
It is inspired by this [github](https://github.com/samvrlewis/minimal-stm32) project but here we will try to fully understand the code.

## Prerequisites

We only need the svd file for the STM32F429 microcontroller.
We can found the official svd file on the [ST website](https://www.st.com/en/microcontrollers-microprocessors/stm32f429zi.html#cad-resources) -> STM32F4 System View Description.

## Code

The Led we will use is the GPIOB pin 0.

### Main program

#### RCC_AHB1ENR

To enable the led, we start by enabling the GPIOB peripheral clock.
The register controlling the GPIOB peripheral clock is the RCC_AHB1ENR register.

Following the svd file, we are looking foir the address of the RCC_AHB1ENR register.

```
<peripheral>
  <name>RCC</name>
  <description>Reset and clock control</description>
  <groupName>RCC</groupName>
  <baseAddress>0x40023800</baseAddress>
  <addressBlock>
```
```
<register>
  <name>AHB1ENR</name>
  <displayName>AHB1ENR</displayName>
  <description>AHB1 peripheral clock register</description>
  <addressOffset>0x30</addressOffset>
```

We define it like this is our `maiu.c` file:

```c
#define PERIPH_BASE 0x40000000U
#define RCC_BASE (PERIPH_BASE + 0x23800)
#define RCC_AHB1ENR (*(volatile unsigned long *)(RCC_BASE + 0x30))

int main() {
  // Enable clock on GPIOB peripheral
  RCC_AHB1ENR = 1 << 1;
  ...
```

#### GPIOB_MODER

The MODER register is used to set the mode of the GPIOB pin 0.
Like previously, we look for the address of the MODER register in the svd file.

```
<peripheral>
  <name>GPIOB</name>
  <description>General-purpose I/Os</description>
  <groupName>GPIO</groupName>
  <baseAddress>0x40020400</baseAddress>
```
```
<registers>
  <register>
    <name>MODER</name>
    <displayName>MODER</displayName>
    <description>GPIO port mode register</description>
    <addressOffset>0x0</addressOffset>
```

To set the pin 0 as an output, we need to set the bit 0 (because we use the pin 0) of the MODER register to 1.
In the `main.c` file, we define it like this:

```c
#define GPIOB_BASE (PERIPH_BASE + 0x20400)
#define GPIOB_MODER (*(volatile unsigned long *)(GPIOB_BASE + 0x00))

#define LED_PIN 0

int main() {
  ...
  // Put GPIOB0 into output mode
  GPIOB_MODER |= 0x01 << LED_PIN * 2;
  ...
```

#### GPIOB_ODR

Finally, we can toggle the pin 0 of the GPIOB port to make the led blink.
The register controlling the output of the GPIOB pin 0 is the ODR register.

```
<register>
  <name>ODR</name>
  <displayName>ODR</displayName>
  <description>GPIO port output data register</description>
  <addressOffset>0x14</addressOffset>
```

`main.c` file:

```c
#define GPIOB_ODR (*(volatile unsigned long *)(GPIOB_BASE + 0x14))
#define LED_PIN 0

int main() {
  ...
  GPIOB_ODR = 1 << LED_PIN; // set LED pin high
  ...
```

#### Delay

To make the led blink, we need to add a delay.
We simply loop for a certain amount of time.

```c
void sleep(unsigned long sleep_time) {
  while (sleep_time--)
    __asm__("nop");
}
```

#### Vector table

We need to define the vector table at the beginning of the `main.c` file.
The vector table is the first thing the microcontroller reads when it boots up.
The first address of the vector table is the stack pointer and the second address is the reset handler.

We will set the stack pointer to the end of the RAM and the reset handler to the main function.
```c
#define SRAM_BASE 0x20000000U
#define SRAM_SIZE 192 * 1024
#define SRAM_END (SRAM_BASE + SRAM_SIZE)

unsigned long *vector_table[] __attribute__((section(".vector_table"))) = {
    (unsigned long *)SRAM_END, // place stack pointer at the end of SRAM
                               // (stack grows down)
    (unsigned long *)main      // reset handler, jump directly to main
};
```

#### Full code

```c
#define PERIPH_BASE 0x40000000U
#define RCC_BASE (PERIPH_BASE + 0x23800)
#define RCC_AHB1ENR (*(volatile unsigned long *)(RCC_BASE + 0x30))

#define GPIOB_BASE (PERIPH_BASE + 0x20400)
#define GPIOB_MODER (*(volatile unsigned long *)(GPIOB_BASE + 0x00))
#define GPIOB_ODR (*(volatile unsigned long *)(GPIOB_BASE + 0x14))

#define LED_PIN 0

#define SRAM_BASE 0x20000000U
#define SRAM_SIZE 192 * 1024
#define SRAM_END (SRAM_BASE + SRAM_SIZE)

void sleep(unsigned long sleep_time) {
  while (sleep_time--)
    __asm__("nop");
}

int main() {
  // Enable clock on GPIOB peripheral
  RCC_AHB1ENR = 1 << 1;

  // Put GPIOB0 into output mode
  GPIOB_MODER |= 0x01 << LED_PIN * 2;

  while (1) {
    GPIOB_ODR = 1 << LED_PIN; // set LED pin high
    sleep(1000000);
    GPIOB_ODR = 0 << LED_PIN; // set LED pin low
    sleep(1000000);
  }
}

unsigned long *vector_table[] __attribute__((section(".vector_table"))) = {
    (unsigned long *)SRAM_END, // place stack pointer at the end of SRAM
                               // (stack grows down)
    (unsigned long *)main      // reset handler, jump directly to main
};
```

### Linker script

#### MEMORY

In the linker script we will define the memory layout of the microcontroller.
The address of the FLASH and the RAM are defined in the datasheet of the microcontroller.

```ld
MEMORY
{
    FLASH (rx)  : ORIGIN = 0x08000000, LENGTH = 2048K
    SRAM (xrw)  : ORIGIN = 0x20000000, LENGTH = 192K
}
```

#### SECTIONS

We will only define the `.text` section for the code and the `.vector_table` section for the vector table.
The `.text` section is used to store the code and the `.vector_table` section is used to store the vector table.

```ld
SECTIONS
{
    .text :
    {
        *(.vector_table)
        *(.text)            /* code */
    } >FLASH
}
```

The `>FLASH` means that the `.text` section will be stored in the FLASH memory.

#### Full linker script

```ld
MEMORY
{
    FLASH (rx)  : ORIGIN = 0x08000000, LENGTH = 2048K
    SRAM (xrw)  : ORIGIN = 0x20000000, LENGTH = 192K
}

SECTIONS
{
    .text :
    {
        *(.vector_table)
        *(.text)            /* code */
    } >FLASH
}
```

### Makefile

The Makefile is used to compile the code:

```makefile
LINKER_SCRIPT = linker_script.ld
C_FLAGS  = -c -Og -mcpu=cortex-m4 -mthumb -g3 -ggdb3
OPENOCD_FLAGS = -f interface/stlink-v2-1.cfg -f target/stm32f4x.cfg
 
all: main.elf
 
%.o: %.c
	arm-none-eabi-gcc $(C_FLAGS) -o $@ $<

main.elf: main.o
	arm-none-eabi-ld -T$(LINKER_SCRIPT) -o main.elf main.o

clean:
	rm -rf *.o *.elf

flash: main.elf
	openocd $(OPENOCD_FLAGS) -c "program main.elf reset" -c "shutdown"
```

## Compile and flash

To compile the code, you can use the following command:

```bash
make
```

To flash the code on the microcontroller, you can use the following command:

```bash
make flash
```

The led should blink.

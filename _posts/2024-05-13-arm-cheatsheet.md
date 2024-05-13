---
title: ARM keywords cheatsheet
date: 2024-05-13
categories: [Programming]
tags: [low-level, debug, arm, cheatsheet]
---

## Glossary

| Name | Meaning |
|-----|-----------------------|
| ARM | Advanced RISC Machines |
| RISC | Reduced Instruction Set Computer |
| I-cache | [Instruction Cache](##I-cache) |
| D-cache | [Data Cache](##D-chache) |
| LR | [Link Register](##Link_Register) |
| SP | [Stack Pointer](##Stack_Pointer) |
| PC | [Program Counter](##Program_Counter) |

## I-cache

Instruction cache is a high speed memory that stores the instructions that the CPU is likely to execute next.
It is used to speed up the execution of instructions by reducing the time it takes to fetch instructions from the main memory.
I-chache is the first place that the processor looks for instructions to execute. If the instruction is found in the cache, it is executed immediately.
If the instruction is not found in the cache, it is fetched from the main memory and stored in the cache for future use.

## D-cache

Data cache is a high speed memory that stores the data that the CPU is likely to access next.

## Link Register

The link register is a register which holds the address to return to when a subroutine call completes.
It is also named R14.

## Stack Pointer

The stack pointer is a register which holds the address of the top of the stack. It is also named R13.

## Program Counter

The program counter is a register which holds the address of the next instruction to be executed. It is also named R15.

% Copyright (c) 2026, Jesse DeGuire
% All rights reserved.
% Licensed using a BSD 3-clause license, see LICENSE at the root of this project.
% Find this project on GitHub at https://github.com/jdeguire/mchpclang_docs.

# Builtin Functions

This chapter describes some predefined functions and function-like macros you may find useful. This
is not a comprehensive list, so you would want to check out Clang and CMSIS documentation for more
info. You can also browse the CMSIS header files in the install location at `CMSIS/Core/Include`. If
you are using an older ARM chip, then this toolchain provides some CMSIS-like functionality in
`arm/include/arm_legacy`.


## Common Functions
CMSIS provides quite a few inline functions and function-like macros you can easily use in your code.
CMSIS (or legacy ARM support for older devices) is provided to you by simply including your processor
specific header or `<which_device.h>`.

Quite a few of these map directly to CPU instructions of the same name: `__NOP()`, `__BKPT()`, `__CLZ()`,
and the memory barrier instructions (explained more below). There are a few less-commonly used macros
for event handling, such as `__WFI()`, `__WFE()`, `__SEV()`. There are more macros for more instructions,
which you can find by browsing the compiler-specific header files in CMSIS.

Another pair of functions you will probably use often in your embedded developement are `__enable_irq()`
to enable interrupts and `__disable_irq()` to disable them. On newer ARM devices, these map directly
to the instructions for enable and disabling interrupts. On older ARM devices, these map to short
instruction sequeences to modify the CPSR register. If you want to check if interrupts are enabled
on microcontroller devices, you can check the memory-mapped PRIMASK register. If you want to check
on microprocessor devices, you need to check some bits in the CPSR register using the `__get_CPSR()`
function.

On some devices, particularly the microcontroller parts, the instruction used to disable interrupts
is "self-synchronizing", meaning that you do not need to put a memory barrier instruction after it.
Other devices, particularly older ones, might require you to do so. In that case, you would want to
use `__ISB()` (see below) right after you disable interrupts. The instruction used to enable interrupts
is never self-synchronizing, but that is presumably less important unless you really need to be sure
interrupts are enabled for the next instruction.

The other set of functions you will need are ones to handle interrupts. This is covered in more detail
[here](./using_device_features.md#interrupts-for-arm-microcontrollers) for microcontrollers and
[here](./using_device_features.md#interrupts-for-arm-microprocessors) for microprocessors. CMSIS
does provide a function for the microcontrollers to perform a system reset called `__NVIC_SystemReset()`.
You will need to consult your device's datasheet to learn how to perform a reset if you are using a
microprocessor device.


## Memory Barrier Functions
Most ARM processors support three types of "memory barrier" instructions. These are instructions or
sequences that forces the CPU to apply some ordering to memory operations that occur before and after
the instruction. You use these instructions when you need to ensure that a memory access happens in
the order you expect it to. This is generally most important when you are accessing things that
affect the system as a whole, such as disabling interrupts, modifying certain system registers,
or doing cache operations. In those cases, functions provided by CMSIS (or mchpClang's legacy ARM
supoprt) will use the correct barriers for you.

Also provided is the `__COMPILER_BARRIER()` macro to tell the compiler to not reorder memory
access instructions past it. This macro evaluates to `asm volatile("":::"memory")`,
which is an expression supported in GCC and Clang.

- `__DMB()`: Data Memory Barrier  
This tells the processor to wait for all outstanding memory accesses to complete before allowing any
memory access instructions after this to run. This is sort of like the compiler barrier described above
in that the intent is to ensure the desired ordering of memory accesses. On newer ARM cores, this is
an explicit `dmb` instruction. On ARMv6, this writes to CP15 register 7. ARMv5 and older have no
equivalent and so this is the same as using the compiler barrier described above.
- `__DSB()`: Data Synchronization Barrier  
This is a more restrictive form of DMB in that this waits for all outstanding memory accesses to
complete before allowing *any* instruction after this to execute. Use this when your memory access
might have important side effects that need to propogate, particularly when modifying certain system
control registers. On newer ARM cores, this is an explicit `dsb` instruction. On ARMv6 and older, this
is called a "Drain Write Buffer" operation and writes to CP15 register 7.
- `__ISB()`: Instruction Synchroniztion Barrier  
This flushes the CPU pipeline and any instruction prefetch buffers in the CPU, causing instructions
following this to be re-fetched after this completes. This is useful for ensuring that upcoming
instructions "see" a new system state, such as changes in system control registers or cache state.
On newer ARM cores, this is an explicit `isb` instruction. On ARMv6, this is called a "Prefetch
Buffer Flush" and writes to CP15 register 7.  
On ARMv5 and older, this was called a "Instruction Memory Barrier" and is implementation-specific.
The implementation provided by mchpClang follows Section 2.7.4 in Part A of the ARMv5TE Reference
Manual, which says that a restricted form of IMB can simply be any instruction other than `b`, `bl`,
or `blx` that updates PC. This implementation therefore uses a dummy `ldr pc, ...` to update the PC
to one instruction ahead. You may need to do additional operations for a "full" IMB; see the Technical
Reference Manual for your CPU for more info.

These barrier function-like macros are implemented to also perform the equivalent of the compiler
barrier described above. If you need to perform both a data and instruction synchroniziation barrier,
then do the DSB first followed immediately by the ISB. This ensures that re-fetched instructions do
not "see" a stale device state.

You may want to do some extra reading to better understand when and how to use these. It's okay, *I*
also need to do some extra reading because it can be confusing at times. Searching for "arm memory
barrier" online yields some useful results from ARM's developer site, including a document titled
"ARM Cortex-M Programming Guide to Memory Barrier Instructions". That document provides good info on
when and how to use these instructions.


## Cache Maintenance Functions
TODO

## System Control Functions
TODO
CP15 registers on the microprocessors and control registers on the microcontrollers.

## Compiler Built-in Functions
TODO


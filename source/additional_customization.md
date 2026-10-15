% Copyright (c) 2026, Jesse DeGuire
% All rights reserved.
% Licensed using a BSD 3-clause license, see LICENSE at the root of this project.
% Find this project on GitHub at https://github.com/jdeguire/mchpclang_docs.

# Additional Customization

This chapter describes ways to customize your application in ways not already discussed in previous
chapters. At the more basic level are functions you can define in your application that, for example,
redirect `stdout` to a serial port. More advanced uses include using your own C startup code and linker
scripts.


## Adding `stdio` Support
LLVM-libc includes support for basic I/O on baremetal systems. That is, you can use functions like
`printf()`, `puts()`, and `scanf()`. You can also access the `stdin`, `stdout` and `stderr` FILE
streams. To make this work, your application needs to provide a few definitions for LLVM-libc to use.

```{note}
LLVM-libc as of version 23 does not yet support full file I/O on baremetal system. So things like
`fopen()`, `fseek()`, and similar functions that directly take `FILE *` do not yet work. Work on
those is ongoing and might be available in LLVM 24. See [this](https://github.com/llvm/llvm-project/pull/221211)
GitHub pull request for more info.
```

First, you need to define a "cookie" type for LLVM-libc to use to handle file streams on baremetal
applications. This type can either be empty or contain data you would use to interact with your I/O
interface. For example, if you were talking over a UDP interface, you might define your cookie to
contain connection info such as IP address, port, and stuff like that. You then create three globals
of this type corresponding to each of the three standard FILE streams. The name of the structure and
the names of the variables must match exactly what is shown here. LLVM-libc references these cookies
with `extern` declarations, which is why they need to be global variables.

```c++
struct __llvm_libc_stdio_cookie
{
    // Put whatever you want in here.
};

struct __llvm_libc_stdio_cookie __llvm_libc_stdin_cookie;
struct __llvm_libc_stdio_cookie __llvm_libc_stdout_cookie;
struct __llvm_libc_stdio_cookie __llvm_libc_stderr_cookie;
```

You then need to define just two functions: `__llvm_libc_stdio_read()` and `__llvm_libc_stdio_write()`.
The first parameter to these functions is a `void *` to a cookie. You can use this to check which of
your cookies you need to work with. The other parameters and return values for these are shown in
the example code below. This is a very simple example for directing `stdio` to a serial port, but
you can use them however you like.

```c++
// Remember to make these two extern "C" if you define them in a C++ file!
//
extern "C" ssize_t __llvm_libc_stdio_read(void *cookie, char *buf, size_t size)
{
    ssize_t bytes_read = 0;

    if(&__llvm_libc_stdin_cookie == cookie)
    {
        bytes_read = read_from_serial_port(buf, size);
    }

    return bytes_read;
}

extern "C" ssize_t __llvm_libc_stdio_write(void *cookie, const char *buf, size_t size)
{
    ssize_t bytes_written = 0;

    // These could go to different places, but here we'll just send them to the same place.
    if(&__llvm_libc_stdout_cookie == cookie  ||  &__llvm_libc_stderr_cookie == cookie)
    {
        bytes_written = write_to_serial_port(buf, size);
    }

    return bytes_written;
}
```

Notice that the return value is signed. This lets you return error codes as negative numbers. Any
error code you return like this will be made positive then stored in your application's `errno`
variable (see [](#custom-errno-handling) for more info about `errno`).


## Clock and Time Support
If you want to use C functions like `clock()` or `timespec_get()`, you need to implement two functions
that provide LLVM-libc with `timespec` structures. This is a structure that contains a field for
seconds and another for nanoseconds.

```c++
extern "C" bool __llvm_libc_timespec_get_utc(struct timespec *ts);
extern "C" bool __llvm_libc_timespec_get_active(struct timespec *ts);
```

The only difference between the two is that `utc` is used to get the time since an implementation
defined epoch (Unix time is used here, which starts 1-Jan-1970) and `active` is used to get the time
since process or application start.

Note that on baremetal ARM platforms, the standard `CLOCKS_PER_SEC` macro is only 100. This means
that `clock()` will return a time with only 10ms resolution. This is done to maintain compatibility
with a feature called "semihosting". This is a mechanism to let ARM devices run standard I/O through
a connected debugger tool to the host computer.


## Custom Exit Functions
The `exit()` function in LLVM-libc calls two other functions that you can customize. Both of these
have weak implementations in the C startup code for your device. You can customize them by providing
your own implementations in your application.

The first is `__cxa_finalize(void *)`. This is normally used to call destructors of C++ global objects
that were created at startup and to call functions that you registered with `atexit()`. The default
weak implementation does nothing at all. Baremetal apps normally do not exit like a regular software
program would and so this would normally never run. The `void *` parameter relates to unloading
shared libraries, which is another thing baremetal apps do not deal with.

The second is `__llvm_libc_exit(int)`. This is the final call made by `exit()` and so it must never
return. The `int` parameter is the exit code that was passed to `exit()`. The default weak implementation
will simply spin in a loop forever. You could override this to, for example, reset your device or
print some debug message.


## Thread-local Variables
Your application can use thread-local variables with the `thread_local` keyword. However, the default
implementation used by mchpClang assumes that you are running a baremetal application with only a
single thread of execution. This would be useful if you brought in external code that expected support
for thread-local variables, but probably not much else.

When you access a thread-local on an ARM device, the compiler generates a call to `void *__aeabi_read_tp(void)`
to get a pointer to a structure called a "thread control block". We do not have a thread control block,
but it just so happens that on 32-bit ARM systems this pointer is always 8 bytes before the thread-local
data. Therefore, the default weak implementation of this function in the startup code returns the
start of the thread-local data minus 8. If you are curious, this would be 16 bytes on a 64-bit ARM
system.

If you wanted to, for example, support thread-locals in an RTOS you are using, then you would create
your own implementation of `void *__aeabi_read_tp(void)` that returns the address of your current
thread's local storage minus 8 (or 16 on 64-bit ARM).

Thread-locals are put into the special linker sections `.tdata` and `.tbss` for initialized and zeroed
out data, respectively. The two sections are contiguous, with `.tdata` coming directly before `.tbss`
in RAM. The startup code initializes these along with the non-local `.data` and `.bss` sections.
Continuing with the RTOS example, you will need to do something similar when you access thread-locals
for your thread. You can do this either when the thread is created or have a flag you check in your
`__aeabi_read_tp()` function to see if you need to initialize them.

You can use linker symbols to help you access the `.tdata` and `.tbss` sections along with the location
in flash from which `.tdata` is initialized. See [](#accessing-symbols-from-code) in the Linker Scripts
section for more info about how to access linker symbols in your code.

| Symbol             | Description |
| ------------------ | ----------- |
| __tls_base         | The start of the TLS sections in RAM. |
| __tdata_start      | The start of the `.tdata` section in RAM. |
| __tdata_source     | The start of the data in flash used to init `.tdata`. |
| __tdata_source_end | Just past the end of the data in flash for `.tdata`. |
| __tdata_size       | The size in bytes of `.tdata`. |
| __tbss_start       | The start of the `.tbss` section in RAM. |
| __tbss_offset      | __tbss_start - __tdata_start. |
| __tbss_end         | Just past the end of the `.tbss` section in RAM. |
| __tbss_size        | The size in bytes of `.tbss`. |
| __tls_size         | The size in bytes of the TLS sections. |
| __tls_end          | Just past the end of the TLS sections in RAM. |

Note that `.tbss` (and `.bss` for that matter) contains data that is initialized to zero and so there
is no "source" needed in flash.


## Custom `errno` Handling
Like with thread-local storage, the default implementation of the C global variable `errno` is suitable
for a typical baremetal application with only a single thread. This is accessed through the function
`int *__llvm_libc_errno()`. The default weak implementation of this function in the startup code
simply returns a pointer to a single statically-allocated `errno` variable. You can provide your
own implementation of this function to, for example, point to a thread-specific `errno` if you were
running an RTOS.


## Startup Code
The last few sections have mentioned "startup code", so this section will describe it in a bit more
detail.

The startup code runs before `main()`. It both sets up a few things in your device to prepare it to
run and also initializes global C and C++ variables in your application. When an ARM device resets
or powers up, it jumps to the `Reset_Handler` located in its interrupt vector table and starts
executing code. The interrupt vector table is also defined in the startup code. It is placed into a
special section in the linker script, usually at the start of program space, so that it can be found.

As indicated in the previous sections, the startup code also contains a few extra functions. These
are not directly related to startup, but are still important parts of the runtime environment being
set up for your device.

Each supported device has its own startup code. You can find the object file that is linked and the
source code in the toolchain location at `arm/proc/<device_name>`. If you want to use your own startup
code, then you should copy the file for your device into your project as a starting point and modify
it. Pass `-nostartfiles` to Clang when linking to not include the default startup code. You might
need to also pass `crt1.o` on the command line when you use that option.

Here are quick rundowns of what the reset handlers in the startup code do.

### Microcontrollers
1. Initialize the two stack pointer registerss `msp` and `psp`. Also initialize the two stack limit
registers `msplim` and `psplim` on devices that have them. If the device supports Stack Sealing,
the top of the stack is initialized with ARM's recommended sealing value.
2. If your application has a function called `_on_reset()`, then call that function. This neither
takes nor returns anything. You should assume that nothing other than the stacks have been initialized
at this point. 
3. Enable the FPU, CPU caches, branch predictors, and CMCC peripheral in that order if each one is
present on the device.
4. Set the `VTOR` register to the base address of the interrupt vector table. This can be used to
change the table used at runtime.
5. Initialize the `.data`/`.tdata` and `.bss`/`.tbss` sections. The normal and thread-local sections
are allocated in the linker script such that the two data and two bss sections can be initialized
together.
6. Call `__libc_init_array()`, which in turn calls C++ constructors for global objects and also
calls functions with the `constructor` attribute.
7. If your application has a function called `_on_bootstrap()`, then call that function. This neither
takes nor returns anything. This happens right before `main()` is called and so initialization is
done.
8. Call `main()`.
9. Call `exit()` with the value returned from `main()`.

### Microprocessors
1. Disable interrupts by clearing the `F` and `I` flags in the `CPSR` register.
2. Initialize the `SCTLR` register (CP15 register 1) to disable caches, the MMU, and branch predictor
if present. Set the vector table base address to 0x00.
3. Set the stack pointers for each processor mode to the tops of stacks defined by the linker script.
When this is done, the processor will be in "System" mode.
4. If your application has a function called `_on_reset()`, then call that function. This neither
takes nor returns anything. You should assume that nothing other than the stacks have been initialized
at this point.
5. Enable the FPU if one is present on the device.
6. Copy the interrupt vector table from program space to the start of SRAM. Its initial location is
used by a bootloader to run your app, but it needs to be moved so that the CPU can use it when your
app is up and running.
7. Zero out the `.bss` and `.tbss` sections. The `.data` and `.tdata` sections do not need copying
because the whole application runs from RAM in a microprocessor.
8. Configure device-specific bus matrix registers to remap SRAM to address 0x00 so that the vector
table that was copied ealier is found at the proper address.
9. Call `__libc_init_array()`, which in turn calls C++ constructors for global objects and also
calls functions with the `constructor` attribute.
10. If your application has a function called `_on_bootstrap()`, then call that function. This neither
takes nor returns anything. This happens right before `main()` is called and so initialization is
done.
11. Call `main()`.
12. Call `exit()` with the value returned from `main()`.

Notice that the microprocessor startup code does **NOT** enable the CPU caches. This is because your
application will need to configure the device's memory management unit before enabling the caches.


## Linker Scripts
A linker script is a special text file that tells the linker at what memory addresses to put program
code, data, and whatever else might be in your application. Baremetal devices need linker scripts so
that the linker knows where the flash and RAM on your device is located. Every supported device therefore
has a default linker script that is used if you do not provide your own. You can find them in the
toolchain location at `arm/proc/<device_name>`. The device configuration file you give to Clang with
the  `--config` option tells it where to find the default linker script.

If you want to use your own linker script, then copy the default one for your device into your own
project and start modifying from there. Pass `-Wl,-T,your_new_script.ld` to Clang when linking to
have it use your script, of course putting your script's name in there.

This section will not go into a ton of detail about linker scripts because the GNU `ld` manual for
linker scripts is really good. Check it out at https://sourceware.org/binutils/docs/ld/Scripts.html
if you are interested in learning more. This section will cover some basics to help you get started.
You should open up a default linker script in the directory given above and follow along.

### Symbols
You can declare symbols in linker scripts using C-like syntax. The most basic is `symbol = expression;`,
but you can use the other C unary assignment operators like `+=`, `-=`, and so on. Math is allowed,
so you can do something like `FOO = expr1 + expr2`. There are also some built-in functions you can
call, which are conered in the GNU `ld` manual. Most of the ones you will see in the default linker
scripts are pretty easy to figure out.

If you define a symbol in your linker script that is also defined in your app, you will get a linker
error. You can use `PROVIDE(symbol = expression);` to instead tell the linker to define a symbol only
if it is referenced and not already defined. If the symbol is defined in your app, the linker will
use that definition instead of giving an error.

There is a special symbol `.` called the "location counter". This indicates the current memory address
at which the linker will put output data. When output data is added, the location counter increments
to the next memory address after the data. The location counter can be read to get the current memory
location or written to update the current memory location. The location counter is regularly written
to with the `ALIGN` function to align it to a particular boundary.

Symbol values are **always** memory addresses, which will be important in the next section. You can
still work with them as though they were integers in the linker script. You will see symbols indicate
the size of a section, for example.

### Accessing Symbols From Code
You can access a linker symbol from your C or C++ code using an `extern` declaration just like any
other external symbol. However, a linker symbol is different in that really only is a memory address.
It does not really have a value or, to put it another way, its value is whatever happens to be in
memory at the address you gave it in the linker script.

Because of this, you generally will want to work with the address of the symbol rather than the
symbol directory. For example, take the `__bss_size` symbol. It gives the size of the `.bss` section
in your app. If you want to print that out, then you need to take its address.

```c
extern unsigned __bss_size;
printf("Size of .bss is %u bytes\n", __bss_size);               // NO
printf("Size of .bss is %u bytes\n", (unsigned)&__bss_size);    // YES
```

Notice, then, that the type you declare the symbol in your code does not matter in this case. Since
you just want the address, you could have declared it as `extern char __bss_size`.

There could be instances in which you **do** want to access a symbol directly, but you need to be
sure you know what is at the memory location of the symbol. For example, you could create your own
linker section with some data you want to put in there. If you create a symbol whose address is the
beginning of that section, then you could access that symbol directly to access the section's contents.
Here, your `extern` declaration in your code would have a type that makes sense for your section.
You could maybe use `extern float __my_symbol;` if you were storing calibration data or something like
that. 

### Some Usable Symbols
The linker scripts define a bunch of symbols, some of which you saw in the previous [](#thread-local-variables)
section. Here is a larger list of symbols you can access in your code.

| Symbol              | Description |
| ------------------- | ----------- |
| __stext             | The start of the code sections (eg. `.text`). |
| __etext             | Just past the end of the code sections. |
| __data_start        | The start of the `.data` section in RAM. |
| __non_tls_data_end  | Just past the end of the `.data` section in RAM. |
| __data_source       | The start of the data in flash used to init `.data`. |
| __data_source_end   | Just past the end of the data in flash for `.data` and `.tdata`. |
| __data_size         | The size in bytes of `.data` and `.tdata`. |
| __tls_base          | The start of the TLS sections in RAM. |
| __tdata_start       | The start of the `.tdata` section in RAM. |
| __tdata_source      | The start of the data in flash used to init `.tdata`. |
| __tdata_source_end  | Just past the end of the data in flash for `.tdata`. |
| __tdata_size        | The size in bytes of `.tdata`. |
| __tbss_start        | The start of the `.tbss` section in RAM. |
| __tbss_offset       | __tbss_start - __tdata_start. |
| __tbss_end          | Just past the end of the `.tbss` section in RAM. |
| __tbss_size         | The size in bytes of `.tbss`. |
| __tls_size          | The size in bytes of the TLS sections. |
| __tls_end           | Just past the end of the TLS sections in RAM. |
| __bss_start         | The start of the `.tbss` section in RAM. |
| __non_tls_bss_start | The start of the `.bss` section in RAM. |
| __bss_end           | Just past the end of the `.bss` section in RAM. |
| __bss_size          | The size in bytes of `.tbss` and `.bss`. |
| __end               | Just past the end of the statically-allocated sections. |
| __heap_start        | Start of space allocated to the heap. |
| __HEAP_SIZE         | Desired heap size (set this on linker command line). |
| __heap_size         | Heap size after alignment. |
| __heap_end          | Just past the end of the space for the heap. |
| __llvm_libc_heap_limit | Synonym for __heap_end used by LLVM-libc. |
| __stack_limit       | Lowest address of stack before colliding with heap. |
| __STACK_SIZE        | Minimum stack size (set this on the linker command line). |
| __stack             | Initial top of stack at startup. |

The four data sections, `.data`, `.tdata`, `.tbss`, and `.bss`, can make understanding these symbols
confusing. Here is a diagram to try to illustrate how they relate. This is just meant to show relative
positioning, not an absolute memory layout. Notice that some symbols cover two sections, such as
`__bss_size` being the size of both `.bss` and `.tbss`.

```
                                               Highest Address
                                        -->    ---------------    <-- __stack
                                        |      |             |
                                        |      |             |
                           __STACK_SIZE |      |   .stack    |
                                        |      |             |
                                        |      |             |
                                        -->    ---------------    <-- (__stack - __STACK_SIZE)
                                               |             |
                                               |    Free     |
                                               |             |
                                        -->    ---------------    <--| __heap_end, __llvm_libc_heap_limit,
                                        |      |             |       | __stack_liimt
                                        |      |             |
                            __HEAP_SIZE |      |    .heap    |
                                        |      |             |
                                        -->    |             |    <--| __heap_start (aligned to 8-byte boundary after .bss)
                                        -->    ---------------    <--| __bss_end, __end
                                        |      |             |
                                        |      |             |
                            __bss_size  |      |    .bss     |
                                        |      |             |
                                        |      |             |
                 -->              -->   |      ---------------    <--| __non_tls_bss_start, __tls_end,
                 |                |     |      |             |       | __tbss_end
                 |                |     |      |             |
      __tls_size |    __tbss_size |     |      |    .tbss    |
                 |                |     |      |             |
                 |                |     |      |             |       | __edata, 
            -->  |                -->   -->    ---------------    <--| __tdata_end, __data_end,
            |    |                |            |             |       | __tbss_start, __bss_start
            |    |                |            |             |
            |    |   __tdata_size |            |   .tdata    |
            |    |                |            |             |
            |    |                |            |             |
__data_size |    -->              -->          ---------------    <--| __non_tls_data_end, __tls_base,
            |                                  |             |       | __tdata_start
            |                                  |             |
            |                                  |    .data    |
            |                                  |             |
            |                                  |             |
            -->                                ---------------    <--| __data_start
                                               Lowest Address 
```

Microcontrollers have additional `source` symbols to indicate the location in flash at which
initialization data for the `.data` and `.tdata` sections is located. The startup code copies the
data from flash into these two sections. The source data is located just after the code sections,
which include `.text` and special ARM sections for handling exceptions. Microprocessors do not need
this because the entire application is loaded into RAM from a bootloader and therefore `.data` and
`.tdata` are aleady initialized.


### Memory Regions and Sections
Near the top of the linker scripts, you will see a list of stuff inside of a `MEMORY` command. As
you probably guessed, this describes the memory regions in your device. You will see the starting
address and length of your flash, data, and peripheral regions, among others. You can look at the
datasheet for your device and find most, if not all, regions listed in here.

Below that, you will see another huge `SECTIONS` command with lots of entries. This is how you tell
the linker how to collect input sections (code vs data vs stuff with the `section` attribute) from
your object files into output sections that go into the output file (the ELF file in our case). These
output sections are then directed to the memory regions that were defined in the `MEMORY` command.
In most cases, you don't need to worry about this because the linker script will put stuff where it
needs to go. These default scripts were adapted from examples in ARM's CMSIS framework, so there are
ARM-specific sections in here, too.

You can add your own section here if you wanted and put stuff in there using the `section` attribute
for functions and variables. Here is an example of what that could look like for a simple section
that goes into the RAM on your device. The actual name of the RAM region varies by device, so you
will need to use the proper name in your actual script.

```c
/* In the linker script. */
.my_section :
{
    . = ALIGN(4);    /* start section on 4-byte alignment */
    __my_section_start__ = .;
    KEEP(*(.my_section*))
    __my_section_end__ = .;
} > ram

/* In your code.
   You will have to initialize this separately because the startup
   code will not know about your new section. */
uint64_t foo __attribute__((section(".my_section")));
```


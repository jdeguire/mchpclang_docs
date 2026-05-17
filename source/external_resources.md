% Copyright (c) 2026, Jesse DeGuire
% All rights reserved.
% Licensed using a BSD 3-clause license, see LICENSE at the root of this project.
% Find this project on GitHub at https://github.com/jdeguire/mchpclang_docs.

# External Resources
Here are some places you can find more info about about LLVM, Clang, the runtime libraries, or stuff
specific to ARM devices.


## Clang Documentation
You can find documentation for the Clang compiler on the web at <https://clang.llvm.org/docs/index.html>
or in the mchpClang install location [here](llvm:clang/html/index.html). The online versions is usually
for the latest LLVM/Clang version, which may not have been released yet. A lot of options and features
are also supported by GCC because Clang is purposefully mostly compatible with GCC, so you can look
online for GCC docs too for even more info.

The first few sections of the "Using Clang as a Compiler" heading are probably the most useful to
you as a user. Here are the direct links to ones you probably want to focus on.

- Clang Compiler User's Manual  
Online: <https://clang.llvm.org/docs/UsersManual.html>  
Local: [Users Manual](llvm:clang/html/UsersManual.html)
- Clang Language Extensions  
Online: <https://clang.llvm.org/docs/LanguageExtensions.html>  
Local: [Language Extensions](llvm:clang/html/LanguageExtensions.html)
- Clang Command Line Argument Reference  
Online: <https://clang.llvm.org/docs/ClangCommandLineReference.html>  
Local: [Command Line Reference](llvm:clang/html/ClangCommandLineReference.html)
- Attributes in Clang  
Online: <https://clang.llvm.org/docs/AttributeReference.html>  
Local: [Attribute Reference](llvm:clang/html/AttributeReference.html)

In addition, you might find the sections under the "Using Clang Tools" heading useful if you want to
use things like `clang-format` or `clang-tidy`. You can find docs for the latter online at
<https://clang.llvm.org/extra/clang-tidy/> or locally [here](llvm:clang-tools/html/clang-tidy/index.html).


## LLVM Documentation
A lot of the LLVM documentation is geared towards developers who want to learn how to build or work
on LLVM itself. A lot of it is therefore probably not useful to a regular user. Still, feel free to
check them out at <https://llvm.org/docs/> or locally [here](llvm:llvm/html/index.html) if your are
curious.

An LLVM document that might be useful to you is the "LLVM Command Guide" that you can find at
<https://llvm.org/docs/CommandGuide/index.html> or locally [here](llvm:llvm/html/CommandGuide/index.html).
You have seen a few of these tools already in the [Other Tools](other_tools.md) chapter of this document.

Like the LLVM documentation, the `lld` linker documentation is largely geared towards developers.
You can find more info about `lld` at <https://lld.llvm.org/> locally [here](llvm:lld/html/index.html).
There is not a lot of useful information for users, so you probably want to rely on GCC or GNU `ld`
docs online instead.


## Runtime Libraries
Here are some links to documentation for the runtime libraries. Of particular importance in those
docs are sections that indicate the C and C++ standards implementation status for these libraries.

- LLVM-libc C standard library  
Online: <https://libc.llvm.org/>  
Local: [LLVM C Library](runtimes:libc/html/index.html)
- libc++ C++ standard library  
Online: <https://libcxx.llvm.org/>  
Local: [LLVM C++ Library](runtimes:libcxx/html/index.html)
- libunwind LLVM Unwinder  
I could not find an "official" documentation site online. Weird.  
Local: [LLVM Unwinder](runtimes:libunwind/html/index.html)
- compiler-rt runtime libraries  
Online: <https://compiler-rt.llvm.org/>
These docs are not currently available locally.


## CMSIS
Arm® provides a device support library called the Common Microcontroller Software Interface Standard,
or CMSIS (pronounced "Sim-sis"). CMSIS is a common base set of macros and routines that will work on
any Cortex®-M MCU and Cortex-A MPUs that use the ARMv7-A instruction set, no matter the vendor. It
includes macros and routines to do things like do cache maintenance operations, access system control
registers, and handle interrupts.

This distrubtion includes an at-the-time recent version of the core CMSIS libraries. These are included
from the device-specific header files so you do not have to do this manually. You can find more info
about CMSIS at <https://www.arm.com/technologies/cmsis>.

CMSIS does not support Arm products that predate the Cortex branding, so this distribution provides
a small set of CMSIS-like functionality you can use. It is intended to support ARMv4 through ARMv6
devices, but most testing was done using the ARM926EJ-S (ARMv5TE). Like CMSIS, this will be included
from device-specific header files for appropriate devices. You will still want to consult documentation
for your CPU to figure out which features apply to your device.

Arm provides additional libraries under the CMSIS umbrella, such as CMSIS-NN for neural network math
or CMSIS-DAP for using their Debug Access Protocol. This additional libraries are NOT provided with
this distribution.

CMSIS also has examples for how to create device-specific files, like headers, linker scripts, and
startup code. This distribution used those examples to create the device-specific files included
with it.


## ACLE
Clang provides support for the Arm C Language Extensions, or ACLE. These are extensions that are
built into the compiler to provide things like predefined macros to provide device info, more builtin
functions to access device features beyond what CMSIS provides, and even new attributes (think the
GNU `__attribute__` keyword or the C++11 `[[attr]]` syntax).

You can find more information on the Arm developer site at <https://developer.arm.com/Architectures/Arm%20C%20Language%20Extensions>.
There are separate specification documents for base ACLE support, SIMD operations for MCUs and MPUs,
and security extensions.

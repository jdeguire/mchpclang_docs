% Copyright (c) 2026, Jesse DeGuire
% All rights reserved.
% Licensed using a BSD 3-clause license, see LICENSE at the root of this project.
% Find this project on GitHub at https://github.com/jdeguire/mchpclang_docs.

# Builtin Macros

This chapter describes some predefined preprocessor macros you may find useful. This is nowhere near
an exhaustive list--there are *a lot* of macros available--so this will also tell you where you can
learn more.

Many of these macros are provided for you when you include `which_device.h` in your code. This is a
header file that will include other header file appropriate for your device. For this to work properly,
you need to use the `--config` option on the command line to tell Clang which device you are using.
See [here](compiling_and_linking.md#device-config-files) for more info.

Many other macros are built into Clang and are generated based on the options you give it, including
the aforementioned `--config` option. Add `-dM -E --` or `-dM -E /dev/null` to the end of your compile
command line options to tell Clang to print those out instead of doing a normal compile. This also
works for GCC.


## Standard Language Macros
These are macros defined by the C or C++ standards used to check for certain language or build features.
This list will indicate the language standard that introduced the macro, though Clang and GCC may let
you use some of these even if you are building with an older standard. Also, macros that start with 
`__STDC` are meant for C, but Clang (and GCC) will also define them when building in C++ mode.
Conversly, macros that start with `__STDCPP` are defined only in C++ mode.

- `__STDC__`  
This is always present and expands to 1. This is defined even when building in C++ mode.
- `__STDC_VERSION__` (C95; C++11)  
This expands to a `long` whose value indicates the which version of the C standard is in use. This
is NOT defined if your compiler is strictly conforming to C89/C90 mode. Here are some values defined
so far. Newer C standards will add new values, so if your future standard is not listed here then
you will need to check documentation online for the proper value. This is also defined when building
in C++ mode.

  - `199409L`: C95
  - `199901L`: C99
  - `201112L`: C11
  - `201710L`: C17
  - `202311L`: C23
- `__cplusplus__`  
This is like the above macro, but is defined only when building in C++ mode.

  - `199711L`: prior to C++11
  - `201103L`: C++11
  - `201402L`: C++14
  - `201703L`: C++17
  - `202002L`: C++20
  - `202302L`: C++23
- `__FILE__`  
This expands to the name of the current file as a C string literal. This can be altered by the `#line`
preprocessor macro.
- `__LINE__`  
This expands to the line in the file at which this macro was used as an integer constant. This can
be altered by the `#line` preprocessor macro.
- `__func__` (C99)  
This is not actually a macro but a special predefined variable. This is equivalent to declaring a
`static char[]` at the very top of your function with the name of that function.
- `__DATE__`  
This expands to the compilation date of the current translation unit in *Mmm dd yyyy* format as a 
string literal, such as "May 02 2026". This always contains 11 characters.
- `__TIME__`  
This expands to the compilation time of the current translation unit in *hh:mm:ss* format as a
string literal, such as "15:02:46". This always contains 8 characters.
- `__STDC_HOSTED__` (C99; C++11)  
This is 1 if the compiler target is in a hosted enironment with access to the entire C standard library
or 0 if not.  In practice, this is 0 only if you use the `-ffreestanding` option to restrict what standard
library facilities are available. MchpClang does NOT use that option by default, giving you access
to the full C library.
- `__STDC_NO_VLA__` (C11)  
This is defined only if the compiler does not provide support for variable length arrays. VLAs were
added in C99 even though this macros was added later.
- `__STDC_NO_THREADS__` (C11)  
This is defined only if the optional threading support is not provided by the compiler. Threading
support was added in C11 in the `threads.h` header.
- `__STDC_NO_ATOMICS__` (C11)  
This is defined only if the optional atomics support is not provided by the compiler. Atomics were
added in C11 with the `_Atomic` type modifier keyboard and `stdatomic.h` header.
- `__STDC_NO_COMPLEX__` (C11)
This is defined only if the optional complex number support is not provided by the compiler. Complex
numbers were added in C99 with the `_Complex` type modifier keyword and the `complex.h` header even
though this macro was added later.
- `__STDC_IEC_559_COMPLEX__` (C99; deprecated for C23) and `__STDC_IEC_60559_COMPLEX__` (C23)  
These are used to indicate support for the `_Imaginary` type modifier keyword for imaginary numbers.
If the second macro is defined, then the compiler provides support for imaginary numbers. If the
first macro is defined, then support depends on the C language standard. For C99, you can check if
the `_Imaginary_I` macro is defined. For C11 and newer, checking for the first macro is sufficient.
- `__STDC_UTF_16__`  (C11; mandatory in C23)
This is defined only if the `char16_t` type uses UTF-16 encoding, which C23 made mandatory. Clang
always defines this in all C and C++ modes.
- `__STDC_UTF_32__`  (C11; mandatory in C23)
This is defined only if the `char32_t` type uses UTF-32 encoding, which C23 made mandatory. Clang
always defines this in all C and C++ modes.
- `__STDC_EMBED_NOT_FOUND__`, `__STDC_EMBED_FOUND__`, and `__STDC_EMBED_EMPTY__` (C23; C++26)  
These are used with the binary resource inclusion feature added to the preprocessor in C23 and C++26.
The preprocessor directive `__has_embed()` will return one of these to indicate if the given resource
can be embedded using the `#embed` directive. This feature is not documented here because the Clang
and GCC docs will be able to better explain it, so check those out for more information.
- `__STDCPP_THREADS__` (C++11)  
This is defind only if the program can have more than one thread. Clang will define this by default
because it normally assumes you are using POSIX threads. You can change to single-threaded mode by
with the `-mthread-model single` option. An initial look at the LLVM sources suggets that the thread
model does affect the sorts of optimizations LLVM can do, but leaving this alone should not cause
problems for a single-threaded baremetal app.
- `__STDCPP_DEFAULT_NEW_ALIGNMENT__` (C++17)  
This expands to a `size_t` literal that indicates the default alignment in bytes used with `operator new`.
If you needed different alignment, you would use a version that takes an alignment parameter such as
`operator new (std::size_t count, std::align_val_t)`.


## C++20 Feature Testing Macros


## Compiler Macros
These are macros provided by the compiler to give you information about itself or the compilation
process. Macros specific to your device or architecture are converted later.

- `__ASSEMBLER__`  
This is defined and set to 1 when the toolchain is preprocessing an assembly language file.


## Device Information Macros
TODO

## Device Feature Macros
These are macros that are used to access internal device features like peripheral registers or
interrupts. This is a big topic all on its own and so has its own chapter [here](using_device_features.md).

## ARM Architecture Macros
TODO


## Other Useful Macros
Things like the _BASE and _ADDR macros in the header files. Also mention the ID macros for things
like GCLK, PAC, and DMA.




## Additional Info
Here are a few places you can look if you want to find more built-in macros or functions that were
not described here.

```{todo}
Move this section to an "External Resources" chapter that also includes links to LLVM and Clang docs.
This new chapter can come before this one.
```

### CMSIS
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

### ACLE
Clang provides support for the Arm C Language Extensions, or ACLE. These are extensions that are
built into the compiler to provide things like predefined macros to provide device info, more builtin
functions to access device features beyond what CMSIS provides, and even new attributes (think the
GNU `__attribute__` keyword or the C++11 `[[attr]]` syntax).

You can find more information on the Arm developer site at <https://developer.arm.com/Architectures/Arm%20C%20Language%20Extensions>.
There are separate specification documents for base ACLE support, SIMD operations for MCUs and MPUs,
and security extensions.

### Clang Documentation
You can find more information on macros built into Clang by referring to the "Clang Language Extensions"
document. You can find that on the web at <https://clang.llvm.org/docs/LanguageExtensions.html> or
in the mchpClang install location [here](llvm:clang/html/LanguageExtensions.html).

Many macros and functions supported by Clang are also supoprted by GCC, so you can check out GCC
docs too for even more info.

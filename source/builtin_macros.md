% Copyright (c) 2026, Jesse DeGuire
% All rights reserved.
% Licensed using a BSD 3-clause license, see LICENSE at the root of this project.
% Find this project on GitHub at https://github.com/jdeguire/mchpclang_docs.

# Builtin Macros

This chapter describes some predefined preprocessor macros you may find useful. This is nowhere near
an exhaustive list so you will want to consult the [Clang documentation](llvm:clang/html/LanguageExtensions.html)
and C++ standards for more info. The site <https://www.cppreference.com> is a good place to look.

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


## C++ Feature Test Macros
C++20 introduced *a lot* of macros you can use to check for C++ language or library features that
have been added in C++11 or newer. There are too many to list here, so you will want to check out
the list online. You can look on isocpp.org [here](https://isocpp.org/std/standing-documents/sd-6-sg10-feature-test-recommendations#table-of-feature-test-macros)
or on cppreference.com [here](https://en.cppreference.com/cpp/feature_test).

Notice that you can also use `__has_cpp_attribute(attr)` to check if the compiler supports a particular
attribute. C++11 introduced a standardized attribute syntax `[[attr]]` that is like the GNU
`__attribute__((attr))` syntax. Clang supports both attribute forms. [Here](llvm:clang/html/AttributeReference.html)
is a list of all of the attributes supported by Clang.


## Compiler Macros
These are macros provided by the compiler to give you information about itself or the compilation
process. Macros specific to your device or architecture are convered later.

- `__llvm__`  
This ia always defined by Clang.
- `__clang__`  
This is always defined by Clang, so use this to check if you are compiling with Clang. Note that
Clang also defines `__GNUC__`, which is used by GCC.
- `__clang_major__`  
This is an integer indicating the major version number of Clang. For example, this would be 23 in
LLVM/Clang 23.1.4.
- `__clang_minor__`  
This is an integer indicating the minor version number of Clang. For example, this would be 1 in
LLVM/Clang 23.1.4.
- `__clang_patchlevel__`  
This is an integer indicating the patch level of Clang. For example, this would be 4 in LLVM/Clang
23.1.4.
- `__clang_version__`  
This is defined to a string literal containing Clang's vesion number followed by the Git tag or
commit number.
- `__ASSEMBLER__`  
This is defined and set to 1 when the toolchain is preprocessing an assembly language file.
- `__BASE_FILE__`  
This contains a string containing the name of the main input file passed to Clang.
- `__BYTE_ORDER__`  
This is defined to either `__ORDER_LITTLE_ENDIAN__`, `__ORDER_BIG_ENDIAN__`, or `__ORDER_PDP_ENDIAN__`
to indicate the memory order of multibyte words. Clang builds ARM in little-endian mode by default.
PDP endian is when bytes in 16-bit words are little-endian but words in 32-bit double-words are 
big-endian.
- `__BIG_ENDIAN__` or `__LITTLE_ENDIAN__`  
These are shorthand for checking if `__BYTE_ORDER__` is little- or big-endian.
- `__COUNTER__`  
This starts at 0 and is incremented with each use. This has the limit of a 32-bit signed integer.
- `__EXCEPTIONS`  
This is defined whenever you are building a C++ file with exceptions enabled. They are enabled by
default in mchpClang. This macro is one of the few that does not end in double underscores.
- `__FILE_NAME__`  
This is like `__FILE__` but contains only the file name rather than the whole file path.
- `__GNUC__`  
This is defined for compatibility with code expecting to be built with GCC, so this will tell you if
your compiler is either GCC or Clang. If you want to check for one of them exclusively, then check
if `__clang__` is also defined.
- `__GNUG__`  
This is defined for compatibility with `g++`, the GCC C++ frontend.
- `__INCLUDE_LEVEL__`  
Defined as the include depth of the current file, with the main file being 0.
- `__OPTIMIZE__`  
This is defined if compiler optimizations are enabled at level 1 or higher.
- `__OPTIMIZE_SIZE__`  
This is defined if the compiler optimization level is one of the size levels--that is, either `-Os`
or `-Oz`.
- `__TIMESTAMP__`  
This is the date and time of the last modification of the source file.
- `__USING_SJLJ_EXCEPTIONS__`  
This is defined in addition to `__EXCEPTIONS` whenever the old setjmp-longjmp exception handling
model is used. Clang by defualt uses the DWARF "zero-cost" exception model and you can enable the
old model with the `-fsjlj-exceptions` option. Note that this has not been tested, however, and may
cause issues.
- `__clang_literal_encoding__`  
This expands to a narrow string literal (i.e. `char *` instead of `wchar_t *`) indicating the encoding
of narrow string literals. This is usually "UTF-8".
- `__clang_wide_literal_encoding__`  
This expands to a narrow string literal (i.e. `char *` instead of `wchar_t *`) indicating the encoding
of wide string literals. This is usually "UTF-16" or "UTF-32", depending on your system and whether
you are using either the `-fshort-char` or `-fno-short-char` options.

The device configuration file you provide to Clang using the `--config` option also contains a few
version macros for mchpClang that are analogous to the Clang version macros described above. These 
are `__mchp_clang__`, `__mchp_clang_major__`, `__mchp_clang_minor__`, `__mchp_clang_patchlevel__`,
and `__mchp_clang_version__`.


## Device Information Macros
These are macros you can use to get information about your particular device, such as its name, CPU
type, and pin count. These are not defined by the compiler but rather by the device config you set
using the `--config` compiler option. You can have a look in those files if you want to see what is
available for you to use.

Macros are provided based on the name of your device. They will use the name of the config file you
passed to the compiler, but in upper-case. There is a form surrounded by double underscores and a
form that only starts with two underscores. For example, `__PIC32CZ8110CA80208` and `__PIC32CZ8110CA80208__`
are defined for PIC32CZ8110CA80208 and `__SAME54P20A` and `__SAME54P20A__` are defined for the SAME54P20A.
For devices that start with "PIC32", extra macros are defined that omit the "PIC" portion such as
`__32CZ8110CA80208` and `__32CZ8110CA80208__`. For parts that start with "SAM" or "ATSAM", extra
macros are defined that add the "AT" to the start such as `__ATSAME54P20A` and `__ATSAME54P20A__`.
These are available to add a bit of compatibility with Microchip's toolchains.

Addtional macros are avaiable that apply to a device family, such as "PIC32C", "PIC32CZ", "SAMD", 
"SAM9", and so on. Like the device name macros, these are available surrounded in double underscores
and starting with double underscores. Some examples are `__PIC32C__`/`__PIC32C`, `__PIC32CX__`/`__PIC32CX`,
and `__SAME`/`__SAME__`. Some devices have additional macros that are more "granular" that these
family macros. These are specifc to the device family, so you might want to check the config file
for what is available. Some examples include `__PIC32CZCA80__`/`__PIC32CZCA80`, `__SAM9X7__`/`__SAM9X7`,
and `__SAMD21__`/`__SAMD21`.

Finally, the config files provide macros to indicate device features you can use in your code. Some
of these macros are like the `__PIC32_<feature>` macros you will find in Microchip toolchains, but
these start with `__DEVICE_` instead.

- `__PIC32` and `__PIC32__`  
These are always defined in the config files to provide some compatibility with code that is excepting
Microchip's XC32 toolchain. These are defined even for SAM devices.
- `__DEVICE_NAME`  
This is a string literal containing the device's name in all upper-case.
- `__DEVICE_PIN_COUNT`  
This is an integer literal indicating the number of usable pins the device has. Devices that are
available in mutliple pacakge types will have the same number of usable pins. Extra pins that exist
only as part of a specific package are usually not used or grounded.
- `__DEVICE_CPU_NAME`  
This is a string literal containg the name of the CPU in lower-case. This is what would get passed
to the compiler's `-mtune` option. Some examples are `"cortex-m0plus"`, `"cortex-m7"`, and `amr926ej-s`.
- `__DEVICE_FPU_NAME`  
This is a string literal containing the name of the FPU in lower-case or `"none"` if the device does
not have an FPU. This is what would get passed to the compiler's `-mfpu` option. Some examples are
`"neon-vfpv4"`, `"fpv5-d16"`, or `"fpv5-sp-d16"`.
- `__DEVICE_ARCH`  
This is a string literal containing the name of the CPU architecture in lower-case. This is what
would get passed to the compiler's `-march` option. Some examples are `"armv7em"`, `"armv5te"`, and
`"armv8m.main"`.
- `__DEVICE_HAS_L1CACHE`  
Defined if the device has an L1 data or instruction cache integrated into the CPU core. This may not
be defined for devices with a separate cache peripheral, such as the CMCC peripheral.
- `__DEVICE_HAS_L1DCACHE`
Defined if the device has an L1 data cache integrated into the CPU core.
- `__DEVICE_HAS_L1ICACHE`
Defined if the device has an L1 instruction cache integrated into the CPU core.
- `__DEVICE_HAS_FPU64`  
This is defined if the device an FPU with support for 64-bit floating point operations.
- `__DEVICE_HAS_FPU32`  
This is defined if the device an FPU with support for 32-bit floating point operations.
- `__DEVICE_HAS_FPU16`  
This is defined if the device an FPU with support for 16-bit floating point operations. This is not
defined for FPUs whose only 16-bit operations are conversion operations. In other words, this would
be defined only for FPUs with `fullfp16` in their name.
- `__DEVICE_HAS_SIMD`  
This is defined if the device supports a SIMD extension such as NEON or MVE.
- `__DEVICE_HAS_NEON`  
This is defined if the device supports the NEON SIMD extension.
- `__DEVICE_HAS_MVE`  
This is defined if the device supports the M-profile Vector Extensions.
- `__DEVICE_HAS_CMSE`  
This is defined if the device supports the Cortex-M Security Extensions. The config file will supply
the `-mcmse` compiler option for devices that support CMSE.


## Device Feature Macros
These are macros that are used to access internal device features like peripheral registers or
interrupts. This is a big topic all on its own and so has its own chapter [here](using_device_features.md).


## ARM Architecture Macros
Clang supports the ARM C Language Extensions, which--among other things--provides a list of macros
that provide ARM architecture information about your device. Some of these are surrounded in double
underscores and some omit the trailing underscores. I don't know why they are inconsistent.

- `__arm` and `__arm__`  
These are defined when the build target is a 32-bit ARM device. This is defined in both ARM and Thumb
instruction modes.
- `__aarch64__`  
This is defined when the build target is a 64-bit ARM device.
- `__ARMEL__` or `__ARMEB__`  
One of these will be defined to indicate little-endian or big-endian mode, respectively.
- `__ARM_ARCH`  
This is defined as an integer indicating the ARM ISA version. For example, an ARM926 would set this
to 5 (ARMv5TE), a Cortex®-A5 or M4 would set this to 7 (ARMv7A and ARMv7EM, respectively), and a 
Cortex-M33 would set this to 8 (ARMv8M.main).
- `__ARM_ARCH_<ISA>__`  
A macro is defined to indicate the ARM ISA version by substituing `<ISA>` for the ISA version in all
caps. Here are a few examples.

  - `__ARM_ARCH_5TE__`
  - `__ARM_ARCH_6M__`
  - `__ARM_ARCH_7A__`
  - `__ARM_ARCH_7M__`
  - `__ARM_ARCH_7EM__`
  - `__ARM_ARCH_8M_BASE__`
  - `__ARM_ARCH_8M_MAIN__`
  - `__ARM_ARCH_8_1M_MAIN__`
- `__ARM_ARCH_ISA_ARM`  
This indicates that the device supports the traditional 32-bit ARM instruction set. This is set even
if the current compilation is using one of the Thumb ISA modes.
- `__ARM_ARCH_ISA_THUMB`  
This indcates that the device supports the compressed Thumb instruction set. This is set even if the
current compilation is using the 32-bit ISA. This will be 1 or 2 to indicate the Thumb version supported
by the device.
- `__ARM_ARCH_PROFILE`  
This macro is a single character indicating the architecture profile of a device: 'A' for application
processors (Cortex-A), 'R' for realtime processors (Cortex-R), or 'M' for microcontrollers (Cortex-M).
This is not defined for devices that predate the Cortex branding.
- `__ARM_VFPV<n>__` or `__ARM_FPV5__`  
These are defined by devices that have a hardware floating-point unit. In the first form, `<n>` can
be 2, 3, or 4 to indicate support for v2, v3, and v4 FPU instructions. The second form is used only
for the v5 FPU. Multiple of these can be defined. For example, a device that uese the v5 FPU will
define all 4 of these macros.
- `__ARM_FP`  
This is defined only if the device has an FPU and the current compilation is set to use it (i.e. the
`-mhard-float` flag is used). The device config files enable FPU instructions for all devices that
have an FPU. This is a bit field indicating the hardware floating-point precision level supported
by the FPU. Bit 0 is not used. Bit 1 indicates half-precision support. Bit 2 indicates single-precision
support. Bit 3 indicates double-precision support. Note that these do not indicate *how much* support
the FPU has for a precision level, only that it has some instruction to handle it. For example, the
FPv5 FPU used on Cortex-M7 devices sets this to 0x0E to indicate support for all three precisions;
however, the FPU has only minimal support for half-precision. It can only convert to and from
half-precision, but cannot do any other operations with them.
- `__ARM_NEON`  
This is defined for devices that support the NEON vector extensions.
- `__VFP_FP__`  
This is always defined even for devices that do not support an FPU, so do NOT use this to check for
FPU support. Use `__ARM_FP` or one of the other FPU macros above for that. This macro is a weird
legacy that relates to an old ARM-specific floating-point format.
- `__thumb__` and `__thumb2__`  
These are defined only when compiling in Thumb mode and indicate which Thumb ISA version is supported.
If `__thumb2__` is defined, then `__thumb__` is also defined.

In addition to these macros, ACLE also defines a bunch of macros starting with `__ARM_FEATURE` to
indicate support for particular ISA features on a device. For example, `__ARM_FEATURE_CLZ` is defined
for devices that support the `CLZ` (Count Leading Zeroes) instruction and `__ARM_FEATURE_DSP` is
defined for devices that support DSP instructions. You can do a search online or look up the ACLE
documentation for a list of these macros.


## Other Useful Macros
The device-specific header files provide some macros to describe the device's memory regions and
peripheral. The headers are located in the toolchain install location under the `arm/include/proc`
subdirectory.

The memory regions are specific to the device and so you will want to have a look in the header to see
what is available. The macros ending in `_BASE` and `_ADDR` give the starting address of the memory
region. The macros ending in `_SIZE` give the size of the region in bytes. The macros ending in
`_PAGESIZE` give the page size of the region in bytes; this is generally most applicable to flash
regions because this is the smallest number of bytes that can be erased for reprogramming.

Similar `_BASE` macros are available for all of the peripheral instances on the device. These are
described [here](./using_device_features.md#base-address-macros).

The header files also provide lots of macros to provide info about the peripheral instances on the
devices. Of particular note are ones with `_ID` in the name. Some peripherals need to refer to other
peripherals by an ID value. Examples of this are DMA controllers that can trigger on events from
other peripherals or peripherals that distribute clocks around the device. Usually, peripherals that
need to access other peripherals will have a table in their datasheet section of ID values of the
peripherals they can access. Additionally, some datasheets will contain a "Perpiheral Dependencies"
section for each peripheral with a table listing the IDs of all peripheral instances.

title: Floating point operations on ARM processor
date: 2014-04-25 21:32:34
tags: embedded
---

This article is a brief introduction of different implementations of floating point operation on an ARM processor.

### Early ages

There's no coprocessor for floating point operations on ARM, those operations are done by CPU, means of which is called `Float Math Emulation`. Normally each float point operation costs thousands of cycles, which is very inefficient. 

### Soft-float

This is for those situations where the CPU does not contain a `floating point unit(FPU)`, which is a built-in coprocessor on CPU to deal with floating point operations. 

There are 2 different ways to do floating point operations without FPU: 

1. The kernel will trap all the opcodes related to the FPU and then perform the calculations itself [1]. This is called `NWFPE` (NetWinder Floating Point Emulator), however, this has now been removed from ARM kernel [2]. As I mentioned before, this is the old fashioned way of implementation, which involves raising of invalid instruction exceptions each time the situation occurs [3]. And this can be very expensive for computation. 

2. All floating point operations are translated to specific inline function calls by compiler. This is done by passing flag `-mfloat-abi=soft` when using `gcc`, and gcc will use built in software(library) to emulate.

### Hard-float

Nowadays, many chips has hardware `FPU` support to accelerate fp operations, for ARM familis, this unit is often called `VFP` (Vector Floating-Point coprocessor). VFP is a fully IEEE-754 compatible floating point unit. But later a much more powerful NEON Advanced SIMD unit was introduced and is suggested instead of VFP by Architecture Reference Manual.

#### VFP
One of the nice feature VFP provides is that it supports single and double-precision arithmetic on vector-vector, vector-scalar, and scalar-scalar data sets where vectors can consist of up to 8 single-precision, or 4 double-precision elements [4]. The "vector mode" instructions of VFP is actually sequential, so the speed up is very limited [8]. There's one thing worth mentioning, that the "vector mode" of VFP is actually deprecated, replaced shortly after its introduce, with the much more powerful NEON Advanced SIMD unit [9]. 

To use VFP, one need to pass flag `-mfloat-abi=softfp` or `-mfloat-abi=hard` to gcc when compiling.

##### Difference between `softfp` and `hard` mode
- For `softfp` mode:
Using general integer registers to pass values (like the `soft` mode). It also support linking to `soft` mode compiled binaris. 

- For `hard` mode:
Using floating point registers on FPU to pass values. Doesn't support linking to `soft` mode binaris. All codes must be compiled in `hard` mode. It saves up to 20 cycles when calling a function with fp arguments, therefore faster than `softfp` mode [6].

Nowadays, ARM Linux is set to `hard` mode by default [3] while Debian Armel set `softfp` as default [7]. And these two modes are not compatible in one application due to different value passing conventions. 

Besides, `-msoft-float` equals to `-mfloat-abi=soft`, so as `-mhard-float` to `-mfloat-abi=hard` [8].

#### New era, with Neon

The Advanced SIMD extension (aka NEON or "MPE" Media Processing Engine) is a combined 64- and 128-bit SIMD instruction set that provides standardized acceleration for media and signal processing applications. NEON is included in all Cortex-A8 devices but is optional in Cortex-A9 devices [9]. 

Neon shares the same floating point registers with VFP, and thanks to floating point pipeline technology that Neon supports, it is a lot faster than VFP, but the draw back is that the NEON floating point pipeline is not entirely IEEE-754 compliant[10]. 

Following is quoted from Peter on [StackOverflow](http://stackoverflow.com/questions/4097034/arm-cortex-a8-whats-the-difference-between-vfp-and-neon). He's got a good introduction on this.

> Because it is not IEEE-754 compliant, a compiler cannot generate these instructions unless you tell the compiler that you are not interested in full compliance. This can be done in several ways.
> 
> 1. Using an intrinsic function to force NEON usage, for example see the [GCC Neon Intrinsic Function List](http://gcc.gnu.org/onlinedocs/gcc/ARM-NEON-Intrinsics.html).
> 
> 2. Ask the compiler, very nicely. Even newer GCC versions with `-mfpu=neon` will not generate floating point NEON instructions unless you also specify `-funsafe-math-optimizations`.

### Conclusion

ARM chips is not specifically designed for floating point operations, but with the help of coprocessors, the speed of these expensive operations can be greatly improved.



### Reference

- [1] http://www.nslu2-linux.org/wiki/FAQ/SoftHardFloatCompiler
- [2] http://lwn.net/Articles/546840/
- [3] http://linux-7110.sourceforge.net/howtos/netbook_new/x1114.htm
- [4] http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ddi0301h/Cegdejjh.html
- [5] http://www.gurucoding.com/en/rpi_cross_compiler/diff_hardfp_softfp.php
- [6] https://community.freescale.com/thread/219966
- [7] https://wiki.debian.org/ArmEabiPort
- [8] http://gcc.gnu.org/onlinedocs/gcc-4.1.1/gcc/ARM-Options.html#ARM-Options
- [9] http://en.wikipedia.org/wiki/ARM_architecture
- [10] http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0204h/Bcfhfbga.html

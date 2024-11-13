---
layout: post
title: "Runtime detection of SIMD Capability"
date: 2024-11-13
---

When coding with SIMD, a common issue is to know what are the instructions supported by the target platform?

First, you need to distinguish between ARM and x86/64 CPUs. Since they use entirely different assemblies, we must generate different code for each of them. Thus, no runtime detection is required as no common assembly is shared.

The headache comes from x86/64 CPU, as they have many different SIMD instruction sets. The most common are SSE2, SSE3, SSSE3, SSE4.1, SSE4.2, AVX, AVX2, AVX-512, and FMA. The problem is that the newer instruction sets are not always supported by older CPUs. So, if you want to use the latest instruction set, you need to check if the CPU supports it, while also shipping the other non-SIMD assemblies as common parts of the program.

Luckily, both Intel and AMD has built-in instructions to detect such capabilities. This is done by checking the CPUID instruction. The details are all listed in the documents:

- AMD: Search AMD CPUID, you all get [CPUID Specification](https://www.amd.com/content/dam/amd/en/documents/archived-tech-docs/design-guides/25481.pdf)
- Intel: Search Intel CPUID, you all get [Intel 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/content/www/us/en/develop/articles/intel-sdm.html). Go for [Intel® 64 and IA-32 Architectures Software Developer's Manual Combined Volumes 2A, 2B, 2C, and 2D: Instruction Set Reference, A- Z](https://cdrdv2.intel.com/v1/dl/getContent/671110). Or for a web version [CPUID Enumeration and Architectural MSRs](https://www.intel.com/content/www/us/en/developer/articles/technical/software-security-guidance/technical-documentation/cpuid-enumeration-and-architectural-msrs.html)

Then it's a matter of using assembly appropriately. One good example is in openh264 [cpuid.asm](https://github.com/cisco/openh264/blob/master/codec/common/x86/cpuid.asm), [cpu.cpp](https://github.com/cisco/openh264/blob/master/codec/common/src/cpu.cpp), and [cpu_core.h](https://github.com/cisco/openh264/blob/master/codec/common/inc/cpu_core.h) that use a flag to return the supported SIMD instructions.

## Appendix - Python Code Snippet to Decipher the Returned Flag Value

```python
import enum


class WELS_CPU(enum.IntFlag):
    WELS_CPU_MMX      = 0x00000001  # mmx
    WELS_CPU_MMXEXT   = 0x00000002  # mmx-ext
    WELS_CPU_SSE      = 0x00000004  # sse
    WELS_CPU_SSE2     = 0x00000008  # sse 2
    WELS_CPU_SSE3     = 0x00000010  # sse 3
    WELS_CPU_SSE41    = 0x00000020  # sse 4.1
    WELS_CPU_3DNOW    = 0x00000040  # 3dnow!
    WELS_CPU_3DNOWEXT = 0x00000080  # 3dnow! ext
    WELS_CPU_ALTIVEC  = 0x00000100  # altivec
    WELS_CPU_SSSE3    = 0x00000200  # ssse3
    WELS_CPU_SSE42    = 0x00000400  # sse 4.2

    WELS_CPU_FPU   = 0x00001000  # x87-FPU on chip
    WELS_CPU_HTT   = 0x00002000  # Hyper-Threading Technology (HTT), Multi-threading enabled feature: physical processor package is capable of supporting more than one logic processor
    WELS_CPU_CMOV  = 0x00004000  # Conditional Move Instructions, also if x87-FPU is present at indicated by the CPUID.FPU feature bit, then FCOMI and FCMOV are supported
    WELS_CPU_MOVBE = 0x00008000  # MOVBE instruction
    WELS_CPU_AES   = 0x00010000  # AES instruction extensions
    WELS_CPU_FMA   = 0x00020000  # AVX VEX FMA instruction sets
    WELS_CPU_AVX   = 0x00000800  # Advanced Vector eXtentions
    WELS_CPU_AVX2  = 0x00040000  # AVX2

    WELS_CPU_AVX512F  = 0x00080000  # AVX512F
    WELS_CPU_AVX512CD = 0x00100000  # AVX512CD
    WELS_CPU_AVX512DQ = 0x00200000  # AVX512DQ
    WELS_CPU_AVX512BW = 0x00400000  # AVX512BW
    WELS_CPU_AVX512VL = 0x00800000  # AVX512VL

    WELS_CPU_CACHELINE_16  = 0x10000000  # CacheLine Size 16
    WELS_CPU_CACHELINE_32  = 0x20000000  # CacheLine Size 32
    WELS_CPU_CACHELINE_64  = 0x40000000  # CacheLine Size 64
    WELS_CPU_CACHELINE_128 = 0x80000000  # CacheLine Size 128

    WELS_CPU_ARMv7 = 0x000001   # ARMv7
    WELS_CPU_VFPv3 = 0x000002   # VFPv3
    WELS_CPU_NEON  = 0x000004   # NEON


if __name__ == "__main__":
    print(WELS_CPU(input()))
```

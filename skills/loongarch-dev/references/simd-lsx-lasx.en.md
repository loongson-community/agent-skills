<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

# LSX and LASX SIMD Intrinsics

## Overview

LoongArch provides SIMD/vector extensions:

| Extension | Register Width | Vectors | Notes |
|-----------|----------------|---------|-------|
| **LSX** (Loongson SIMD eXtension) | 128-bit | 16 × 128-bit | Baseline SIMD, widely supported |
| **LASX** (Loongson Advanced SIMD eXtension) | 256-bit | 16 × 256-bit | Extended SIMD, newer chips only |

## LSX (128-bit SIMD)

### Key Features
- 16 × 128-bit vector registers ($v0-$v15)
- Operations: integer, floating-point, bitwise, shuffle, compare
- Data types: i8/i16/i32/i64, f32/f64

### Header
```c
#include <lsxintrin.h>
```

### Basic Types
```c
typedef int32_t __v4i32 __attribute__((__vector_size__(16)));
typedef float   __v4f32 __attribute__((__vector_size__(16)));
```

### Common Operations

#### Load/Store
```c
__m128i v = __lsx_vld(const void *addr, int offset);  // Load 128 bits
__lsx_vst(__m128i v, void *addr, int offset);         // Store 128 bits
```

#### Arithmetic
```c
__m128i sum = __lsx_vadd_w(__m128i a, __m128i b);     // Vector add (32-bit)
__m128i diff = __lsx_vsub_w(__m128i a, __m128i b);    // Vector subtract
__m128i prod = __lsx_vmul_w(__m128i a, __m128i b);    // Vector multiply
```

#### Floating Point
```c
__m128 fadd = __lsx_vfadd_s(__m128 a, __m128 b);      // Float add
__m128 fmul = __lsx_vfmul_s(__m128 a, __m128 b);      // Float multiply
```

#### Shuffles
```c
__m128i shuf = __lsx_vshuf_b(__m128i ctrl, __m128i a, __m128i b);
```

## LASX (256-bit SIMD)

### Key Features
- 16 × 256-bit vector registers ($x0-$x15)
- Superset of LSX operations on 256-bit vectors
- Available on newer LoongArch CPUs (3A5000+, 3C6000, etc.)

### Header
```c
#include <lasxintrin.h>
```

### Common Operations

#### Load/Store
```c
__m256i v = __lasx_xvld(const void *addr, int offset);  // Load 256 bits
__lasx_xvst(__m256i v, void *addr, int offset);         // Store 256 bits
```

#### Arithmetic
```c
__m256i sum = __lasx_xvadd_w(__m256i a, __m256i b);     // Vector add (32-bit)
__m256i prod = __lasx_xvmul_w(__m256i a, __m256i b);    // Vector multiply
```

## CPU Detection

### Compile-time
```c
#ifdef __loongarch_sx
    // LSX is available at compile time
#endif

#ifdef __loongarch_asx
    // LASX is available at compile time
#endif
```

### Runtime Detection
```c
#include <sys/auxv.h>
#include <asm/hwcap.h>

unsigned long hwcap = getauxval(AT_HWCAP);
if (hwcap & HWCAP_LOONGARCH_LSX) {
    // LSX available
}
if (hwcap & HWCAP_LOONGARCH_LASX) {
    // LASX available
}
```

## Porting from Other SIMD

### x86 SSE/AVX → LSX/LASX

| x86 Intrinsic | LSX Equivalent | Notes |
|---------------|----------------|-------|
| `_mm_loadu_si128` | `__lsx_vld` | LSX uses offset parameter |
| `_mm_storeu_si128` | `__lsx_vst` | LSX uses offset parameter |
| `_mm_add_epi32` | `__lsx_vadd_w` | 'w' = word (32-bit) |
| `_mm_mullo_epi32` | `__lsx_vmul_w` | LSX has direct multiply |
| `_mm_set1_epi32` | `__lsx_vreplgr2vr_w` | Replicate scalar to vector |
| `_mm_shuffle_epi8` | `__lsx_vshuf_b` | Byte shuffle |

### ARM NEON → LSX/LASX

| NEON Intrinsic | LSX Equivalent |
|----------------|----------------|
| `vld1q_s32` | `__lsx_vld` + cast |
| `vst1q_s32` | `__lsx_vst` |
| `vaddq_s32` | `__lsx_vadd_w` |
| `vmulq_s32` | `__lsx_vmul_w` |
| `vdupq_n_s32` | `__lsx_vreplgr2vr_w` |

## Best Practices

1. **Always check CPU support at runtime** when distributing binaries
2. **Use function multi-versioning** for optimal performance across CPU generations
3. **Prefer LASX when available** — 2× throughput for many operations
4. **Align memory accesses** when possible (16-byte for LSX, 32-byte for LASX)
5. **Use intrinsics over inline assembly** — compiler can optimize better

## References

- **Unofficial Intrinsics Guide**: https://github.com/jiegec/unofficial-loongarch-intrinsics-guide
- **GCC LSX intrinsics**: https://gcc.gnu.org/onlinedocs/gcc/LoongArch-LSX-Built-in-Functions.html
- **GCC LASX intrinsics**: https://gcc.gnu.org/onlinedocs/gcc/LoongArch-LASX-Built-in-Functions.html

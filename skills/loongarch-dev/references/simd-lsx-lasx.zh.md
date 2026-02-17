<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

# LSX 和 LASX SIMD 内建函数

## 概述

LoongArch 提供 SIMD/向量扩展：

| 扩展 | 寄存器宽度 | 向量数 | 说明 |
|-----------|----------------|---------|-------|
| **LSX**（Loongson SIMD eXtension） | 128 位 | 16 × 128 位 | 基线 SIMD，广泛支持 |
| **LASX**（Loongson Advanced SIMD eXtension） | 256 位 | 16 × 256 位 | 扩展 SIMD，仅新芯片支持 |

## LSX（128 位 SIMD）

### 主要特性
- 16 个 128 位向量寄存器（$v0-$v15）
- 操作：整数、浮点、位运算、混洗、比较
- 数据类型：i8/i16/i32/i64、f32/f64

### 头文件
```c
#include <lsxintrin.h>
```

### 基本类型
```c
typedef int32_t __v4i32 __attribute__((__vector_size__(16)));
typedef float   __v4f32 __attribute__((__vector_size__(16)));
```

### 常用操作

#### 加载/存储
```c
__m128i v = __lsx_vld(const void *addr, int offset);  // 加载 128 位
__lsx_vst(__m128i v, void *addr, int offset);         // 存储 128 位
```

#### 算术运算
```c
__m128i sum = __lsx_vadd_w(__m128i a, __m128i b);     // 向量加法（32 位）
__m128i diff = __lsx_vsub_w(__m128i a, __m128i b);    // 向量减法
__m128i prod = __lsx_vmul_w(__m128i a, __m128i b);    // 向量乘法
```

#### 浮点运算
```c
__m128 fadd = __lsx_vfadd_s(__m128 a, __m128 b);      // 浮点加法
__m128 fmul = __lsx_vfmul_s(__m128 a, __m128 b);      // 浮点乘法
```

#### 混洗
```c
__m128i shuf = __lsx_vshuf_b(__m128i ctrl, __m128i a, __m128i b);
```

## LASX（256 位 SIMD）

### 主要特性
- 16 个 256 位向量寄存器（$x0-$x15）
- LSX 操作的超集，支持 256 位向量
- 仅在较新的 LoongArch CPU 上可用（3A5000+、3C6000 等）

### 头文件
```c
#include <lasxintrin.h>
```

### 常用操作

#### 加载/存储
```c
__m256i v = __lasx_xvld(const void *addr, int offset);  // 加载 256 位
__lasx_xvst(__m256i v, void *addr, int offset);         // 存储 256 位
```

#### 算术运算
```c
__m256i sum = __lasx_xvadd_w(__m256i a, __m256i b);     // 向量加法（32 位）
__m256i prod = __lasx_xvmul_w(__m256i a, __m256i b);    // 向量乘法
```

## CPU 检测

### 编译时
```c
#ifdef __loongarch_sx
    // 编译时 LSX 可用
#endif

#ifdef __loongarch_asx
    // 编译时 LASX 可用
#endif
```

### 运行时检测
```c
#include <sys/auxv.h>
#include <asm/hwcap.h>

unsigned long hwcap = getauxval(AT_HWCAP);
if (hwcap & HWCAP_LOONGARCH_LSX) {
    // LSX 可用
}
if (hwcap & HWCAP_LOONGARCH_LASX) {
    // LASX 可用
}
```

## 从其他 SIMD 移植

### x86 SSE/AVX → LSX/LASX

| x86 内建函数 | LSX 等效函数 | 说明 |
|---------------|----------------|-------|
| `_mm_loadu_si128` | `__lsx_vld` | LSX 使用偏移参数 |
| `_mm_storeu_si128` | `__lsx_vst` | LSX 使用偏移参数 |
| `_mm_add_epi32` | `__lsx_vadd_w` | 'w' = word（32 位） |
| `_mm_mullo_epi32` | `__lsx_vmul_w` | LSX 有直接乘法 |
| `_mm_set1_epi32` | `__lsx_vreplgr2vr_w` | 将标量复制到向量 |
| `_mm_shuffle_epi8` | `__lsx_vshuf_b` | 字节混洗 |

### ARM NEON → LSX/LASX

| NEON 内建函数 | LSX 等效函数 |
|----------------|----------------|
| `vld1q_s32` | `__lsx_vld` + 类型转换 |
| `vst1q_s32` | `__lsx_vst` |
| `vaddq_s32` | `__lsx_vadd_w` |
| `vmulq_s32` | `__lsx_vmul_w` |
| `vdupq_n_s32` | `__lsx_vreplgr2vr_w` |

## 最佳实践

1. **在分发二进制文件时，应始终在运行时检查 CPU 支持**
2. **使用函数多版本化** — 在跨 CPU 代际时获得最佳性能
3. **在 LASX 可用时应优先使用** — 许多操作的吞吐量可提高 2 倍
4. **尽可能对齐内存访问**（LSX 16 字节，LASX 32 字节）
5. **优先使用内建函数而非内联汇编** — 以便编译器能更好地进行优化

## 参考

- **非官方内建函数指南**：https://github.com/jiegec/unofficial-loongarch-intrinsics-guide
- **GCC LSX 内建函数**：https://gcc.gnu.org/onlinedocs/gcc/LoongArch-LSX-Built-in-Functions.html
- **GCC LASX 内建函数**：https://gcc.gnu.org/onlinedocs/gcc/LoongArch-LASX-Built-in-Functions.html

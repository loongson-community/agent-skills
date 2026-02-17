<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

# LoongArch Porting Guide

## Quick Start Checklist

When porting software to LoongArch:

- [ ] Identify target ABI (old world vs new world)
- [ ] Check compiler support (GCC 12+, LLVM 16+ for new world)
- [ ] Review architecture-specific code (inline asm, intrinsics)
- [ ] Check for endianness assumptions (LoongArch is little-endian)
- [ ] Verify syscall usage (avoid hardcoded numbers)
- [ ] Test SIMD code paths (LSX/LASX alternatives)
- [ ] Check signal handling code
- [ ] Review memory alignment requirements

## Endianness

LoongArch is **little-endian** (like x86_64, unlike some legacy MIPS/SPARC).

Most modern software handles this correctly, but watch for:
```c
// Bad: Hardcoded endianness assumption
#ifdef __x86_64__
    // little-endian code
#else
    // wrong assumption about other archs
#endif

// Good: Use standard macros
#if __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__
    // little-endian code
#else
    // big-endian code
#endif
```

## Compiler Feature Detection

### Preprocessor Macros

**Important**: LoongArch distinguishes between ISA/HW capability (GPR width) and ABI data model. These are **different** from other architectures where they may be conflated.

```c
// All LoongArch
#ifdef __loongarch__
    // LoongArch-specific code
#endif

// ABI Data Model (RECOMMENDED for most checks)
#ifdef __loongarch_lp64
    // LP64 ABI: long and pointers are 64-bit (standard for LoongArch64)
#endif

#ifdef __loongarch_ilp32
    // ILP32 ABI: int, long, and pointers are 32-bit (rare)
#endif

// ISA/HW Capability (GPR width) -- __loongarch64 is deprecated!
#if __loongarch_grlen == 64
    // Hardware has 64-bit general purpose registers
#endif

#if __loongarch_grlen == 32
    // Hardware has 32-bit general purpose registers
#endif

// Specific extensions
#ifdef __loongarch_sx    // LSX SIMD
#ifdef __loongarch_asx   // LASX SIMD  
#ifdef __loongarch_bt    // LBT binary translation
#endif
```

### Key Distinction

| Macro | Meaning | Use When |
|-------|---------|----------|
| `__loongarch_lp64` | ABI uses LP64 data model | **Almost always** — checking pointer/size_t width |
| `__loongarch_ilp32` | ABI uses ILP32 data model | Embedded 32-bit scenarios |
| `__loongarch_grlen == 64` | Hardware GPR width is 64-bit | ISA-level code generation |
| `__loongarch64` | Deprecated, same as above | **Avoid** — use `__loongarch_grlen` instead |

### Compiler Identification

```c
#if defined(__GNUC__) && !defined(__clang__)
    // GCC-specific
#elif defined(__clang__)
    // Clang-specific
#endif
```

## Inline Assembly

### Basic Syntax

LoongArch uses standard GCC extended asm:

```c
uint64_t result;
__asm__ volatile (
    "add.d %0, %1, %2"    // instruction
    : "=r" (result)       // outputs
    : "r" (a), "r" (b)    // inputs
    :                     // clobbers
);
```

### Common Instructions

| Operation | Instruction | Notes |
|-----------|-------------|-------|
| Add 64-bit | `add.d` | d = double (64-bit) |
| Add 32-bit | `add.w` | w = word (32-bit) |
| Load 64-bit | `ld.d` | |
| Store 64-bit | `st.d` | |
| Move | `move` | Register move |
| Syscall | `syscall` | Prefer libc wrappers |

### Register Names

- `$r0-$r31`: General purpose registers
- `$zero` ($r0): Hardwired zero
- `$ra` ($r1): Return address
- `$tp` ($r2): Thread pointer
- `$sp` ($r3): Stack pointer
- `$a0-$a7` ($r4-$r11): Arguments/return
- `$t0-$t8` ($r12-$r20): Temporaries
- `$s0-$s8` ($r23-$r31): Saved registers

## Common Porting Issues

### 1. Hardcoded Syscall Numbers

**Problem**:
```c
// x86_64 syscall number — breaks on LoongArch
syscall(0, ...);  // SYS_read on x86_64, wrong on LoongArch
```

**Solution**:
```c
#include <sys/syscall.h>
#include <unistd.h>

// Use standard headers
syscall(SYS_read, ...);  // Correct on all architectures
// Or better: use libc wrapper
read(fd, buf, count);
```

### 2. Architecture-Specific Memory Barriers

**Problem**:
```c
// x86-specific: mfence
__asm__ __volatile__ ("mfence" ::: "memory");
```

**Solution**:
```c
// Use standard atomics
#include <stdatomic.h>
atomic_thread_fence(memory_order_seq_cst);

// Or compiler barrier
__atomic_thread_fence(__ATOMIC_SEQ_CST);
```

### 3. Signal Context (ucontext_t)

**Problem**: Old-world LoongArch has different `ucontext_t` layout.

**Solution**: Use `ucontext.h` properly, avoid accessing `uc_mcontext` fields directly when possible. If you must, abstract behind macros:

```c
#ifdef __loongarch__
    #ifdef __loongarch_lp64
        #define GET_PC(ctx) ((ctx)->uc_mcontext.__pc)
    #endif
#endif
```

### 4. Self-Modifying Code / JIT

LoongArch requires explicit cache synchronization:

```c
// After writing code, flush caches
__builtin___clear_cache(start, end);

// Or inline asm:
__asm__ volatile (
    "dbar 0 \n\t"      // Data barrier
    "ibar 0"          // Instruction barrier
    ::: "memory"
);
```

### 5. Aligned Memory Access

LoongArch handles unaligned access but with performance penalty:

```c
// Prefer aligned allocations for SIMD
void *ptr;
posix_memalign(&ptr, 16, size);  // For LSX
posix_memalign(&ptr, 32, size);  // For LASX
```

## Build System Integration

### Autotools

```bash
# configure.ac
AC_CANONICAL_HOST
case $host_cpu in
  loongarch*)
    AC_DEFINE([ARCH_LOONGARCH], [1], [LoongArch architecture])
    ;;
esac
```

### CMake

```cmake
if(CMAKE_SYSTEM_PROCESSOR MATCHES "loongarch")
    target_compile_definitions(mytarget PRIVATE ARCH_LOONGARCH=1)
endif()

# Feature detection
check_c_source_compiles("
#include <lsxintrin.h>
int main() { __m128i v; return 0; }
" HAVE_LSX)
```

### Meson

```meson
host_cpu = host_machine.cpu_family()
if host_cpu == 'loongarch64'
    add_project_arguments('-DARCH_LOONGARCH=1', language: 'c')
endif
```

## Testing

### QEMU User Mode

Test LoongArch binaries on x86_64 host:

```bash
# Install qemu-user-loongarch64
# Debian/Ubuntu: apt install qemu-user-static

qemu-loongarch64 ./mybinary
```

### Cross-Compilation

```bash
# GCC cross-compiler
loongarch64-linux-gnu-gcc -o mybinary mysource.c

# Clang (with cross toolchain)
clang --target=loongarch64-linux-gnu -o mybinary mysource.c
```

### Native Build

On LoongArch hardware:

```bash
# Check compiler version (need GCC 12+ or LLVM 16+ for new world)
gcc --version
clang --version

# Standard build
./configure && make
# or
cmake -B build && cmake --build build
```

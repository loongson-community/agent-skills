<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

# LBT: LoongArch Binary Translation

## Overview

**LBT** (LoongArch Binary Translation) is a set of hardware features designed to accelerate binary translation from other architectures (primarily x86 and ARM) to LoongArch.

It provides:
- Specialized instructions for emulating x86/ARM behavior
- Hardware-assisted EFLAGS/condition code handling
- Memory ordering compatibility features
- Floating-point exception compatibility

## When to Use LBT

LBT is primarily used by:
- **QEMU** — for faster x86/ARM emulation
- **Box64/Box86** — for running x86/x86_64 binaries
- **Custom emulators** — for architecture translation

Most application developers do **not** need to use LBT directly unless building emulation layers.

## LBT Instruction Categories

### 1. EFLAGS Operations (x86-style condition codes)

LBT provides hardware support for x86's EFLAGS register:

```asm
# Push/pop EFLAGS-like state
lbt_push_pwd  # Save processor word (EFLAGS equivalent)
lbt_pop_pwd   # Restore processor word

# Condition code operations
lbt_add_w    # Add with x86-style flags update
lbt_sub_w    # Subtract with x86-style flags update
```

### 2. Memory Ordering Compatibility

x86 has strong memory ordering; ARM has weak ordering. LBT provides:

```asm
# Memory barrier operations for x86 compatibility
lbt_mb_x86   # x86-style full barrier
lbt_mb_arm   # ARM-style barrier
```

### 3. Floating-Point Exception Handling

Different architectures handle FP exceptions differently:

```asm
# FP exception state management
lbt_get_fp_except  # Get FP exception flags
lbt_set_fp_except  # Set FP exception flags
```

## CPU Detection

### Compile-time
```c
#ifdef __loongarch_bt
    // LBT is available at compile time
#endif
```

### Runtime Detection
```c
#include <sys/auxv.h>
#include <asm/hwcap.h>

unsigned long hwcap = getauxval(AT_HWCAP);
if (hwcap & HWCAP_LOONGARCH_LBT) {
    // LBT available
}
```

## Using LBT

### Inline Assembly Example
```c
// Example: x86-style add with flags
uint32_t x86_add(uint32_t a, uint32_t b, uint32_t *flags) {
    uint32_t result;
    uint32_t f;
    
    __asm__ volatile (
        "lbt.add.w %0, %1, %2\n\t"
        "lbt.get.pwd %3"
        : "=r" (result), "=r" (f)
        : "r" (a), "r" (b)
        : "cc"
    );
    
    *flags = f;
    return result;
}
```

### From C via Intrinsics

Some compilers provide intrinsics for LBT operations:

```c
// GCC/Clang may provide:
__builtin_loongarch_lbt_add_w(a, b);  // Add with flags
__builtin_loongarch_lbt_push_pwd();   // Save state
__builtin_loongarch_lbt_pop_pwd(val); // Restore state
```

**Note**: LBT intrinsics are less standardized than LSX/LASX. Check your compiler version.

## LBT in Practice: Box64 Example

Box64 uses LBT for fast x86_64 emulation:

```c
// Simplified example from Box64
void emulate_x86_add() {
    if (lbt_available) {
        // Use hardware LBT
        __asm__ volatile ("lbt.add.w ...");
    } else {
        // Software emulation of flags
        compute_flags_software();
    }
}
```

## Performance Considerations

1. **LBT overhead**: LBT instructions have some overhead — use only when emulating x86/ARM behavior
2. **Mixed code**: Avoid mixing LBT and non-LBT code paths frequently
3. **State management**: Save/restore LBT state around context switches

## References

- **LBT Instruction Reference**: https://github.com/jiegec/la-inst/blob/master/LBT.md
- **Box64**: https://github.com/ptitSeb/box64 (practical LBT usage)
- **QEMU LoongArch target**: https://gitlab.com/qemu-project/qemu/-/tree/master/target/loongarch

## See Also

- [abi-worlds.en.md](abi-worlds.en.md) — ABI compatibility considerations for translated binaries
- [simd-lsx-lasx.en.md](simd-lsx-lasx.en.md) — SIMD operations often used alongside translation

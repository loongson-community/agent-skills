<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

# LBT：LoongArch 二进制翻译

## 概述

**LBT**（LoongArch Binary Translation）是一组硬件特性，旨在加速从其他架构（主要是 x86 和 ARM）到 LoongArch 的二进制翻译。

它提供：
- 用于模拟 x86/ARM 行为的专用指令
- 硬件辅助的 EFLAGS/条件码处理
- 内存排序兼容性特性
- 浮点异常兼容性

## 何时使用 LBT

LBT 主要由以下场景使用：
- **QEMU** — 用于更快的 x86/ARM 模拟
- **Box64/Box86** — 用于运行 x86/x86_64 二进制文件
- **自定义模拟器** — 用于架构翻译

大多数应用程序开发者**不需要**直接使用 LBT，除非他们需要构建模拟层。

## LBT 指令类别

### 1. EFLAGS 操作（x86 风格条件码）

LBT 为 x86 的 EFLAGS 寄存器提供硬件支持：

```asm
# 压入/弹出 EFLAGS 风格状态
lbt_push_pwd  # 保存处理器字（EFLAGS 等效）
lbt_pop_pwd   # 恢复处理器字

# 条件码操作
lbt_add_w    # 带 x86 风格标志更新的加法
lbt_sub_w    # 带 x86 风格标志更新的减法
```

### 2. 内存排序兼容性

x86 具有强内存排序；ARM 具有弱排序。LBT 提供：

```asm
# 用于 x86 兼容性的内存屏障操作
lbt_mb_x86   # x86 风格完全屏障
lbt_mb_arm   # ARM 风格屏障
```

### 3. 浮点异常处理

不同架构处理 FP 异常的方式不同：

```asm
# FP 异常状态管理
lbt_get_fp_except  # 获取 FP 异常标志
lbt_set_fp_except  # 设置 FP 异常标志
```

## CPU 检测

### 编译时
```c
#ifdef __loongarch_bt
    // 编译时 LBT 可用
#endif
```

### 运行时检测
```c
#include <sys/auxv.h>
#include <asm/hwcap.h>

unsigned long hwcap = getauxval(AT_HWCAP);
if (hwcap & HWCAP_LOONGARCH_LBT) {
    // LBT 可用
}
```

## 使用 LBT

### 内联汇编示例
```c
// 示例：带标志的 x86 风格加法
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

### 通过内建函数从 C 使用

某些编译器为 LBT 操作提供内建函数：

```c
// GCC/Clang 可能提供：
__builtin_loongarch_lbt_add_w(a, b);  // 带标志的加法
__builtin_loongarch_lbt_push_pwd();   // 保存状态
__builtin_loongarch_lbt_pop_pwd(val); // 恢复状态
```

**注意**：LBT 内建函数的标准化程度不如 LSX/LASX。请检查您的编译器版本。

## 实践中的 LBT：Box64 示例

Box64 使用 LBT 进行快速 x86_64 模拟：

```c
// 来自 Box64 的简化示例
void emulate_x86_add() {
    if (lbt_available) {
        // 使用硬件 LBT
        __asm__ volatile ("lbt.add.w ...");
    } else {
        // 标志的软件模拟
        compute_flags_software();
    }
}
```

## 性能注意事项

1. **LBT 开销**：LBT 指令有一定开销——仅在需要模拟 x86/ARM 行为时才使用它们
2. **混合代码**：避免频繁混合 LBT 和非 LBT 代码路径
3. **状态管理**：在上下文切换前后保存/恢复 LBT 状态

## 参考

- **LBT 指令参考**：https://github.com/jiegec/la-inst/blob/master/LBT.md
- **Box64**：https://github.com/ptitSeb/box64（实际 LBT 使用）
- **QEMU LoongArch 目标**：https://gitlab.com/qemu-project/qemu/-/tree/master/target/loongarch

## 另请参阅

- [abi-worlds.zh.md](abi-worlds.zh.md) — 翻译二进制文件的 ABI 兼容性注意事项
- [simd-lsx-lasx.zh.md](simd-lsx-lasx.zh.md) — 通常与翻译一起使用的 SIMD 操作

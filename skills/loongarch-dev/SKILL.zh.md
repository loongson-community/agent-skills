<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

---
name: loongarch-dev
description: LoongArch 架构开发与移植指南。在处理 LoongArch（龙芯）软件移植、编写架构特定代码、处理旧世界（ABI 1.0）与新世界（ABI 2.0）之间的 ABI 兼容性、使用 LSX/LASX SIMD 指令、LBT 二进制翻译功能或解决 LoongArch 特定构建问题时使用。
---

# LoongArch 开发

## 快速参考

LoongArch 是龙芯开发的 RISC 指令集架构。主要特点：
- **64 位小端**架构
- **两种不兼容的 ABI**：旧世界（ABI 1.0）与新世界（ABI 2.0）
- **SIMD**：LSX（128 位）和 LASX（256 位）向量扩展
- **二进制翻译**：LBT，用于 x86/ARM 模拟加速

## 何时加载参考资料

### [abi-worlds.md](references/abi-worlds.md)
处理以下问题时加载：
- 二进制兼容性问题
- "旧世界"与"新世界"的困惑
- liblol 兼容层相关问题
- 发行版差异

### [simd-lsx-lasx.md](references/simd-lsx-lasx.md)
在以下情况加载：
- 从 x86（SSE/AVX）或 ARM（NEON）移植 SIMD 代码
- 编写 LSX/LASX 指令
- 运行时检测 SIMD 能力
- 优化向量化代码

### [lbt.md](references/lbt.md)
在以下情况加载：
- 处理二进制翻译（QEMU、Box64 等）
- 需要 x86 风格的 EFLAGS 模拟
- 构建模拟层

### [porting-guide.md](references/porting-guide.md)
用于：
- 通用移植清单和最佳实践
- 编译器特性检测（`__loongarch__` 宏）
- 内联汇编语法
- 常见移植问题和解决方案

### [resources.md](references/resources.md)
用于：
- 官方文档链接
- 社区资源（loongfans.cn、areweloongyet.com）
- 发行版支持状态
- 获取帮助

## 常见任务

### 将代码库移植到 LoongArch
1. 阅读 [porting-guide.md](references/porting-guide.md) 了解清单
2. 阅读 [abi-worlds.md](references/abi-worlds.md) 确定目标 ABI
3. 查看 [resources.md](references/resources.md) 了解依赖项的移植状态

### 修复 SIMD 相关的构建失败
1. 阅读 [simd-lsx-lasx.md](references/simd-lsx-lasx.md)
2. 将现有指令映射到 LSX/LASX 等效指令
3. 如需要，添加运行时 CPU 检测

### 在新世界系统上运行旧世界二进制文件
1. 阅读 [abi-worlds.md](references/abi-worlds.md) 的 liblol 章节
2. 从 https://liblol.aosc.io/ 安装 liblol

### 编写内联汇编
1. 阅读 [porting-guide.md](references/porting-guide.md) 的内联汇编章节
2. 参考 LoongArch 指令集手册
3. 有内建函数时优先使用内建函数而非内联汇编

## 架构检测

```c
// 通用 LoongArch
#ifdef __loongarch__

// ABI 数据模型（推荐——使用此宏！）
#ifdef __loongarch_lp64     // LP64：long 和指针为 64 位
#ifdef __loongarch_ilp32    // ILP32：int/long/指针为 32 位

// ISA/硬件能力（通用寄存器宽度）—— __loongarch64 已弃用！
#if __loongarch_grlen == 64  // 硬件具有 64 位通用寄存器

// 扩展
#ifdef __loongarch_sx   // LSX SIMD
#ifdef __loongarch_asx  // LASX SIMD
#ifdef __loongarch_bt   // LBT 二进制翻译
```

**注意**：LoongArch 区分 **ABI 数据模型**（`__loongarch_lp64`）和 **ISA 能力**（`__loongarch_grlen`）。对于大多数软件，应检查 ABI（`lp64`）而非硬件。详见 [porting-guide.md](references/porting-guide.md)。

## 社区

- **loongfans.cn** — 社区维基
- **areweloongyet.com** — 软件移植状态
- **github.com/loongson-community** — 社区项目

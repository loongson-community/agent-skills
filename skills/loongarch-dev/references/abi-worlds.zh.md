<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

# LoongArch ABI 世界：旧世界与新世界

## 概述

LoongArch 有**两种不兼容的 ABI**，这是移植软件时的主要困惑来源：

| 方面 | 旧世界（ABI 1.0） | 新世界（ABI 2.0） |
|------|------------------|------------------|
| **内核** | 4.x 时代补丁 | 上游 Linux 5.19+ |
| **工具链** | 龙芯的分支版本 | 上游 GCC 12+、LLVM 16+ |
| **发行版** | Loongnix、Kylin、UOS | Gentoo、Debian、AOSC 等 |
| **系统调用** | 非标准 | 标准上游 |
| **errno 位置** | 线程本地（非标准） | 标准 TLS 布局 |
| **ld.so 路径** | `/lib64/ld.so.1` | `/lib64/ld-linux-loongarch-lp64d.so.1` |
| **信号处理** | 结构体布局不同 | 标准布局 |

## 识别您所在的世界

### 检查动态链接器路径：
```bash
# 旧世界
$ readelf -l /bin/ls | grep interpreter
      [Requesting program interpreter: /lib64/ld.so.1]

# 新世界
$ readelf -l /bin/ls | grep interpreter
      [Requesting program interpreter: /lib64/ld-linux-loongarch-lp64d.so.1]
```

### 检查内核版本：
```bash
uname -r
# 旧世界：带自定义补丁的 4.x
# 新世界：5.19+ 上游内核
```

## 不兼容问题

**旧世界和新世界二进制文件无法直接在对方系统上运行**，因为：

1. **不同的系统调用号** — 内核 ABI 不匹配
2. **不同的 TLS/errno 布局** — 线程本地存储不同
3. **不同的信号帧布局** — 信号处理程序损坏
4. **不同的 ld.so** — 动态链接器不兼容

## liblol：兼容性解决方案

**libLOL**（Loongson Old-world Linux）是 AOSC 提供的开源兼容层：

- **项目**：https://liblol.aosc.io/
- **源码**：https://github.com/AOSC-Dev/liblol
- **功能**：允许旧世界二进制文件在新世界系统上运行

工作原理：
- 提供一个兼容的 ld.so，拦截系统调用
- 将旧世界系统调用转换为新世界等效调用
- 处理 TLS/errno 布局差异

## 移植策略

### 目标新世界（推荐）

新世界是上游路径。在以下情况下应选择此目标：
- 新软件移植
- 长期可维护性
- 上游工具链支持

### 同时支持两者

如果您必须同时支持两者：

1. **使用特性检测**，而非硬编码假设
2. **在两种环境中测试** — 如可能使用 CI
3. **记录您的二进制文件目标世界**
4. **考虑 liblol** 用于在新世界系统上运行旧世界二进制文件

## 构建系统注意事项

### Autotools
```bash
# 新世界工具链通常能自动检测目标元组
./configure --build=loongarch64-unknown-linux-gnu
```

### CMake
```cmake
# 检查 LoongArch
cmake_host_system_information(RESULT LOONGARCH QUERY IS_LOONGARCH)
if(LOONGARCH)
    # LoongArch 特定设置
endif()
```

### Rust
```toml
# Cargo.toml 目标规范
# 新世界使用标准 loongarch64-unknown-linux-gnu
```

## 常见移植问题

### 信号处理
旧世界信号帧具有不同的布局。检查 `ucontext_t` 或进行基于信号的栈切换的代码可能需要被调整。

### 内联汇编
直接系统调用号在两个世界之间有所不同。请尽可能使用 libc 包装器，或在运行时检测系统调用号。

### errno 位置
直接通过 TLS 假设访问 `errno` 的代码可能会出现异常。请始终使用 `<errno.h>` 并正确处理错误。

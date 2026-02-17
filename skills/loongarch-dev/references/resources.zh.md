<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

# 社区资源与参考

## 官方文档

### 龙芯（厂商）

**⚠️ 重要**：主文档仓库中的 ABI 规范**已过时**。请改用下面的新规范仓库。

- **LoongArch 手册**：https://loongson.github.io/LoongArch-Documentation/
  - **ISA 手册（英文）**：仍然是指令集文档的权威来源
  - **ABI 规范**：⚠️ 已过时 —— 请改用下方的 la-abi-specs
- **当前 ABI 规范**：https://github.com/loongson/la-abi-specs
  - 最新的 ABI 文档（取代 LoongArch-Documentation 中的旧规范）
- **软件开发约定**：https://github.com/loongson/la-softdev-convention
  - LoongArch 软件的编码约定和指南
- **龙芯 GitHub**：https://github.com/loongson
- **内核补丁**：https://github.com/loongson/linux

## 社区资源

### LoongArch 社区
- **loongfans.cn**：https://loongfans.cn/ — 社区维基和资源
- **areweloongyet.com**：https://areweloongyet.com/ — 软件移植状态追踪器
- **GitHub 组织**：https://github.com/loongson-community

在这些站点上可以追踪：
- 软件移植状态
- 已知问题和解决方法
- 社区文档
- 软件包可用性

### 非官方社区资源
- **社区文档存档**：https://github.com/loongson-community/docs
  - 龙芯产品信息、数据表和技术文档的存档
- **社区讨论**：https://github.com/loongson-community/discussions
  - 整个龙芯生态系统的问题追踪器
  - 提问、报告问题、讨论移植问题

### 支持 LoongArch 的发行版

| 发行版 | 世界 | 说明 |
|--------------|-------|-------|
| **Gentoo** | 新世界 | 主要参考平台 |
| **Debian** | 新世界 | 官方移植进行中 |
| **AOSC OS** | 新世界 | liblol 用于兼容性 |
| **Loongnix** | 旧世界 | 龙芯官方发行版 |
| **Kylin/UOS** | 旧世界 | 商业中文发行版 |

## 技术参考

### SIMD 和内建函数
- **非官方 LSX/LASX 指南**：https://github.com/jiegec/unofficial-loongarch-intrinsics-guide
  - 全面的内建函数参考
  - 从 x86/ARM 移植的示例

### 二进制翻译（LBT）
- **LBT 文档**：https://github.com/jiegec/la-inst/blob/master/LBT.md
  - 二进制翻译特性的指令参考

### 兼容性
- **libLOL**：https://liblol.aosc.io/
  - 在新世界系统上运行旧世界二进制文件
- **源码**：https://github.com/AOSC-Dev/liblol

### 模拟和翻译
- **Box64**：https://github.com/ptitSeb/box64
  - 在 LoongArch 上运行 x86_64 二进制文件（可用时使用 LBT）
- **Box86**：https://github.com/ptitSeb/box86
  - 在 LoongArch 上运行 x86 二进制文件
- **QEMU**：https://gitlab.com/qemu-project/qemu
  - 完整系统和用户模式模拟

## 工具链信息

### GCC 支持
- **新世界最低版本**：GCC 12.1+
- **LoongArch 选项**：https://gcc.gnu.org/onlinedocs/gcc/LoongArch-Options.html
- **LSX 内建函数**：https://gcc.gnu.org/onlinedocs/gcc/LoongArch-LSX-Built-in-Functions.html
- **LASX 内建函数**：https://gcc.gnu.org/onlinedocs/gcc/LoongArch-LASX-Built-in-Functions.html

### LLVM/Clang 支持
- **新世界最低版本**：LLVM 16+
- **状态**：上游完全支持

### Rust 支持
- **目标**：`loongarch64-unknown-linux-gnu`
- **状态**：Tier 2（可用，不保证所有工具）
- **追踪**：https://github.com/rust-lang/rust/issues/1

## 获取帮助

### IRC/Matrix
- OFTC 或 Libera.Chat 上的 `#loongarch`
- 桥接到 Matrix

### 论坛
- LoongArch 社区论坛（中文）：从 loongfans.cn 链接

### 问题追踪
提交与 LoongArch 相关的问题时：
1. 指定旧世界与新世界
2. 包含 `uname -a` 输出
3. 包含编译器版本
4. 注意是否使用兼容层（liblol）

## 快速参考链接

| 主题 | 链接 |
|-------|------|
| ISA 手册（权威） | https://loongson.github.io/LoongArch-Documentation/ |
| ABI 规范（当前） | https://github.com/loongson/la-abi-specs |
| 软件开发约定 | https://github.com/loongson/la-softdev-convention |
| 移植状态追踪器 | https://areweloongyet.com/ |
| 社区维基 | https://loongfans.cn/ |
| 社区文档存档 | https://github.com/loongson-community/docs |
| 社区讨论 | https://github.com/loongson-community/discussions |
| SIMD 内建函数 | https://github.com/jiegec/unofficial-loongarch-intrinsics-guide |
| LBT 参考 | https://github.com/jiegec/la-inst/blob/master/LBT.md |
| 兼容层 | https://liblol.aosc.io/ |

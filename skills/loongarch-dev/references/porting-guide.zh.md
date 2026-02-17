<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

# LoongArch 移植指南

## 快速开始清单

将软件移植到 LoongArch 时：

- [ ] 确定目标 ABI（旧世界与新世界）
- [ ] 检查编译器支持（新世界需要 GCC 12+、LLVM 16+）
- [ ] 审查架构特定代码（内联汇编、内建函数）
- [ ] 检查字节序假设（LoongArch 是小端）
- [ ] 验证系统调用使用（避免硬编码数字）
- [ ] 测试 SIMD 代码路径（LSX/LASX 替代方案）
- [ ] 检查信号处理代码
- [ ] 审查内存对齐要求

## 字节序

LoongArch 是**小端**（与 x86_64 相同，不同于某些旧版 MIPS/SPARC）。

大多数现代软件正确处理这一点，但请注意：
```c
// 不良：硬编码字节序假设
#ifdef __x86_64__
    // 小端代码
#else
    // 对其他架构的错误假设
#endif

// 良好：使用标准宏
#if __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__
    // 小端代码
#else
    // 大端代码
#endif
```

## 编译器特性检测

### 预处理器宏

**重要**：LoongArch 区分 ISA/硬件能力（通用寄存器宽度）和 ABI 数据模型。这**不同于**其他架构中它们可能被混淆的情况。

```c
// 所有 LoongArch
#ifdef __loongarch__
    // LoongArch 特定代码
#endif

// ABI 数据模型（大多数检查的推荐方式）
#ifdef __loongarch_lp64
    // LP64 ABI：long 和指针为 64 位（LoongArch64 的标准）
#endif

#ifdef __loongarch_ilp32
    // ILP32 ABI：int、long 和指针为 32 位（罕见）
#endif

// ISA/硬件能力（通用寄存器宽度）—— __loongarch64 已弃用！
#if __loongarch_grlen == 64
    // 硬件具有 64 位通用寄存器
#endif

#if __loongarch_grlen == 32
    // 硬件具有 32 位通用寄存器
#endif

// 特定扩展
#ifdef __loongarch_sx    // LSX SIMD
#ifdef __loongarch_asx   // LASX SIMD  
#ifdef __loongarch_bt    // LBT 二进制翻译
#endif
```

### 关键区别

| 宏 | 含义 | 何时使用 |
|-------|---------|----------|
| `__loongarch_lp64` | ABI 使用 LP64 数据模型 | **几乎总是** — 检查指针/size_t 宽度时 |
| `__loongarch_ilp32` | ABI 使用 ILP32 数据模型 | 嵌入式 32 位场景 |
| `__loongarch_grlen == 64` | 硬件通用寄存器宽度为 64 位 | ISA 级代码生成 |
| `__loongarch64` | 已弃用，同上 | **避免** — 使用 `__loongarch_grlen` |

### 编译器识别

```c
#if defined(__GNUC__) && !defined(__clang__)
    // GCC 特定
#elif defined(__clang__)
    // Clang 特定
#endif
```

## 内联汇编

### 基本语法

LoongArch 使用标准 GCC 扩展汇编：

```c
uint64_t result;
__asm__ volatile (
    "add.d %0, %1, %2"    // 指令
    : "=r" (result)       // 输出
    : "r" (a), "r" (b)    // 输入
    :                     // 破坏描述
);
```

### 常用指令

| 操作 | 指令 | 说明 |
|-----------|-------------|-------|
| 64 位加法 | `add.d` | d = double（64 位） |
| 32 位加法 | `add.w` | w = word（32 位） |
| 64 位加载 | `ld.d` | |
| 64 位存储 | `st.d` | |
| 移动 | `move` | 寄存器移动 |
| 系统调用 | `syscall` | 优先使用 libc 包装器 |

### 寄存器名称

- `$r0-$r31`：通用寄存器
- `$zero` ($r0)：硬编码零
- `$ra` ($r1)：返回地址
- `$tp` ($r2)：线程指针
- `$sp` ($r3)：栈指针
- `$a0-$a7` ($r4-$r11)：参数/返回值
- `$t0-$t8` ($r12-$r20)：临时寄存器
- `$s0-$s8` ($r23-$r31)：保存的寄存器

## 常见移植问题

### 1. 硬编码系统调用号

**问题**：
```c
// x86_64 系统调用号 — 在 LoongArch 上损坏
syscall(0, ...);  // x86_64 上的 SYS_read，在 LoongArch 上错误
```

**解决方案**：
```c
#include <sys/syscall.h>
#include <unistd.h>

// 使用标准头文件
syscall(SYS_read, ...);  // 在所有架构上都正确
// 或更好：使用 libc 包装器
read(fd, buf, count);
```

### 2. 架构特定的内存屏障

**问题**：
```c
// x86 特定：mfence
__asm__ __volatile__ ("mfence" ::: "memory");
```

**解决方案**：
```c
// 使用标准原子操作
#include <stdatomic.h>
atomic_thread_fence(memory_order_seq_cst);

// 或编译器屏障
__atomic_thread_fence(__ATOMIC_SEQ_CST);
```

### 3. 信号上下文（ucontext_t）

**问题**：旧世界 LoongArch 具有不同的 `ucontext_t` 布局。

**解决方案**：正确使用 `ucontext.h`，尽可能避免直接访问 `uc_mcontext` 字段。如果必须访问，使用宏抽象：

```c
#ifdef __loongarch__
    #ifdef __loongarch_lp64
        #define GET_PC(ctx) ((ctx)->uc_mcontext.__pc)
    #endif
#endif
```

### 4. 自修改代码 / JIT

LoongArch 需要显式缓存同步：

```c
// 写入代码后，刷新缓存
__builtin___clear_cache(start, end);

// 或内联汇编：
__asm__ volatile (
    "dbar 0 \n\t"      // 数据屏障
    "ibar 0"          // 指令屏障
    ::: "memory"
);
```

### 5. 对齐的内存访问

LoongArch 可以处理未对齐访问，但有性能损失：

```c
// 优先为 SIMD 使用对齐分配
void *ptr;
posix_memalign(&ptr, 16, size);  // 用于 LSX
posix_memalign(&ptr, 32, size);  // 用于 LASX
```

## 构建系统集成

### Autotools

```bash
# configure.ac
AC_CANONICAL_HOST
case $host_cpu in
  loongarch*)
    AC_DEFINE([ARCH_LOONGARCH], [1], [LoongArch 架构])
    ;;
esac
```

### CMake

```cmake
if(CMAKE_SYSTEM_PROCESSOR MATCHES "loongarch")
    target_compile_definitions(mytarget PRIVATE ARCH_LOONGARCH=1)
endif()

# 特性检测
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

## 测试

### QEMU 用户模式

在 x86_64 主机上测试 LoongArch 二进制文件：

```bash
# 安装 qemu-user-loongarch64
# Debian/Ubuntu：apt install qemu-user-static

qemu-loongarch64 ./mybinary
```

### 交叉编译

```bash
# GCC 交叉编译器
loongarch64-linux-gnu-gcc -o mybinary mysource.c

# Clang（带交叉工具链）
clang --target=loongarch64-linux-gnu -o mybinary mysource.c
```

### 原生构建

在 LoongArch 硬件上：

```bash
# 检查编译器版本（新世界需要 GCC 12+ 或 LLVM 16+）
gcc --version
clang --version

# 标准构建
./configure && make
# 或
cmake -B build && cmake --build build
```

<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

# LoongArch ABI Worlds: Old World vs New World

## Overview

LoongArch has **two incompatible ABIs** that are a major source of confusion when porting software:

| Aspect | Old World (ABI 1.0) | New World (ABI 2.0) |
|--------|---------------------|---------------------|
| **Kernel** | 4.x-era patches | Upstream Linux 5.19+ |
**Toolchain** | Loongson's fork | Upstream GCC 12+, LLVM 16+ |
| **Distros** | Loongnix, Kylin, UOS | Gentoo, Debian, AOSC, etc. |
| **System calls** | Non-standard | Standard upstream |
| **errno location** | Thread-local (non-standard) | Standard TLS layout |
| **ld.so path** | `/lib64/ld.so.1` | `/lib64/ld-linux-loongarch-lp64d.so.1` |
| **Signal handling** | Different struct layout | Standard layout |

## Identifying Which World You're On

### Check the dynamic linker path:
```bash
# Old world
$ readelf -l /bin/ls | grep interpreter
      [Requesting program interpreter: /lib64/ld.so.1]

# New world
$ readelf -l /bin/ls | grep interpreter
      [Requesting program interpreter: /lib64/ld-linux-loongarch-lp64d.so.1]
```

### Check kernel version:
```bash
uname -r
# Old world: 4.x with custom patches
# New world: 5.19+ upstream kernel
```

## The Incompatibility Problem

**Old world and new world binaries cannot run on each other's systems** without compatibility layers because:

1. **Different syscall numbers** — kernel ABI mismatch
2. **Different TLS/errno layout** — thread-local storage differs
3. **Different signal frame layout** — signal handlers break
4. **Different ld.so** — dynamic linker incompatibility

## liblol: Compatibility Solution

**libLOL** (Loongson Old-world Linux) is an open-source compatibility layer from AOSC:

- **Project**: https://liblol.aosc.io/
- **Source**: https://github.com/AOSC-Dev/liblol
- **Function**: Allows old-world binaries to run on new-world systems

It works by:
- Providing a compatibility ld.so that intercepts syscalls
- Translating old-world syscalls to new-world equivalents
- Handling TLS/errno layout differences

## Porting Strategy

### Target New World (Recommended)

New world is the upstream path. Target this for:
- New software ports
- Long-term maintainability
- Upstream toolchain support

### Supporting Both

If you must support both:

1. **Use feature detection**, not hardcoded assumptions
2. **Test on both environments** — CI if possible
3. **Document which world your binaries target**
4. **Consider liblol** for running old-world binaries on new-world systems

## Build System Considerations

### Autotools
```bash
# New world toolchain typically auto-detects correctly
./configure --build=loongarch64-unknown-linux-gnu
```

### CMake
```cmake
# Check for LoongArch
cmake_host_system_information(RESULT LOONGARCH QUERY IS_LOONGARCH)
if(LOONGARCH)
    # LoongArch-specific settings
endif()
```

### Rust
```toml
# Cargo.toml target specification
# New world uses standard loongarch64-unknown-linux-gnu
```

## Common Porting Issues

### Signal Handling
Old-world signal frames have a different layout. Code that inspects `ucontext_t` or does signal-based stack switching may need adjustment.

### Inline Assembly
Direct syscall numbers differ between worlds. Use libc wrappers when possible, or detect at runtime.

### errno Location
Code that directly accesses `errno` via TLS assumptions may break. Always use `<errno.h>` and proper error handling.

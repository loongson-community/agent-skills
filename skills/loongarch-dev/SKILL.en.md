<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

---
name: loongarch-dev
description: LoongArch architecture development and porting guidance. Use when working with LoongArch (龙芯) for porting software, writing architecture-specific code, handling ABI compatibility between old world (ABI 1.0) and new world (ABI 2.0), using LSX/LASX SIMD intrinsics, LBT binary translation features, or resolving LoongArch-specific build issues.
---

# LoongArch Development

## Quick Reference

LoongArch is a RISC ISA from Loongson. Key characteristics:
- **64-bit little-endian** architecture
- **New world (ABI 2.0)** is the current standard ABI — assume it by default
- **SIMD**: LSX (128-bit) and LASX (256-bit) vector extensions
- **Binary Translation**: LBT for x86/ARM emulation acceleration

> **Default ABI assumption**: Unless the user explicitly mentions "old world",
> "ABI 1.0", legacy Loongson distributions, or the conversation suggests ABI
> confusion, always target the **new world (ABI 2.0)**. Do not proactively
> load old-world reference material.

## When to Load References

### [abi-worlds.en.md](references/abi-worlds.en.md)
Load **only** when:
- The user explicitly asks about old world / ABI 1.0
- There are signs of ABI confusion (e.g. mysterious runtime failures mixing old-world and new-world binaries)
- liblol compatibility layer questions arise
- Comparing legacy vs modern distribution differences

Do **not** load this for normal new-world porting work.

### [simd-lsx-lasx.en.md](references/simd-lsx-lasx.en.md)
Load when:
- Porting SIMD code from x86 (SSE/AVX) or ARM (NEON)
- Writing LSX/LASX intrinsics
- Detecting SIMD capabilities at runtime
- Optimizing vectorized code

### [lbt.en.md](references/lbt.en.md)
Load when:
- Working on binary translation (QEMU, Box64, etc.)
- Need x86-style EFLAGS emulation
- Building emulation layers

### [porting-guide.en.md](references/porting-guide.en.md)
Load for:
- General porting checklist and best practices
- Compiler feature detection (`__loongarch__` macros)
- Inline assembly syntax
- Common porting issues and solutions

### [resources.en.md](references/resources.en.md)
Load for:
- Links to official documentation
- Community resources (loongfans.cn, areweloongyet.com)
- Distribution support status
- Getting help

## Common Tasks

### Port a codebase to LoongArch
1. Read [porting-guide.en.md](references/porting-guide.en.md) for checklist
2. Target **new world (ABI 2.0)** unless the user explicitly requests old world
3. Check [resources.en.md](references/resources.en.md) for porting status of dependencies

### Fix SIMD-related build failures
1. Read [simd-lsx-lasx.en.md](references/simd-lsx-lasx.en.md)
2. Map existing intrinsics to LSX/LASX equivalents
3. Add runtime CPU detection if needed

### Run old-world binaries on new-world system
1. Read [abi-worlds.en.md](references/abi-worlds.en.md) section on liblol
2. Install liblol from https://liblol.aosc.io/

### Write inline assembly
1. Read [porting-guide.en.md](references/porting-guide.en.md) inline assembly section
2. Reference LoongArch instruction set manual
3. Prefer intrinsics over inline asm when available

## Architecture Detection

```c
// General LoongArch
#ifdef __loongarch__

// ABI Data Model (RECOMMENDED — use this!)
#ifdef __loongarch_lp64     // LP64: long and pointers are 64-bit
#ifdef __loongarch_ilp32    // ILP32: int/long/pointers are 32-bit

// ISA/HW Capability (GPR width) — __loongarch64 is deprecated!
#if __loongarch_grlen == 64  // Hardware has 64-bit GPRs

// Extensions
#ifdef __loongarch_sx   // LSX SIMD
#ifdef __loongarch_asx  // LASX SIMD
#ifdef __loongarch_bt   // LBT binary translation
```

**Note**: LoongArch distinguishes between **ABI data model** (`__loongarch_lp64`) and **ISA capability** (`__loongarch_grlen`). For most software, check the ABI (`lp64`) not the hardware. See [porting-guide.en.md](references/porting-guide.en.md) for details.

## Community

- **loongfans.cn** — Community wiki
- **areweloongyet.com** — Software porting status
- **github.com/loongson-community** — Community projects

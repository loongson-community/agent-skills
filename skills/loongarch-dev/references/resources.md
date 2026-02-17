<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

# Community Resources and References

## Official Documentation

### Loongson (Vendor)

**⚠️ Important**: The ABI specifications in the main documentation repository are **outdated**. Use the newer spec repositories below.

- **LoongArch Manuals**: https://loongson.github.io/LoongArch-Documentation/
  - **ISA Manual (English)**: Still the canonical source for instruction set documentation
  - **ABI Specs**: ⚠️ Outdated — see la-abi-specs below instead
- **Current ABI Specs**: https://github.com/loongson/la-abi-specs
  - Up-to-date ABI documentation (replaces older specs in LoongArch-Documentation)
- **Software Development Convention**: https://github.com/loongson/la-softdev-convention
  - Coding conventions and guidelines for LoongArch software
- **Loongson GitHub**: https://github.com/loongson
- **Kernel patches**: https://github.com/loongson/linux

## Community Resources

### LoongArch Community
- **loongfans.cn**: https://loongfans.cn/ — Community wiki and resources
- **areweloongyet.com**: https://areweloongyet.com/ — Software porting status tracker
- **GitHub Org**: https://github.com/loongson-community

These sites track:
- Software porting status
- Known issues and workarounds
- Community documentation
- Package availability

### Unofficial Community Resources
- **Community Docs Archive**: https://github.com/loongson-community/docs
  - Archive of Loongson product information, datasheets, and technical docs
- **Community Discussions**: https://github.com/loongson-community/discussions
  - Issue tracker for the greater Loongson ecosystem
  - Ask questions, report issues, discuss porting problems

### Distributions Supporting LoongArch

| Distribution | World | Notes |
|--------------|-------|-------|
| **Gentoo** | New world | Primary reference platform |
| **Debian** | New world | Official port in progress |
| **AOSC OS** | New world | liblol for compatibility |
| **Loongnix** | Old world | Loongson's official distro |
| **Kylin/UOS** | Old world | Commercial Chinese distros |

## Technical References

### SIMD and Intrinsics
- **Unofficial LSX/LASX Guide**: https://github.com/jiegec/unofficial-loongarch-intrinsics-guide
  - Comprehensive intrinsics reference
  - Porting examples from x86/ARM

### Binary Translation (LBT)
- **LBT Documentation**: https://github.com/jiegec/la-inst/blob/master/LBT.md
  - Instruction reference for binary translation features

### Compatibility
- **libLOL**: https://liblol.aosc.io/
  - Run old-world binaries on new-world systems
- **Source**: https://github.com/AOSC-Dev/liblol

### Emulation and Translation
- **Box64**: https://github.com/ptitSeb/box64
  - Run x86_64 binaries on LoongArch (uses LBT when available)
- **Box86**: https://github.com/ptitSeb/box86
  - Run x86 binaries on LoongArch
- **QEMU**: https://gitlab.com/qemu-project/qemu
  - Full system and user-mode emulation

## Toolchain Information

### GCC Support
- **Minimum for new world**: GCC 12.1+
- **LoongArch options**: https://gcc.gnu.org/onlinedocs/gcc/LoongArch-Options.html
- **LSX intrinsics**: https://gcc.gnu.org/onlinedocs/gcc/LoongArch-LSX-Built-in-Functions.html
- **LASX intrinsics**: https://gcc.gnu.org/onlinedocs/gcc/LoongArch-LASX-Built-in-Functions.html

### LLVM/Clang Support
- **Minimum for new world**: LLVM 16+
- **Status**: Fully supported upstream

### Rust Support
- **Target**: `loongarch64-unknown-linux-gnu`
- **Status**: Tier 2 (available, not all tools guaranteed)
- **Tracking**: https://github.com/rust-lang/rust/issues/1

## Getting Help

### IRC/Matrix
- `#loongarch` on OFTC or Libera.Chat
- Bridged to Matrix

### Forums
- LoongArch community forums (Chinese): linked from loongfans.cn

### Issue Tracking
When filing issues related to LoongArch:
1. Specify old world vs new world
2. Include `uname -a` output
3. Include compiler version
4. Note if using compatibility layers (liblol)

## Quick Reference Links

| Topic | Link |
|-------|------|
| ISA Manual (canonical) | https://loongson.github.io/LoongArch-Documentation/ |
| ABI Specs (current) | https://github.com/loongson/la-abi-specs |
| Software Dev Convention | https://github.com/loongson/la-softdev-convention |
| Port status tracker | https://areweloongyet.com/ |
| Community wiki | https://loongfans.cn/ |
| Community docs archive | https://github.com/loongson-community/docs |
| Community discussions | https://github.com/loongson-community/discussions |
| SIMD intrinsics | https://github.com/jiegec/unofficial-loongarch-intrinsics-guide |
| LBT reference | https://github.com/jiegec/la-inst/blob/master/LBT.md |
| Compatibility layer | https://liblol.aosc.io/ |

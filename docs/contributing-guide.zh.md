<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

# AI 智能体贡献指南

本文档是面向 AI 智能体的维护者首选贡献指南的中文版本。英文版请参阅 [contributing-guide.en.md](contributing-guide.en.md)。

## 核心要求

- 保持变更最小化、范围明确。
- 每个提交只做一件事（不在同一提交中混入无关修改）。
- 避免重新格式化无关文件。
- 提交前审查 diff，排除无关变更。
- 架构变更时同步更新 `AGENTS.md`。

## AIGC 政策

本项目执行严格的人工智能生成内容（AIGC）政策。**所有 AI 智能体在贡献之前必须阅读并遵守完整政策。**

- English: [aigc-policy.en.md](aigc-policy.en.md)
- 中文: [aigc-policy.zh.md](aigc-policy.zh.md)

要点（以完整政策为准）：

- **提交拆分**：每个提交的内容要么完全由人类撰写，要么完全由 AI 撰写，不得混合。
- **身份披露**：AI 智能体必须在每个 AIGC 提交中添加 `AI-assisted-by` trailer（如 `AI-assisted-by: Claude Opus 4.6 (GitHub Copilot)`）。
- **原始提示词**：在提交说明正文中（trailer 之前）记录用户的原始提示词。
- **人类复核**：所有提交必须经人类复核，由人类添加 `Signed-off-by`（DCO）。AI 智能体**禁止**代替用户添加此标签。
- **禁止敏感信息**：提交中不得包含密钥、凭据或个人隐私数据。

## 项目概况

- **定位**：为龙芯社区项目收集和维护智能体技能（AGENTS.md 文件、提示词工程资产及相关文档）。
- **语言**：以 Markdown 为主，技能定义中可能包含任意编程语言的代码。
- **许可证**：
  - 代码：`AGPL-3.0-or-later`
  - 文档：`CC-BY-NC-SA-4.0`

仓库结构：

- `skills/`：智能体技能定义（本仓库的主要内容）。每个技能是一份或一组自成一体的文档。
- `docs/`：*本项目自身*的文档（AIGC 政策、贡献指南等）以及多个技能之间共享的知识。
- `README.md` / `README.en.md`：中英文项目介绍。

## 内容工作流

### 添加新技能

- 将技能文件放在 `skills/` 目录下，按项目或主题适当组织。
- 每个技能应自成一体、文档完备。
- 如技能引用了外部上下文（架构细节、工具链等），请包含或链接必要的背景资料。

### 编辑现有技能

- 保留原有技能的意图和结构。
- 如从其他项目的 AGENTS.md 改编技能，请注明来源。

### 文档

- 项目级文档位于 `docs/`，涵盖*本项目自身*的政策、指南以及技能之间的共享知识——它们本身不是技能。
- 技能相关内容位于 `skills/`，请勿将两者混淆。
- AIGC 政策文档（`docs/aigc-policy.*.md`）具有规范性——不得在其他文档中削弱或与之矛盾。

### 双语要求

- 所有文档（项目级文档和技能文档）均须提供**中英文双语版本**，以最大化传播范围。
- 创建或修改文档时，须在同一提交或紧随其后的提交中同步另一语言版本的变更。
- 代码标识符和提交说明保持英文。

## 代码风格与规范

- Markdown 文件使用 ATX 风格标题（`#`、`##` 等）。
- 标题后、代码块前使用空行。
- 行宽保持合理长度（建议在 80–100 字符左右折行）。
- 文档文件顶部应添加 SPDX 许可证头（`<!-- SPDX-License-Identifier: ... -->`）。

## 提交说明风格

遵循 Conventional Commits 规范：

```plain
<type>(<scope>): <summary>
```

准则：

- 使用祈使语气、现在时态的简要描述（末尾不加句号）。
- 简要描述控制在约 50–72 个字符。
- 每个提交只做一件事——不要合并无关变更。
- 必要时在正文中说明动机或关键变更。
- 正文与简要描述之间用空行分隔；正文每行约 72 字符折行。

类型（type）：

- `feat`：新技能或新功能（含 `skills/` 下的技能文档新增）
- `fix`：Bug 修复或纠正（含 `skills/` 下的技能内容修正）
- `docs`：*本项目自身*文档的变更（`docs/`、`README`、`AGENTS.md` 等）——**不用于**技能内容
- `refactor`：不改变行为的重构
- `build`：构建系统或依赖变更
- `ci`：CI/CD 配置变更

**不要**使用 `chore`——请根据情况使用 `build` 或 `ci`。

范围（scope）：

- `skills`：技能定义
- `docs`：文档
- `policy`：AIGC 政策
- `meta`：仓库元数据（README、AGENTS.md、许可证）

### AIGC 提交要求

根据 [AIGC 政策](aigc-policy.zh.md)，AI 智能体必须：

1. 使用 `AI-assisted-by` trailer 披露身份。
2. 在提交说明正文中记录原始提示词。
3. **禁止**代替用户添加 `Signed-off-by`。

示例：

```plain
feat(skills): add LoongArch cross-compilation skill

Add a skill definition covering GCC and LLVM cross-compilation
workflows for LoongArch targets.

Original prompt:

> Create a skill for cross-compiling C/C++ projects targeting
> LoongArch using GCC and LLVM toolchains.

AI-assisted-by: Claude Opus 4.6 (GitHub Copilot)
Signed-off-by: Contributor Name <contributor@example.com>
```

## 验证清单

提交前请检查：

- ✅ Markdown 文件格式正确（无断链、标题层级合理）。
- ✅ 新文档文件包含 SPDX 许可证头。
- ✅ 中英文双语版本已同步创建或更新。
- ✅ 提交说明符合 Conventional Commits 规范及 AIGC 政策要求。
- ✅ 不包含敏感信息。
- ✅ 变更范围限于单一逻辑单元。

<!-- SPDX-License-Identifier: CC-BY-NC-SA-4.0 -->

# Contributing Guide for AI Agents

This document is the English version of the maintainer-preferred guide for AI agents working on this repository. For the Chinese version, see [contributing-guide.zh.md](contributing-guide.zh.md).

## Key expectations

- Keep changes minimal and scoped.
- One logical change per commit (no unrelated edits in the same commit).
- Avoid reformatting unrelated files.
- Review diffs for unrelated changes before finalizing.
- Update `AGENTS.md` when architectural changes occur.

## AIGC policy

This project enforces a strict AI-Generated Content (AIGC) policy. **All AI agents must read and comply with the full policy before contributing.**

- English: [aigc-policy.en.md](aigc-policy.en.md)
- 中文: [aigc-policy.zh.md](aigc-policy.zh.md)

Key points (the full policy is authoritative):

- **Commit separation**: Each commit should be either entirely human-written or entirely AI-written. Do not mix.
- **Identity disclosure**: AI agents must add an `AI-assisted-by` trailer to every AIGC commit (e.g., `AI-assisted-by: Claude Opus 4.6 (GitHub Copilot)`).
- **Original prompt**: Record the original user prompt in the commit message body before the trailers.
- **Human review**: All commits must be reviewed by a human, who appends `Signed-off-by` (DCO). AI agents must **not** add this tag on behalf of the user.
- **No sensitive information**: Never include keys, credentials, or personal data in commits.

## Project overview

- **Purpose**: A collection of agent skills (AGENTS.md files, prompt engineering assets, and supporting docs) for Loongson community projects.
- **Languages**: Markdown (primary), with potential code in any language within skill definitions.
- **Licenses**:
  - Code: `AGPL-3.0-or-later`
  - Documentation: `CC-BY-NC-SA-4.0`

High-level layout:

- `skills/`: Agent skill definitions (the main content of this repo). Each skill is a self-contained document or set of documents.
- `docs/`: Documentation pertaining to *this project itself* (AIGC policy, contributing guides, etc.) and shared knowledge referenced by multiple skills.
- `README.md` / `README.en.md`: Project introduction in Chinese and English.

## Content workflows

### Adding a new skill

- Place skill files under `skills/`. Organize by project or topic as appropriate.
- Each skill should be self-contained and well-documented.
- If a skill references external context (architecture details, toolchains, etc.), include or link to the necessary background.

### Editing existing skills

- Preserve the intent and structure of the original skill.
- If adapting a skill from another project's AGENTS.md, attribute the source.

### Documentation

- Project-wide docs live in `docs/`. These cover *this project's* policies, guides, and shared knowledge between skills — they are not skills themselves.
- Skill-related content lives under `skills/`. Do not confuse the two.
- The AIGC policy documents (`docs/aigc-policy.*.md`) are normative — do not weaken or contradict them elsewhere.

### Bilingual requirement

- All documentation (both project-level docs and skills) must be provided in **English and Chinese** to maximize outreach.
- When creating or modifying a document, always sync the change to the other language version in the same commit or an immediately following commit.
- Code identifiers and commit messages remain in English.

## Code style and conventions

- Markdown files should use ATX-style headings (`#`, `##`, etc.).
- Use blank lines after headings and before code blocks.
- Keep lines at a reasonable length (wrap around 80–100 characters where practical).
- SPDX license headers (`<!-- SPDX-License-Identifier: ... -->`) should appear at the top of documentation files.

## Commit message style

Follow Conventional Commits:

```plain
<type>(<scope>): <summary>
```

Guidelines:

- Imperative, present-tense summary (no trailing period).
- ~50–72 characters for summary.
- One logical change per commit — do not combine unrelated changes.
- Include a body when needed to explain motivation or key changes.
- Separate body from summary with a blank line; wrap body lines around 72 characters.

Types:

- `feat`: New skill or feature (including new skill documentation under `skills/`)
- `fix`: Bug fix or correction (including fixes to skill content under `skills/`)
- `docs`: Changes to *this project's own* documentation (under `docs/`, `README`, `AGENTS.md`, etc.) — **not** for skill content
- `refactor`: Restructuring without behavior change
- `build`: Build system or dependency changes
- `ci`: CI/CD configuration changes

Do **not** use `chore` — use `build` or `ci` as appropriate instead.

Scopes:

- `skills`: Skill definitions
- `docs`: Documentation
- `policy`: AIGC policy
- `meta`: Repository metadata (README, AGENTS.md, licenses)

### AIGC commit requirements

Per the [AIGC policy](aigc-policy.en.md), AI agents must:

1. Disclose identity with an `AI-assisted-by` trailer.
2. Record the original prompt in the commit body.
3. **Not** add `Signed-off-by` on behalf of the user.

Example:

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

## Validation checklist

Before finalizing a commit, verify:

- ✅ Markdown files are well-formed (no broken links, proper heading hierarchy).
- ✅ SPDX license headers are present on new documentation files.
- ✅ Both English and Chinese versions are created or updated in sync.
- ✅ Commit message follows Conventional Commits and AIGC policy requirements.
- ✅ No sensitive information is included.
- ✅ Changes are scoped to a single logical unit.

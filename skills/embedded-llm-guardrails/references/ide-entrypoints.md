# 多 IDE / Agent 入口文件

本文件记录同一套嵌入式 LLM 规则在不同 IDE 或 Agent 中的落地方式。

## 通用策略

推荐把完整规则和边界配置放在项目 `.claude/` 目录下，避免污染工程根目录：

```text
.claude/LLM_RULES.md
.claude/LLM_BOUNDARY.md
```

然后让不同 IDE 的规则文件引用这两个文件，避免多处规则不一致。

## Claude Code

推荐入口：

```text
CLAUDE.md
```

建议内容：

```markdown
# {{项目名称}} — Claude Code 指令

## 项目概览

- MCU: {{MCU 型号}}
- 工具链: {{工具链}}
- 生成工具: {{代码生成器，如 STM32CubeMX / 无}}
- 状态: {{裸机/RTOS}}

## 安全边界

**修改代码前必须先阅读并遵守：**

- `.claude/LLM_RULES.md` — LLM 修改行为总规则
- `.claude/LLM_BOUNDARY.md` — 允许/禁止修改的文件边界

核心规则：
- 只修改当前任务 `Allowed Files` 列表中的文件
- 禁止修改启动文件、链接脚本、HAL 驱动、CMSIS、代码生成器输出中保护区域外的部分
- 遵守 `VERIFIED_ON_HARDWARE` 等硬件验证标记
- 每次只做最小可验证补丁，然后给出验证步骤
- 涉及外设驱动时，先确认硬件参数再生成代码

## 构建

```bash
{{构建命令}}
```

## 项目结构

```
{{关键目录结构}}
```
```

**说明：** `CLAUDE.md` 会在每次 Claude Code 会话开始时自动加载到上下文中，因此应保持简洁，重点放引用和构建命令，详细规则放在 `.claude/LLM_RULES.md` 和 `.claude/LLM_BOUNDARY.md` 中。

## Codex

推荐入口：

```text
AGENTS.md
```

建议内容：

```markdown
请严格遵守项目 .claude/LLM_RULES.md 和 .claude/LLM_BOUNDARY.md。
在嵌入式代码任务中，只做最小可验证修改。
未列入 Allowed Files 的文件默认禁止修改。
```

## Cursor

推荐入口：

```text
.cursor/rules/embedded-llm-guardrails.mdc
```

或旧版：

```text
.cursorrules
```

建议内容：

```markdown
Follow .claude/LLM_RULES.md and .claude/LLM_BOUNDARY.md before editing embedded firmware.
Only modify files listed in the current task.
Never refactor hardware-verified code unless explicitly requested.
```

## GitHub Copilot

推荐入口：

```text
.github/copilot-instructions.md
```

建议内容：

```markdown
This is an embedded firmware project.
Follow .claude/LLM_RULES.md and .claude/LLM_BOUNDARY.md.
Prefer minimal, buildable, hardware-verifiable patches.
Do not alter startup files, linker scripts, BSP, vendor drivers, ISR, or verified timing code unless explicitly allowed.
```

## Windsurf

推荐入口：

```text
.windsurfrules
```

建议内容：

```markdown
Use .claude/LLM_RULES.md and .claude/LLM_BOUNDARY.md as the source of truth.
Do not rewrite working embedded code.
Do not remove main-loop logic or timing-sensitive code.
```

## JetBrains AI

JetBrains AI 可使用项目级说明文档。推荐在项目 `.claude/` 目录保留：

```text
.claude/LLM_RULES.md
.claude/LLM_BOUNDARY.md
```

并在会话或项目说明中明确：

```markdown
所有 AI 代码修改必须遵守 .claude/LLM_RULES.md 和 .claude/LLM_BOUNDARY.md。
```

## Cline / Roo Code

推荐入口：

```text
.clinerules
```

或项目级：

```text
.cline/rules/embedded-llm-guardrails.md
```

建议内容：

```markdown
Follow .claude/LLM_RULES.md and .claude/LLM_BOUNDARY.md before editing embedded firmware.
Only modify files listed in the current task.
Never alter startup, linker, BSP, vendor drivers, ISR, or hardware-verified code.
```

## Augment

推荐入口：

```text
.augment-guidelines
```

建议内容：

```markdown
This is an embedded firmware project.
Follow .claude/LLM_RULES.md and .claude/LLM_BOUNDARY.md for all code modifications.
Minimal verifiable patches only.
```

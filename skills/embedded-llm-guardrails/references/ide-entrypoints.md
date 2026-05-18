# 多 IDE / Agent 入口文件

本文件记录同一套嵌入式 LLM 规则在不同 IDE 或 Agent 中的落地方式。

## 通用策略

推荐把完整规则和边界配置放在项目根目录：

```text
LLM_RULES.md
LLM_BOUNDARY.md
```

然后让不同 IDE 的规则文件引用这两个文件，避免多处规则不一致。

## Codex

推荐入口：

```text
AGENTS.md
```

建议内容：

```markdown
请严格遵守项目根目录的 LLM_RULES.md 和 LLM_BOUNDARY.md。
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
Follow LLM_RULES.md and LLM_BOUNDARY.md before editing embedded firmware.
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
Follow LLM_RULES.md and LLM_BOUNDARY.md.
Prefer minimal, buildable, hardware-verifiable patches.
Do not alter startup files, linker scripts, BSP, vendor drivers, ISR, or verified timing code unless explicitly allowed.
```

## Claude Code

推荐入口：

```text
CLAUDE.md
```

建议内容：

```markdown
Read LLM_RULES.md and LLM_BOUNDARY.md before making changes.
Respect allowed files, forbidden files, and hardware-verified markers.
Stop after the smallest verifiable patch and provide verification steps.
```

## Windsurf

推荐入口：

```text
.windsurfrules
```

建议内容：

```markdown
Use LLM_RULES.md and LLM_BOUNDARY.md as the source of truth.
Do not rewrite working embedded code.
Do not remove main-loop logic or timing-sensitive code.
```

## JetBrains AI

JetBrains AI 可使用项目级说明文档。推荐在仓库根目录保留：

```text
LLM_RULES.md
LLM_BOUNDARY.md
```

并在会话或项目说明中明确：

```markdown
所有 AI 代码修改必须遵守 LLM_RULES.md 和 LLM_BOUNDARY.md。
```

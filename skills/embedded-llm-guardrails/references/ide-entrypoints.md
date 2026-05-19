# 主流 IDE / Agent 入口文件

本文件记录同一套嵌入式 LLM 规则在主流 IDE / Agent 中的落地方式。

## 通用策略

推荐把完整规则和边界配置放在项目内的统一目录，默认使用：

```text
.claude/LLM_RULES.md
.claude/LLM_BOUNDARY.md
```

如果团队不希望使用 `.claude/` 作为通用规则目录，也可以改为：

```text
.ai/LLM_RULES.md
.ai/LLM_BOUNDARY.md
```

无论使用哪个目录，各 IDE/Agent 的入口文件只做一件事：引用这两个规则文件，避免多处复制导致规则不一致。

推荐入口文件都包含三条硬规则：

```markdown
Read and follow {{RULE_DIR}}/LLM_RULES.md and {{RULE_DIR}}/LLM_BOUNDARY.md before editing firmware.
Only modify files explicitly listed in the current task's Allowed Files.
Prefer minimal, buildable, hardware-verifiable patches.
```

下文默认使用 `.claude/`。如项目使用 `.ai/`，替换路径即可。

## Cursor

推荐入口：

```text
.cursor/rules/embedded-llm-guardrails.mdc
```

旧版 Cursor 可使用：

```text
.cursorrules
```

建议内容：

```markdown
Follow `.claude/LLM_RULES.md` and `.claude/LLM_BOUNDARY.md` before editing embedded firmware.
Only modify files listed in the current task's Allowed Files.
Never refactor hardware-verified code unless explicitly requested.
Do not edit startup, linker, BSP, vendor SDK/HAL, ISR, generated-code protected areas, or timing-sensitive code unless explicitly allowed.
Use minimal verifiable patches and provide build or board validation steps.
```

## Codex

推荐入口：

```text
AGENTS.md
```

建议内容：

```markdown
请严格遵守项目 `.claude/LLM_RULES.md` 和 `.claude/LLM_BOUNDARY.md`。
在嵌入式代码任务中，只做最小可验证修改。
未列入 Allowed Files 的文件默认禁止修改。
不要修改启动文件、链接脚本、BSP、厂商 SDK/HAL、ISR、代码生成器保护区和已验证时序代码，除非用户明确允许。
涉及外设驱动时，先确认硬件参数和完整信号路径，再生成代码。
```

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

修改代码前必须先阅读并遵守：

- `.claude/LLM_RULES.md`
- `.claude/LLM_BOUNDARY.md`

核心规则：
- 只修改当前任务 Allowed Files 列表中的文件
- 禁止修改启动文件、链接脚本、HAL/SDK、CMSIS、代码生成器输出中保护区域外的部分
- 遵守 VERIFIED_ON_HARDWARE 等硬件验证标记
- 每次只做最小可验证补丁，然后给出验证步骤
- 涉及外设驱动时，先确认硬件参数和完整信号路径再生成代码
- main.c 中只允许添加带注释的函数调用，不写复杂业务逻辑

## 构建

```bash
{{构建命令}}
```
```

## OpenCode

推荐入口：

```text
AGENTS.md
```

如果团队希望与 Codex 分开，也可以使用项目说明文档：

```text
OPENCODE.md
```

建议内容：

```markdown
This is an embedded firmware project.
Read `.claude/LLM_RULES.md` and `.claude/LLM_BOUNDARY.md` before editing code.
Only modify files explicitly listed in the current task's Allowed Files.
If Allowed Files are missing, ask for them and do not edit code.
Do not edit startup files, linker scripts, BSP, vendor SDK/HAL, ISR, generated-code protected areas, or hardware-verified code unless explicitly allowed.
Use minimal buildable patches and provide validation steps.
```

## 兜底建议

IDE/Agent 规则不能完全防止误改。建议配合 Git diff 审查或 CI 检查：

```text
- diff 中出现启动文件、链接脚本、BSP、SDK/HAL、ISR、代码生成区修改时提醒或失败
- diff 中删除 VERIFIED_ON_HARDWARE / DO_NOT_TOUCH_TIMING 标记时提醒或失败
- main.c 或主循环中新增超过函数调用级别的复杂逻辑时提醒或失败
```

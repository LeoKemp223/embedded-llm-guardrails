# Claude Code / Codex / Cursor 入口文件

本文件记录同一套嵌入式 LLM 规则在 Claude Code、Codex 和 Cursor 中的落地方式。

## 通用策略

推荐把完整规则和边界配置放在目标项目内的统一目录：

```text
.ai/LLM_RULES.md
.ai/LLM_BOUNDARY.md
```

各工具的入口文件只做一件事：引用这两个规则文件，避免多处复制导致规则不一致。

推荐入口文件都包含三条硬规则：

```markdown
修改固件前必须阅读并遵守 `.ai/LLM_RULES.md` 和 `.ai/LLM_BOUNDARY.md`。
只允许修改当前任务 Allowed Files 列表中显式列出的文件。
优先输出最小、可编译、可上板验证的补丁。
```

所有入口文件模板（CLAUDE.md / AGENTS.md / Cursor `.mdc`）一律使用中文。Codex、Claude Code、Cursor 在生成或更新这些入口文件时，必须逐字使用本文件提供的中文模板，不得自行翻译为英文或重写为其它语言。

如果项目只使用 Claude Code，也可以把共享规则放在 `.claude/` 下；多工具共用时推荐 `.ai/`。

## Claude Code

Claude Code 的 skill 本体放在：

```text
.claude/skills/embedded-llm-guardrails/
```

推荐入口文件：

```text
CLAUDE.md
```

建议内容：

```markdown
# {{项目名称}} — Claude Code 指令

## 项目概述

- 芯片: {{MCU 型号，如 STM32F103RCT6}}
- 工具链: {{工具链，如 arm-none-eabi-gcc 10.3}}
- 构建系统: {{构建系统，如 CMake + Ninja / Makefile / Keil}}
- 生成工具: {{代码生成器，如 STM32CubeMX 6.15.0 / ESP-IDF idf.py / 无}}
- 状态: {{裸机 / FreeRTOS / Zephyr}}

## 安全边界

修改代码前必须先阅读并遵守：

- `.ai/LLM_RULES.md`
- `.ai/LLM_BOUNDARY.md`

核心规则：
- 只修改当前任务 Allowed Files 列表中的文件
- 禁止修改启动文件、链接脚本、HAL/SDK、CMSIS、代码生成器输出中保护区域外的部分
- 遵守 VERIFIED_ON_HARDWARE 等硬件验证标记
- 每次只做最小可验证补丁，然后给出验证步骤
- 涉及外设驱动时，先确认硬件参数和完整信号路径再生成代码
- main.c 中只允许添加带注释的函数调用，不写复杂业务逻辑
- 代码生成器项目中，存在「在生成器中配置 vs 直接改代码」两条实现路径时，必须先与用户确认路径

## 构建

```bash
{{构建命令，如 cmake --build build}}
```

## 烧录

```bash
{{烧录命令，如 STM32_Programmer_CLI -c port=SWD -w build/firmware.hex -rst}}
```

- 烧录器：{{ST-Link V2 / J-Link / DAPLink / 板载 USB-DFU}}
- 接口：{{SWD / JTAG / UART / USB-DFU}}
- 复位/启动模式：{{是否需要手动按 BOOT0+RESET，是否自动复位运行}}
```

## Codex

Codex 支持从本仓库根目录安装 skill 包，因为根目录包含 `SKILL.md`。

推荐项目入口：

```text
AGENTS.md
```

Codex 生成 AGENTS.md 时必须逐字使用下列中文模板，不得自行翻译为英文。建议内容：

```markdown
# {{项目名称}} — Codex 指令

请严格遵守项目 `.ai/LLM_RULES.md` 和 `.ai/LLM_BOUNDARY.md`。

## 项目概述

- 芯片: {{MCU 型号，如 STM32F103RCT6}}
- 工具链: {{工具链，如 arm-none-eabi-gcc 10.3}}
- 构建系统: {{构建系统，如 CMake + Ninja / Makefile / Keil}}
- 生成工具: {{代码生成器，如 STM32CubeMX 6.15.0 / ESP-IDF idf.py / 无}}
- 状态: {{裸机 / FreeRTOS / Zephyr}}

## 安全边界

在嵌入式代码任务中，只做最小可验证修改。

核心规则：
- 未列入 Allowed Files 的文件默认禁止修改
- 不要修改启动文件、链接脚本、BSP、厂商 SDK/HAL、ISR、代码生成器保护区和已验证时序代码，除非用户明确允许
- 遵守 VERIFIED_ON_HARDWARE 等硬件验证标记，不删除、重排或重构
- 每次只做最小可验证补丁，然后给出验证步骤
- 涉及外设驱动时，先确认硬件参数和完整信号路径，再生成代码
- main.c 中只允许添加带注释的函数调用，不写复杂业务逻辑
- 代码生成器项目中，存在「在生成器中配置 vs 直接改代码」两条实现路径时，必须先与用户确认路径

## 构建

```bash
{{构建命令，如 cmake --build build}}
```

## 烧录

```bash
{{烧录命令，如 STM32_Programmer_CLI -c port=SWD -w build/firmware.hex -rst}}
```

- 烧录器：{{ST-Link V2 / J-Link / DAPLink / 板载 USB-DFU}}
- 接口：{{SWD / JTAG / UART / USB-DFU}}
- 复位/启动模式：{{是否需要手动按 BOOT0+RESET，是否自动复位运行}}
```

## Cursor

Cursor 不读取 `SKILL.md` 作为 skill 包。Cursor 使用 rules 入口文件：

```text
.cursor/rules/embedded-llm-guardrails.mdc
```

建议内容：

```markdown
---
description: 嵌入式固件安全边界
alwaysApply: true
---

# {{项目名称}} — Cursor 规则

## 项目概述

- 芯片: {{MCU 型号，如 STM32F103RCT6}}
- 工具链: {{工具链，如 arm-none-eabi-gcc 10.3}}
- 构建系统: {{构建系统，如 CMake + Ninja / Makefile / Keil}}
- 生成工具: {{代码生成器，如 STM32CubeMX 6.15.0 / ESP-IDF idf.py / 无}}
- 状态: {{裸机 / FreeRTOS / Zephyr}}

## 安全边界

修改固件前必须阅读并遵守 `.ai/LLM_RULES.md` 和 `.ai/LLM_BOUNDARY.md`。

核心规则：
- 只允许修改当前任务 Allowed Files 列表中显式列出的文件
- 未经用户明确允许，不得重构已上板验证过的代码
- 不得修改启动文件、链接脚本、BSP、厂商 SDK/HAL、ISR、代码生成器保护区和已验证时序代码
- 遵守 VERIFIED_ON_HARDWARE / DO_NOT_TOUCH_TIMING 等硬件验证标记
- 只输出最小、可编译、可上板验证的补丁，并附构建或上板验证步骤
- 涉及外设驱动时，先确认硬件参数和完整信号路径，再生成代码
- main.c 中只允许添加带注释的函数调用，不写复杂业务逻辑
- 代码生成器项目中，存在「在生成器中配置 vs 直接改代码」两条实现路径时，必须先与用户确认路径

## 构建

```bash
{{构建命令，如 cmake --build build}}
```

## 烧录

```bash
{{烧录命令，如 STM32_Programmer_CLI -c port=SWD -w build/firmware.hex -rst}}
```

- 烧录器：{{ST-Link V2 / J-Link / DAPLink / 板载 USB-DFU}}
- 接口：{{SWD / JTAG / UART / USB-DFU}}
- 复位/启动模式：{{是否需要手动按 BOOT0+RESET，是否自动复位运行}}
```

## 兜底建议

IDE/Agent 规则不能完全防止误改。建议配合 Git diff 审查或 CI 检查：

```text
- diff 中出现启动文件、链接脚本、BSP、SDK/HAL、ISR、代码生成区修改时提醒或失败
- diff 中删除 VERIFIED_ON_HARDWARE / DO_NOT_TOUCH_TIMING 标记时提醒或失败
- main.c 或主循环中新增超过函数调用级别的复杂逻辑时提醒或失败
```

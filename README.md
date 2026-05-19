# Embedded LLM Guardrails

用于在嵌入式、单片机、RTOS、驱动和板级工程中使用 LLM 辅助开发时建立安全边界的技能模板。

## 核心原则

- **白名单制**：未显式允许修改的文件，默认禁止修改
- **最小补丁**：每次只做当前任务要求的最小可验证修改
- **TDD 闭环**：先定义验证方式，再实现最小代码，验证通过后再扩展
- **保护已验证代码**：带 `VERIFIED_ON_HARDWARE` 等标记的代码不允许删除、重排或重构
- **新功能独立文件**：添加新功能时必须创建独立的 `.c` / `.h` 文件

## 项目结构

```text
skills/embedded-llm-guardrails/
  SKILL.md                          技能主文件：执行顺序和硬规则
  assets/
    LLM_RULES.md                    嵌入式开发规则模板（含 {{...}} 占位符）
    LLM_BOUNDARY.md                 修改边界配置模板（STM32CubeMX / ESP-IDF / 裸机）
    llm-change-request.md           单次修改请求模板
  references/
    ide-entrypoints.md              Claude Code / Cursor / Codex 入口方式
```

## 使用时生成的文件

本模板区分两类文件：

- **项目共享规则**：放在目标项目的 `.ai/` 目录，供 Claude Code、Cursor、Codex 共同引用。
- **IDE 入口文件**：放在各 IDE 自己识别的位置，只负责引用 `.ai/` 下的共享规则。

推荐目标项目结构：

```text
/path/to/project/
  .ai/
    LLM_RULES.md                    规则文件（从模板填写）
    LLM_BOUNDARY.md                 边界配置（从模板填写）
  CLAUDE.md                         Claude Code 入口文件
  AGENTS.md                         Codex 入口文件
  .cursor/
    rules/
      embedded-llm-guardrails.mdc   Cursor 入口文件
```

如果只使用 Claude Code，也可以把共享规则放在 `.claude/` 下；但多 IDE 共用时推荐 `.ai/`，避免把 Claude Code 的目录误当成所有工具的公共安装目录。

`LLM_RULES.md` 中的 `{{...}}` 占位符需替换为项目实际信息（MCU 型号、工具链、构建系统等）。`LLM_BOUNDARY.md` 提供了 STM32CubeMX、ESP-IDF、裸机三种项目模板，根据项目类型选择并填写路径。

## 安装

### 1. 配置项目共享规则

将模板文件复制到目标项目的 `.ai/` 目录，并填写项目实际信息：

```bash
mkdir -p /path/to/project/.ai
cp skills/embedded-llm-guardrails/assets/LLM_RULES.md /path/to/project/.ai/
cp skills/embedded-llm-guardrails/assets/LLM_BOUNDARY.md /path/to/project/.ai/
```

需要填写的内容：

- `LLM_RULES.md`：替换 `{{...}}` 占位符（MCU 型号、工具链、构建系统、RTOS 等）
- `LLM_BOUNDARY.md`：根据项目类型选择模板章节（STM32CubeMX / ESP-IDF / 裸机），填写实际文件路径

### 2. 配置 IDE 入口

当前仅维护 Claude Code、Cursor、Codex 三种入口方式。

**Claude Code**

Claude Code 的 skill 本体放在 `.claude/skills/`，入口文件使用项目根目录的 `CLAUDE.md`：

```bash
mkdir -p /path/to/project/.claude/skills
cp -r skills/embedded-llm-guardrails /path/to/project/.claude/skills/
```

`CLAUDE.md` 示例：

```markdown
# Embedded Firmware Instructions

Read and follow `.ai/LLM_RULES.md` and `.ai/LLM_BOUNDARY.md` before editing firmware.
Only modify files explicitly listed in the current task's Allowed Files.
Prefer minimal, buildable, hardware-verifiable patches.
```

**Cursor**

Cursor 不需要把文件放到 `.claude/`。创建 Cursor 规则入口：

```bash
mkdir -p /path/to/project/.cursor/rules
cp skills/embedded-llm-guardrails/references/ide-entrypoints.md \
  /path/to/project/.cursor/rules/embedded-llm-guardrails.mdc
```

也可以只写入下面的最小规则：

```markdown
Follow `.ai/LLM_RULES.md` and `.ai/LLM_BOUNDARY.md` before editing embedded firmware.
Only modify files listed in the current task's Allowed Files.
Never edit startup, linker, BSP, vendor SDK/HAL, ISR, generated-code protected areas, or hardware-verified code unless explicitly allowed.
Use minimal verifiable patches and provide build or board validation steps.
```

**Codex**

Codex 的项目入口文件是 `AGENTS.md`。如果需要把本 skill 安装给 Codex 使用，放到 Codex 的 skill 目录；项目内只需要 `AGENTS.md` 引用共享规则。

```bash
cp skills/embedded-llm-guardrails/SKILL.md /path/to/project/AGENTS.md
```

`AGENTS.md` 示例：

```markdown
请严格遵守项目 `.ai/LLM_RULES.md` 和 `.ai/LLM_BOUNDARY.md`。
在嵌入式代码任务中，只做最小可验证修改。
未列入 Allowed Files 的文件默认禁止修改。
不要修改启动文件、链接脚本、BSP、厂商 SDK/HAL、ISR、代码生成器保护区和已验证时序代码，除非用户明确允许。
涉及外设驱动时，先确认硬件参数和完整信号路径，再生成代码。
```

## 卸载

**Claude Code**

```bash
rm -rf /path/to/project/.claude/skills/embedded-llm-guardrails
rm -f /path/to/project/CLAUDE.md
```

**Cursor**

```bash
rm -f /path/to/project/.cursor/rules/embedded-llm-guardrails.mdc
```

**Codex**

```bash
rm -f /path/to/project/AGENTS.md
```

**删除项目共享规则**

```bash
rm -f /path/to/project/.ai/LLM_RULES.md
rm -f /path/to/project/.ai/LLM_BOUNDARY.md
```

## 更新

从本仓库拉取最新版本后，重新复制对应入口文件或 skill 目录：

```bash
git pull
# 然后重新执行对应 IDE 的安装命令
```

更新后需检查：

- `LLM_RULES.md` 中新增的占位符是否需要填写
- `LLM_BOUNDARY.md` 是否需要根据项目变化更新路径

## 支持的项目类型

`LLM_BOUNDARY.md` 内置三种项目模板：

| 项目类型 | 检测标识 | 模板章节 |
|---------|---------|---------|
| STM32CubeMX | `.ioc` 文件 | CubeMX 项目模板（含 USER CODE 保护） |
| ESP-IDF | `sdkconfig` 文件 | ESP-IDF 项目模板 |
| 裸机 / 通用 | 无标识文件 | 通用项目模板 |

## 许可证

MIT

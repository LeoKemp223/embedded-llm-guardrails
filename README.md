# Embedded LLM Guardrails

用于在嵌入式、单片机、RTOS、驱动和板级工程中使用 LLM 辅助开发时建立安全边界的 skill。

本仓库根目录就是可直接安装的 skill 包，根目录包含必需的 `SKILL.md`。不要再从 `skills/embedded-llm-guardrails/` 子目录安装。

## 核心原则

- **白名单制**：未显式允许修改的文件，默认禁止修改
- **最小补丁**：每次只做当前任务要求的最小可验证修改
- **TDD 闭环**：先定义验证方式，再实现最小代码，验证通过后再扩展
- **保护已验证代码**：带 `VERIFIED_ON_HARDWARE` 等标记的代码不允许删除、重排或重构
- **新功能独立文件**：添加新功能时必须创建独立的 `.c` / `.h` 文件

## Skill 包结构

```text
embeded-ai/
  SKILL.md                          skill 主文件：执行顺序和硬规则
  assets/
    LLM_RULES.md                    嵌入式开发规则模板（含 {{...}} 占位符）
    LLM_BOUNDARY.md                 修改边界配置模板（STM32CubeMX / ESP-IDF / 裸机）
    llm-change-request.md           单次修改请求模板
    settings.json                   Claude Code 权限配置模板
  references/
    ide-entrypoints.md              Claude Code / Codex / Cursor 入口方式
    project-patterns.md             嵌入式项目类型识别指南
```

## 目标项目中的文件

本 skill 区分两类文件：

- **项目共享规则**：放在目标项目的 `.ai/` 目录，供 Claude Code、Codex 和 Cursor 共同引用。
- **入口文件**：放在工具自己识别的位置，只负责引用 `.ai/` 下的共享规则。

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
      embedded-llm-guardrails.mdc   Cursor 规则入口
```

如果只使用 Claude Code，也可以把共享规则放在 `.claude/` 下；多工具共用时推荐 `.ai/`，避免把 Claude Code 的目录误当成所有工具的公共安装目录。

`LLM_RULES.md` 中的 `{{...}}` 占位符需替换为项目实际信息（MCU 型号、工具链、构建系统等）。`LLM_BOUNDARY.md` 提供 STM32CubeMX、ESP-IDF、裸机三种项目模板，根据项目类型选择并填写路径。

## 安装 Skill / 规则入口

当前维护 Claude Code、Codex 和 Cursor 三种接入方式。Claude Code 和 Codex 使用 `SKILL.md` skill 包；Cursor 使用 `.cursor/rules/*.mdc` 规则入口。

### Codex

Codex 可以从本仓库根目录直接安装，因为根目录包含 `SKILL.md`。

使用 skill 安装器时，仓库路径应指向根目录：

```bash
codex skill install --repo LeoKemp223/embeded-ai --path .
```

安装后，Codex 会把整个 skill 包放到 Codex 的 skill 目录。不要使用 `--path skills/embedded-llm-guardrails`，该旧路径已废弃。

目标固件项目仍需要创建 `AGENTS.md` 作为项目入口：

```markdown
请严格遵守项目 `.ai/LLM_RULES.md` 和 `.ai/LLM_BOUNDARY.md`。
在嵌入式代码任务中，只做最小可验证修改。
未列入 Allowed Files 的文件默认禁止修改。
不要修改启动文件、链接脚本、BSP、厂商 SDK/HAL、ISR、代码生成器保护区和已验证时序代码，除非用户明确允许。
涉及外设驱动时，先确认硬件参数和完整信号路径，再生成代码。
```

### Claude Code

Claude Code 的 skill 本体放在目标项目的 `.claude/skills/` 下。复制本仓库根目录作为 skill 目录：

```bash
mkdir -p /path/to/project/.claude/skills
cp -r /path/to/embeded-ai /path/to/project/.claude/skills/embedded-llm-guardrails
```

目标固件项目使用根目录的 `CLAUDE.md` 作为入口。完整 CLAUDE.md 模板（含项目概述、安全边界、构建三节）见 `references/ide-entrypoints.md`，安装时按目标项目实际值替换 `{{...}}` 占位符。

如需 Claude Code 权限配置，可根据 `assets/settings.json` 生成或合并到目标项目的 `.claude/settings.json`。

### Cursor

Cursor 不读取 `SKILL.md` 作为 skill 包。给 Cursor 使用时，在目标固件项目创建 rules 入口，让它引用 `.ai/` 下的共享规则：

```bash
mkdir -p /path/to/project/.cursor/rules
cp references/ide-entrypoints.md /path/to/project/.cursor/rules/embedded-llm-guardrails.mdc
```

完整 `.cursor/rules/embedded-llm-guardrails.mdc` 模板（含项目概述、安全边界、构建三节）见 `references/ide-entrypoints.md`，安装时按目标项目实际值替换 `{{...}}` 占位符。

## 配置目标项目

安装 skill 只让工具“知道这套流程”。每个目标固件项目仍需要自己的规则和边界文件。

将模板文件复制到目标项目的 `.ai/` 目录，并填写项目实际信息：

```bash
mkdir -p /path/to/project/.ai
cp assets/LLM_RULES.md /path/to/project/.ai/
cp assets/LLM_BOUNDARY.md /path/to/project/.ai/
```

需要填写的内容：

- `LLM_RULES.md`：替换 `{{...}}` 占位符（MCU 型号、工具链、构建系统、RTOS 等）
- `LLM_BOUNDARY.md`：根据项目类型选择模板章节（STM32CubeMX / ESP-IDF / 裸机），填写实际文件路径

## 卸载

**Codex**

从 Codex 的 skill 目录删除 `embedded-llm-guardrails`。

**Claude Code**

```bash
rm -rf /path/to/project/.claude/skills/embedded-llm-guardrails
rm -f /path/to/project/CLAUDE.md
```

**Cursor**

```bash
rm -f /path/to/project/.cursor/rules/embedded-llm-guardrails.mdc
```

**删除目标项目共享规则**

```bash
rm -f /path/to/project/.ai/LLM_RULES.md
rm -f /path/to/project/.ai/LLM_BOUNDARY.md
```

## 更新

从本仓库拉取最新版本后，重新安装 skill 或重新复制对应入口文件：

```bash
git pull
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

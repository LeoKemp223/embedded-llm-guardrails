# Embedded LLM Guardrails

用于在嵌入式、单片机、RTOS、驱动和板级工程中使用 LLM 辅助开发时建立安全边界的技能模板。

## 核心原则

- **白名单制**：未显式允许修改的文件，默认禁止修改
- **最小补丁**：每次只做当前任务要求的最小可验证修改
- **TDD 闭环**：先定义验证方式 → 实现最小代码 → 验证通过后再扩展
- **保护已验证代码**：带 `VERIFIED_ON_HARDWARE` 等标记的代码不允许删除/重排/重构
- **新功能独立文件**：添加新功能时必须创建独立的 `.c` / `.h` 文件

## 项目结构

```
skills/embedded-llm-guardrails/
  SKILL.md                          技能主文件：执行顺序和硬规则
  assets/
    LLM_RULES.md                    嵌入式开发规则模板（含 {{...}} 占位符）
    LLM_BOUNDARY.md                 修改边界配置模板（STM32CubeMX / ESP-IDF / 裸机）
    llm-change-request.md           单次修改请求模板
  references/
    ide-entrypoints.md              多 IDE 接入方式
```

## 使用时生成的文件

安装到目标项目后，规则文件统一放在 `.claude/` 目录下：

```
/path/to/project/
  .claude/
    skills/embedded-llm-guardrails/   skill 本身（Claude Code）
    LLM_RULES.md                      规则文件（从模板填写）
    LLM_BOUNDARY.md                   边界配置（从模板填写）
  CLAUDE.md                           IDE 入口文件（引用上述规则）
```

`LLM_RULES.md` 中的 `{{...}}` 占位符需替换为项目实际信息（MCU 型号、工具链、构建系统等）。`LLM_BOUNDARY.md` 提供了 STM32CubeMX、ESP-IDF、裸机三种项目模板，根据项目类型选择并填写路径。

## 安装

### 1. 安装 Skill 到 IDE

将 skill 目录复制到目标项目对应的 IDE skill/rules 目录下：

**Claude Code**

```bash
# 项目级安装（仅当前项目生效）
mkdir -p /path/to/project/.claude/skills
cp -r skills/embedded-llm-guardrails /path/to/project/.claude/skills/

# 或用户级安装（所有项目生效）
mkdir -p ~/.claude/skills
cp -r skills/embedded-llm-guardrails ~/.claude/skills/
```

**Cursor**

```bash
mkdir -p /path/to/project/.cursor/rules
cp -r skills/embedded-llm-guardrails /path/to/project/.cursor/rules/
mv /path/to/project/.cursor/rules/embedded-llm-guardrails/SKILL.md \
   /path/to/project/.cursor/rules/embedded-llm-guardrails.mdc
```

**GitHub Copilot**

```bash
mkdir -p /path/to/project/.github
cp skills/embedded-llm-guardrails/SKILL.md /path/to/project/.github/copilot-instructions.md
cp -r skills/embedded-llm-guardrails/assets /path/to/project/.github/
cp -r skills/embedded-llm-guardrails/references /path/to/project/.github/
```

**Codex**

```bash
cp skills/embedded-llm-guardrails/SKILL.md /path/to/project/AGENTS.md
cp -r skills/embedded-llm-guardrails/assets /path/to/project/.claude/
cp -r skills/embedded-llm-guardrails/references /path/to/project/.claude/
```

**Windsurf**

```bash
cp skills/embedded-llm-guardrails/SKILL.md /path/to/project/.windsurfrules
cp -r skills/embedded-llm-guardrails/assets /path/to/project/.claude/
cp -r skills/embedded-llm-guardrails/references /path/to/project/.claude/
```

**Cline / Roo Code**

```bash
mkdir -p /path/to/project/.cline/rules
cp skills/embedded-llm-guardrails/SKILL.md /path/to/project/.cline/rules/embedded-llm-guardrails.md
cp -r skills/embedded-llm-guardrails/assets /path/to/project/.claude/
cp -r skills/embedded-llm-guardrails/references /path/to/project/.claude/
```

**Augment**

```bash
cp skills/embedded-llm-guardrails/SKILL.md /path/to/project/.augment-guidelines
cp -r skills/embedded-llm-guardrails/assets /path/to/project/.claude/
cp -r skills/embedded-llm-guardrails/references /path/to/project/.claude/
```

### 2. 配置目标项目

将模板文件复制到目标项目的 `.claude/` 目录，并填写项目实际信息：

```bash
mkdir -p /path/to/project/.claude
cp assets/LLM_RULES.md   /path/to/project/.claude/
cp assets/LLM_BOUNDARY.md /path/to/project/.claude/
```

需要填写的内容：

- `LLM_RULES.md`：替换 `{{...}}` 占位符（MCU 型号、工具链、构建系统、RTOS 等）
- `LLM_BOUNDARY.md`：根据项目类型选择模板章节（STM32CubeMX / ESP-IDF / 裸机），填写实际文件路径

### 3. 配置 IDE 入口文件

根据使用的 IDE 创建入口文件，引用 `.claude/` 下的规则：

| IDE | 入口文件 |
|-----|---------|
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursor/rules/embedded-llm-guardrails.mdc` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Codex | `AGENTS.md` |
| Windsurf | `.windsurfrules` |
| Cline / Roo Code | `.cline/rules/embedded-llm-guardrails.md` |
| Augment | `.augment-guidelines` |

入口文件模板见 `references/ide-entrypoints.md`。

## 卸载

**Claude Code**

```bash
rm -rf /path/to/project/.claude/skills/embedded-llm-guardrails
# 用户级
rm -rf ~/.claude/skills/embedded-llm-guardrails
```

**Cursor**

```bash
rm -rf /path/to/project/.cursor/rules/embedded-llm-guardrails
rm -f /path/to/project/.cursor/rules/embedded-llm-guardrails.mdc
```

**GitHub Copilot**

```bash
rm -f /path/to/project/.github/copilot-instructions.md
rm -rf /path/to/project/.github/assets
rm -rf /path/to/project/.github/references
```

**Codex**

```bash
rm -f /path/to/project/AGENTS.md
```

**Windsurf**

```bash
rm -f /path/to/project/.windsurfrules
```

**Cline / Roo Code**

```bash
rm -f /path/to/project/.cline/rules/embedded-llm-guardrails.md
```

**Augment**

```bash
rm -f /path/to/project/.augment-guidelines
```

**删除公共文件（所有 IDE 共用）**

```bash
rm -f /path/to/project/.claude/LLM_RULES.md
rm -f /path/to/project/.claude/LLM_BOUNDARY.md
rm -rf /path/to/project/.claude/assets
rm -rf /path/to/project/.claude/references
```

## 更新

从本仓库拉取最新版本后，重新执行安装步骤覆盖已有文件：

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

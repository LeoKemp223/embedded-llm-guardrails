# Embedded LLM Guardrails

用于在嵌入式、单片机、RTOS、驱动和板级工程中使用 LLM 辅助开发时建立安全边界的技能模板。

## 核心原则

- **白名单制**：未显式允许修改的文件，默认禁止修改
- **最小补丁**：每次只做当前任务要求的最小可验证修改
- **TDD 闭环**：先定义验证方式 → 实现最小代码 → 验证通过后再扩展
- **保护已验证代码**：带 `VERIFIED_ON_HARDWARE` 等标记的代码不允许删除/重排/重构

## 项目结构

```
skills/embedded-llm-guardrails/
  SKILL.md                          技能主文件：执行顺序和硬规则
  assets/
    LLM_RULES.md                    嵌入式开发规则模板
    LLM_BOUNDARY.md                 修改边界配置模板
    llm-change-request.md           单次修改请求模板
  references/
    ide-entrypoints.md              多 IDE 接入方式
```

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
cp -r skills/embedded-llm-guardrails/assets /path/to/project/
cp -r skills/embedded-llm-guardrails/references /path/to/project/
```

**Windsurf**

```bash
cp skills/embedded-llm-guardrails/SKILL.md /path/to/project/.windsurfrules
cp -r skills/embedded-llm-guardrails/assets /path/to/project/
cp -r skills/embedded-llm-guardrails/references /path/to/project/
```

### 2. 配置目标项目

在目标嵌入式项目根目录放置模板文件，并根据实际情况填写：

```bash
cp assets/LLM_RULES.md   /path/to/project/
cp assets/LLM_BOUNDARY.md /path/to/project/
```

填写 `LLM_BOUNDARY.md` 中的禁止/允许修改范围、已验证代码位置、主循环保护点等。

## 卸载

**Claude Code**

```bash
# 项目级
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
rm -rf /path/to/project/.github/copilot-instructions.md
rm -rf /path/to/project/.github/assets
rm -rf /path/to/project/.github/references
```

**Codex**

```bash
rm -f /path/to/project/AGENTS.md
rm -rf /path/to/project/assets
rm -rf /path/to/project/references
```

**Windsurf**

```bash
rm -f /path/to/project/.windsurfrules
rm -rf /path/to/project/assets
rm -rf /path/to/project/references
```

最后删除目标项目中的规则和边界文件：

```bash
rm /path/to/project/LLM_RULES.md
rm /path/to/project/LLM_BOUNDARY.md
```

## 更新

从本仓库拉取最新版本后，重新执行安装步骤覆盖已有文件：

```bash
git pull
# 然后重新执行对应 IDE 的安装命令
```

更新后检查目标项目的 `LLM_BOUNDARY.md` 是否需要根据项目变化重新配置。

## 许可证

MIT

---
name: embedded-llm-guardrails
description: 用于在已有嵌入式、单片机、RTOS、驱动、板级工程中使用 LLM 辅助开发时建立安全边界。适用于防止误改启动文件、BSP、外设驱动、主循环、ISR、已上板验证代码，并强制采用项目边界配置、最小可验证补丁、TDD 和硬件验证闭环。
---

# Embedded LLM Guardrails

本技能用于约束 LLM 在嵌入式项目中的代码修改行为。目标是让 LLM 在明确边界内生成最小补丁，而不是接管整个工程。

## 触发条件

当用户执行 `/embedded-llm-guardrails` 时触发。典型场景：

1. **首次在嵌入式项目中接入 LLM** — 初始化安全边界配置。
2. **审查或更新现有边界配置** — 项目结构变化后重新校准。
3. **在嵌入式项目中请求代码修改前** — 确保边界已就位。

## 初始化流程

### 第一步：检查项目现有配置

读取项目 `.ai/` 目录是否存在以下共享规则文件：

- `.ai/LLM_RULES.md`
- `.ai/LLM_BOUNDARY.md`

如果项目使用 Claude Code，可额外检查 `.claude/settings.json` 是否存在。它仅用于 Claude Code 权限配置，不作为 Codex 或 Cursor 的公共安装目录。

> **注意：** 如果项目根目录或旧版 `.claude/` 目录存在 `LLM_RULES.md` 和 `LLM_BOUNDARY.md`，提示用户迁移到 `.ai/` 目录下，并更新入口文件引用路径。

### 第二步：自动分析项目结构

无论是否存在配置文件，都必须先分析项目结构，识别：

1. **项目类型**（匹配以下任一）：
   - STM32CubeMX（查找 `.ioc` 文件、`Core/Src/`、`Drivers/STM32*`）
   - ESP-IDF（查找 `sdkconfig`、`components/`、`main/`）
   - NXP MCUXpresso（查找 `.mex` 文件、`board.h`）
   - 裸机 / 手写 Makefile（查找 `Makefile`、无 IDE 标识文件）
   - Keil / IAR（查找 `.uvprojx`、`.ewp`）
   - Arduino（查找 `.ino` 文件）
   - PlatformIO（查找 `platformio.ini`）

2. **MCU 信息**：型号、内核、时钟、Flash/RAM 大小

3. **关键目录和文件**：
   - 启动文件（`startup_*.s`、`startup_*.c`）
   - 链接脚本（`*.ld`、`*.icf`）
   - HAL/SDK 驱动目录
   - 应用代码目录
   - 构建系统文件
   - 主入口文件和主循环位置

4. **特殊机制**：
   - 代码生成器保护区域（如 CubeMX 的 `USER CODE BEGIN/END`）
   - 已验证代码标记
   - RTOS 使用情况

### 第三步：根据分析结果执行

**如果项目没有 `.ai/LLM_RULES.md` 和 `.ai/LLM_BOUNDARY.md`：**

1. 基于分析结果，从模板生成贴合项目实际结构的 `.ai/LLM_RULES.md` 和 `.ai/LLM_BOUNDARY.md`。
2. 模板中的示例路径必须替换为实际项目路径。
3. 根据项目类型添加特定章节（如 CubeMX 项目添加 USER CODE 保护说明）。
4. 在当前项目使用的 IDE/Agent 入口文件中添加引用（仅维护 Claude Code、Codex 和 Cursor）。
5. 如果项目使用 Claude Code，可生成 `.claude/settings.json` 权限配置（见下方说明）。
6. 向用户展示生成的文件，说明关键保护点，请用户确认或调整。

#### Claude Code settings.json 权限配置生成

仅当项目使用 Claude Code 时，根据项目类型和工具链自动生成 `.claude/settings.json`：

1. 从模板 `assets/settings.json` 读取基础配置。
2. 根据项目类型和检测到的工具，填充 `allow` 规则：
   - **通用**：`git *`、`ls *`、`cat *`、`where *`
   - **CMake 项目**：`cmake *`
   - **Makefile 项目**：`make *`
   - **检测到 OpenOCD**：`*openocd*`
   - **检测到 STM32CubeProgrammer**：`STM32_Programmer_CLI *`
   - **检测到 arm-none-eabi 工具链**：`arm-none-eabi-*`
   - **检测到 JLink**：`JLink.exe *`
3. 询问用户选择权限模式（`dontAsk` 或逐条白名单）。
4. 如果选择 `dontAsk` 模式，添加破坏性命令的 `deny` 列表。
5. 如果项目已有 `.claude/settings.json`，合并而非覆盖。

**如果项目已有配置文件：**

1. 读取现有配置，与当前项目结构对比。
2. 检查是否有新增文件或目录需要纳入边界管理。
3. 向用户报告差异，建议更新。

## 代码修改流程（每次任务）

处理嵌入式代码任务时，必须按顺序执行：

1. 强制重新读取项目规则：优先查看项目 `.ai/` 目录下的 `LLM_RULES.md` 和 `LLM_BOUNDARY.md`，不能只依赖历史上下文记忆。
2. 如果没有项目边界配置，停止修改代码，要求用户先运行初始化流程。
3. 检查长任务上下文状态；如果对规则、边界、用户最新目标或已完成修改不确定，先提醒用户执行 clear / compact 或重新给出当前任务摘要。
4. 判断需求是否复杂或模糊；如果是，先梳理需求、拆分最小目标、列出待确认问题，等用户确认。
5. 明确本次 `Allowed Files` 和 `Forbidden Files / Areas`；未显式允许的文件默认禁止修改。以文本格式向用户复述本次边界。
6. 识别已验证代码标记，例如 `VERIFIED_ON_HARDWARE`、`DO_NOT_TOUCH_TIMING`、`PRODUCTION_VERIFIED`。
7. 如果涉及外设驱动，先梳理开发流程、已知信息和缺失信息，等用户确认后再生成代码。确认硬件参数时，必须覆盖从 MCU 引脚到目标器件的完整信号路径，不能只确认终端器件就假设中间环节。未确认的环节视为未知参数，按「不假设」规则处理。
8. 如果涉及 `main` 入口或主 `while` 循环，只允许添加带注释的函数调用；复杂逻辑必须放到独立函数。
9. 如果项目使用代码生成器（如 CubeMX），确认修改在 `USER CODE` 区域内；如果需要修改生成区域，提醒用户通过代码生成器操作。
10. 按 TDD 思路先定义最小验证方式，再实现刚好能通过验证的最小代码。
11. 修改后说明变更文件、验证方式、硬件风险和下一步扩展前需要用户确认的内容。

## 长任务上下文管理

LLM 长时间运行后容易忘记或弱化规则。每次执行前必须重新读取规则文件，并根据用户输入判断是否需要提醒用户清理或压缩上下文。

需要提醒用户考虑 `clear` / 新会话的情况：

- 用户切换了目标、硬件平台、外设、工程目录或 IDE。
- 当前上下文中混有多个无关任务，容易污染判断。
- LLM 无法确定当前最新需求和历史需求哪个优先。
- 已经出现过违反边界、误改文件、删除代码或偏离最小验证目标。

需要提醒用户考虑 `compact` / 上下文压缩的情况：

- 当前任务已经经历多个阶段，规则、边界、验证结果散落在长对话中。
- 用户希望继续同一任务，但需要保留已确认的边界、决策、验证结果和剩余任务。
- 进入下一阶段前，需要把当前状态压缩成短摘要，避免后续遗忘。

提醒时不要直接停止工作过久，应给出明确选择：

```text
当前任务已经进入较长上下文。继续前建议：
1. compact：保留当前项目边界、已完成修改、验证结果和下一步目标。
2. clear：如果要切换目标或重新开始，清空上下文后重新加载规则。
3. 继续：我将重新读取 LLM_RULES.md 和 LLM_BOUNDARY.md，并只按本轮 Allowed Files 执行。
```

## 硬规则

- 每次执行前必须重新读取 `.ai/LLM_RULES.md` 和 `.ai/LLM_BOUNDARY.md`。
- 未配置允许修改范围时，不修改代码。
- 不把"实现这个功能"解释为允许修改任意文件。
- 不删除、重排、重构、优化已验证代码。
- 不为了测试删除 `main` 或主循环中的已有函数调用；只能临时屏蔽并备注原因、日期和恢复条件。
- 不在用户确认前自行假设外设型号、I2C 地址、SPI 模式、寄存器地址、初始化顺序等关键硬件参数。
- 最小功能验证通过前，不继续实现扩展模块。
- 扩展功能失败时，先回到已验证的最小功能，再重新拆分更小目标。
- 代码生成器保护的区域外代码，不直接修改；提醒用户通过生成器操作。
- 添加新功能时，必须创建独立的 `.c` 和 `.h` 文件实现功能，不允许把业务逻辑直接写在 `main.c` 或已有文件中。`main.c` 中只放函数调用。

## 需要读取的资源

- 项目级规则模板：`assets/LLM_RULES.md`
- 项目边界配置模板：`assets/LLM_BOUNDARY.md`
- 权限配置模板：`assets/settings.json`
- 单次任务模板：`assets/llm-change-request.md`
- 项目类型检测指南：`references/project-patterns.md`
- Claude Code / Codex / Cursor 接入方式：`references/ide-entrypoints.md`

## 建议落地方式

共享规则文件默认放在项目 `.ai/` 目录下，避免误用某个 IDE 的私有目录：

```text
.ai/
  LLM_RULES.md
  LLM_BOUNDARY.md
```

并在 IDE/Agent 入口文件中引用它们。当前仅维护 Claude Code、Codex 和 Cursor 的入口方式，参考 `references/ide-entrypoints.md`。

Claude Code 的 skill 本体仍安装到 `.claude/skills/`，Codex 入口放到 `AGENTS.md`，Cursor 入口放到 `.cursor/rules/embedded-llm-guardrails.mdc`。这些入口文件只引用 `.ai/` 下的同一套规则和边界文件。

> **迁移说明：** 如果项目根目录或旧版 `.claude/` 目录已有 `LLM_RULES.md` 和 `LLM_BOUNDARY.md`，初始化时应提示用户迁移到 `.ai/` 目录下，并更新所有引用路径。

---
name: embedded-llm-guardrails
description: 用于在已有嵌入式产品、单片机、RTOS、驱动、板级工程中使用 LLM 辅助开发时建立安全边界。适用于防止误改启动文件、BSP、外设驱动、主循环、ISR、已上板验证代码，并强制采用项目边界配置、修改前确认、最小可验证补丁和硬件验证闭环。
---

# Embedded LLM Guardrails

本技能用于约束 LLM 在嵌入式产品开发中的代码修改行为。目标是让 LLM 在用户确认的边界内生成最小补丁，并先完成最小验证闭环，而不是接管整个工程或一次性实现完整方案。

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
4. 在当前项目使用的 IDE/Agent 入口文件中添加引用（仅维护 Claude Code、Codex 和 Cursor）。生成的入口文件（`CLAUDE.md`、`AGENTS.md`、`.cursor/rules/*.mdc`）必须逐字使用 `references/ide-entrypoints.md` 中提供的中文模板，不得自行翻译为英文或重写为其它语言。
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

1. 首次使用预检：在任何代码修改、分析、回答之前，必须按当前 IDE/Agent 检查下列文件是否齐备：
   - `.ai/LLM_RULES.md`（核心 5 项标准确认单已存在）
   - `.ai/LLM_BOUNDARY.md`（Allowed Files / 禁改区域已填写）
   - 当前 IDE 的入口文件，且已填写「项目概述（芯片、工具链、构建系统、生成工具）」「安全边界」「构建」「烧录」四节：
     - Claude Code → `CLAUDE.md`
     - Codex → `AGENTS.md`
     - Cursor → `.cursor/rules/embedded-llm-guardrails.mdc`
2. 任一文件缺失或四节关键字段含 `{{...}}` 占位符未替换时，立即停止任何代码修改请求，向用户输出待办清单：
   - 列出缺失的文件和需要补全的字段
   - 提示运行 `/embedded-llm-guardrails` 进入初始化流程
   - 在用户补齐前，只允许只读分析（阅读文件、解释代码、回答架构问题）
3. 文件齐备后，强制重新读取 `.ai/LLM_RULES.md` 和 `.ai/LLM_BOUNDARY.md`，不能只依赖历史上下文记忆。
4. 检查长任务上下文状态；如果对规则、边界、用户最新目标或已完成修改不确定，先提醒用户执行 clear / compact 或重新给出当前任务摘要。
5. 判断需求是否涉及硬件行为、外设、GPIO、主入口、主循环、任务调度、中断、时序、功耗、构建文件或生产参数。只要涉及，先梳理需求、拆分最小目标、列出待确认问题，等用户确认。
6. 明确本次 `Allowed Files`；未显式允许的文件默认禁止修改，不再单列 Forbidden Files。以文本格式向用户复述 Allowed Files 范围。
7. 修改前强制确认：只要本轮会修改源码、头文件、构建脚本、配置文件或硬件行为，必须先输出「本轮任务确认单」并停止等待用户回复。用户明确回复“确认 / 继续 / 按这个做”或等价表达前，禁止编辑文件。
8. 「本轮任务确认单」字段定义见 `.ai/LLM_RULES.md` 中的标准确认单，核心 5 项：本轮目标、Allowed Files、最小验证方式、硬件风险、待确认问题。不要在此处再展开字段定义，避免与 `LLM_RULES.md` 出现表述差异。
9. 识别已验证代码标记，例如 `VERIFIED_ON_HARDWARE`、`DO_NOT_TOUCH_TIMING`、`PRODUCTION_VERIFIED`。
10. 如果涉及外设驱动，先梳理开发流程、已知信息和缺失信息，等用户确认后再生成代码。确认硬件参数时，必须覆盖从 MCU/SoC 引脚到目标器件的完整信号路径，不能只确认终端器件就假设中间环节。未确认的环节视为未知参数，按「不假设」规则处理。
11. 如果涉及 `main` 入口、主 `while` 循环、`app_main`、RTOS 任务入口或调度循环，只允许添加带注释的函数调用；复杂逻辑必须放到独立函数或独立模块。
12. 如果项目使用代码生成器（STM32CubeMX、NXP MCUXpresso、Arduino、PlatformIO 等）：
    a. **双路径确认**：当前需求若既可通过生成器配置（重新生成代码）实现，也可通过直接修改代码实现（例如「新增 SPI2 外设」「修改 UART 波特率」「加一个定时器中断」），必须先列出两条路径的优劣（可维护性、是否会被下次生成覆盖、配置完整性、上手成本、回滚难度），等用户明确选择后再继续，**不得自行替用户决定**。
    b. 选生成器路径：引导用户在 IDE 中配置并重新生成，LLM 仅协助审查 diff，不自行触发生成器。
    c. 选直接改代码路径：确认修改在用户代码保护区域内（如 CubeMX 的 `USER CODE BEGIN/END`）；需要修改保护区域外的代码时，立即停止并改用生成器路径。
13. 按 TDD 思路先定义最小验证方式，再实现刚好能通过验证的最小代码。不要因为发现可扩展空间就继续实现第二阶段功能。
14. 修改后说明变更文件、验证方式、硬件风险和下一步扩展前需要用户确认的内容。

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
- 涉及代码修改或硬件行为修改时，用户确认本轮任务确认单前，不调用写文件工具。
- 不把“合理默认值”视为用户确认；可以推荐默认方案，但必须等用户确认。
- 不删除、重排、重构、优化已验证代码。
- 不为了测试删除 `main` 或主循环中的已有函数调用；只能临时屏蔽并备注原因、日期和恢复条件。
- 不在用户确认前自行假设外设型号、I2C 地址、SPI 模式、寄存器地址、初始化顺序等关键硬件参数。
- 最小功能验证通过前，不继续实现扩展模块。
- 扩展功能失败时，先回到已验证的最小功能，再重新拆分更小目标。
- 代码生成器保护的区域外代码，不直接修改；提醒用户通过生成器操作。
- 添加新功能时，必须创建独立的 `.c` 和 `.h` 文件实现功能，不允许把业务逻辑直接写在 `main.c` 或已有文件中。`main.c` 中只放函数调用。
- 生成或更新 IDE/Agent 入口文件（`CLAUDE.md`、`AGENTS.md`、`.cursor/rules/*.mdc`）时，必须使用 `references/ide-entrypoints.md` 中提供的中文模板；不得擅自翻译为英文或重写为其它语言。
- 首次使用预检未通过时，禁止任何代码修改请求；只允许只读分析。预检包括 `.ai/LLM_RULES.md`、`.ai/LLM_BOUNDARY.md` 和当前 IDE 入口文件四节（项目概述、安全边界、构建、烧录）的存在与字段填写完整性。
- 代码生成器项目中，需求若同时存在生成器配置路径和直接代码路径，必须先与用户确认走哪条路径；禁止自行决定路径。

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

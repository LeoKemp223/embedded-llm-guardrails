---
name: embedded-llm-guardrails
description: 用于在已有嵌入式、单片机、RTOS、驱动、板级工程中使用 LLM 辅助开发时建立安全边界。适用于防止误改启动文件、BSP、外设驱动、主循环、ISR、已上板验证代码，并强制采用项目边界配置、最小可验证补丁、TDD 和硬件验证闭环。
---

# Embedded LLM Guardrails

本技能用于约束 LLM 在嵌入式项目中的代码修改行为。目标是让 LLM 在明确边界内生成最小补丁，而不是接管整个工程。

## 执行顺序

处理嵌入式代码任务时，必须按顺序执行：

1. 读取项目规则：优先查看项目根目录的 `LLM_RULES.md` 和 `LLM_BOUNDARY.md`。
2. 如果没有项目边界配置，停止修改代码，要求用户先配置允许修改和禁止修改范围。
3. 判断需求是否复杂或模糊；如果是，先梳理需求、拆分最小目标、列出待确认问题，等用户确认。
4. 明确本次 `Allowed Files` 和 `Forbidden Files / Areas`；未显式允许的文件默认禁止修改。
5. 识别已验证代码标记，例如 `VERIFIED_ON_HARDWARE`、`DO_NOT_TOUCH_TIMING`、`PRODUCTION_VERIFIED`。
6. 如果涉及外设驱动，先梳理开发流程、已知信息和缺失信息，等用户确认后再生成代码。
7. 如果涉及 `main` 入口或主 `while` 循环，只允许添加带注释的函数调用；复杂逻辑必须放到独立函数。
8. 按 TDD 思路先定义最小验证方式，再实现刚好能通过验证的最小代码。
9. 修改后说明变更文件、验证方式、硬件风险和下一步扩展前需要用户确认的内容。

## 硬规则

- 未配置允许修改范围时，不修改代码。
- 不把“实现这个功能”解释为允许修改任意文件。
- 不删除、重排、重构、优化已验证代码。
- 不为了测试删除 `main` 或主循环中的已有函数调用；只能临时屏蔽并备注原因、日期和恢复条件。
- 不在用户确认前自行假设外设型号、I2C 地址、SPI 模式、寄存器地址、初始化顺序等关键硬件参数。
- 最小功能验证通过前，不继续实现扩展模块。
- 扩展功能失败时，先回到已验证的最小功能，再重新拆分更小目标。

## 需要读取的资源

- 项目级规则模板：`assets/LLM_RULES.md`
- 项目边界配置模板：`assets/LLM_BOUNDARY.md`
- 单次任务模板：`assets/llm-change-request.md`
- 多 IDE 接入方式：`references/ide-entrypoints.md`

## 建议落地方式

在实际嵌入式项目根目录放置：

```text
LLM_RULES.md
LLM_BOUNDARY.md
```

并在 IDE/Agent 入口文件中引用它们。不同 IDE 的入口文件参考 `references/ide-entrypoints.md`。

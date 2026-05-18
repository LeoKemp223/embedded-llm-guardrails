# LLM 修改边界配置

本文件由用户在项目接入 LLM 辅助开发前填写，用于防止 LLM 修改不允许修改的位置。

## 执行原则

- 未显式允许修改的文件，默认禁止修改。
- 每次任务必须列出 `Allowed Files`。
- 如果任务需要修改禁止区域，必须单独请求用户确认。
- 用户确认必须具体到文件或目录，不能泛化成整个工程可改。
- 已上板验证、生产验证、时序敏感代码默认只读。

## 绝对禁止修改

```text
# 示例
startup/
linker.ld
bsp/clock/
bsp/pinmux/
vendor/
bootloader/
production_config/
```

## 默认允许修改

```text
# 示例
app/new_feature/
tests/
debug/
docs/
```

## 需要用户单独确认后才能修改

```text
# 示例
main.c
app/main_loop.c
drivers/
rtos_tasks.c
interrupts.c
flash_storage.c
ota/
```

## 已验证代码位置

```text
# 示例
drivers/motor/motor_start_sequence.c
app/control/verified_loop.c
```

保护标记：

```text
VERIFIED_ON_HARDWARE
DO_NOT_TOUCH_TIMING
LOGIC_ANALYZER_VERIFIED
OSCILLOSCOPE_VERIFIED
PRODUCTION_VERIFIED
```

## 主入口和主循环保护点

```text
# 示例
main.c: system_init()
main.c: app_normal_task()
main.c: feed_watchdog()
main.c: communication_process()
```

规则：

- 不删除已有调用。
- 不重排已有调用。
- 测试代码只允许添加带注释的函数调用。
- 调试时只允许临时屏蔽，并备注原因、日期和恢复条件。

## 测试代码允许放置位置

```text
# 示例
tests/
debug/
app/test_hooks/
```

## 生成代码允许放置位置

```text
# 示例
app/generated/
app/new_feature/
```

## 本次任务边界

每次让 LLM 修改代码前，用户应填写：

```text
Allowed Files:
- 

Forbidden Files / Areas:
- 

Protected Verified Code:
- 

Temporary Debug Allowed:
- yes/no

Need User Confirmation Before Touching:
- 
```


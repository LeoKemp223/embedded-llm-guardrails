# 项目类型检测指南

本文件指导 LLM 如何通过文件特征自动识别嵌入式项目类型，以便生成贴合实际的边界配置。

## 检测方法

按优先级依次检查以下特征文件，匹配到第一个即确定项目类型。

## STM32CubeMX

**特征文件：**
- `*.ioc` 文件存在
- `Core/Src/` 和 `Core/Inc/` 目录存在
- `Drivers/STM32*` 目录存在
- 源文件中有 `USER CODE BEGIN` / `USER CODE END` 标记

**关键信息提取：**
- MCU 型号：从 `.ioc` 文件读取 `Mcu.Name`
- 时钟配置：从 `.ioc` 文件读取时钟树配置，或从 `main.c` 的 `SystemClock_Config()` 读取
- Flash/RAM：从链接脚本 `*.ld` 读取 `MEMORY` 区域
- 已启用外设：从 `.ioc` 文件读取 `Mcu.IPx` 行
- 引脚配置：从 `.ioc` 文件读取 `Px.GPIOParameters` 和 `Px.Signal`

**边界配置要点：**
- `USER CODE` 区域内允许修改，区域外禁止修改
- `.ioc` 文件只读（引导用户通过 CubeMX 操作）
- HAL 驱动和 CMSIS 目录绝对禁止修改
- CubeMX 重新生成后需检查边界是否需要更新

## ESP-IDF

**特征文件：**
- `sdkconfig` 文件存在
- `main/` 目录存在
- `CMakeLists.txt` 包含 `include($ENV{IDF_PATH}/tools/cmake/project.cmake)` 或类似引用
- `components/` 目录存在

**关键信息提取：**
- 芯片型号：从 `sdkconfig` 读取 `CONFIG_IDF_TARGET`
- Flash/PSRAM 大小：从 `sdkconfig` 读取
- 已启用组件：从 `sdkconfig` 读取 `CONFIG_*` 相关配置

**边界配置要点：**
- `sdkconfig` 只读（引导用户通过 `idf.py menuconfig` 操作）
- ESP-IDF 组件目录只读
- `main/` 和项目自定义 `components/` 允许修改

## NXP MCUXpresso

**特征文件：**
- `*.mex` 文件存在
- `board.h` 或 `pin_mux.h` 存在
- `CMSIS/` 或 `drivers/` 目录包含 NXP 相关文件

**关键信息提取：**
- MCU 型号：从 `.mex` 文件或 `board.h` 读取
- 已启用外设：从 `.mex` 文件读取

**边界配置要点：**
- 生成代码的保护区域（类似 CubeMX 的 USER CODE）
- NXP SDK 驱动目录只读
- 引脚和时钟配置引导用户通过 MCUXpresso Config Tools 操作

## Keil MDK

**特征文件：**
- `*.uvprojx` 文件存在
- `*.uvoptx` 文件存在
- `RTE/` 目录存在

**关键信息提取：**
- MCU 型号：从 `.uvprojx` 读取 `Device` 字段
- 源文件列表：从 `.uvprojx` 读取 `IncludeFile` 或 `SourceFile`

**边界配置要点：**
- Keil 项目文件只读
- RTE（Run-Time Environment）目录只读
- CMSIS 和 HAL 驱动目录只读

## IAR Embedded Workbench

**特征文件：**
- `*.ewp` 文件存在
- `*.eww` 文件存在
- `settings/` 目录包含 IAR 配置

**关键信息提取：**
- MCU 型号：从 `.ewp` 读取 `device` 或 `MCU` 节点

**边界配置要点：**
- IAR 项目文件只读
- 厂商驱动目录只读

## Arduino

**特征文件：**
- `*.ino` 文件存在
- 无标准嵌入式项目标识文件

**关键信息提取：**
- 目标板：从代码或 `platformio.ini` 读取（如果同时是 PlatformIO 项目）

**边界配置要点：**
- `*.ino` 文件允许修改
- 第三方库目录只读
- 较简单的项目，但仍需遵守最小修改原则

## PlatformIO

**特征文件：**
- `platformio.ini` 文件存在
- `lib/` 和 `src/` 目录存在

**关键信息提取：**
- 平台和框架：从 `platformio.ini` 读取 `[env:]` 下的 `platform` 和 `framework`
- MCU：从 `platformio.ini` 读取 `board`

**边界配置要点：**
- `platformio.ini` 核心配置需用户确认后修改
- `.pio/` 目录（依赖缓存）只读
- `lib/` 中的第三方库只读
- `src/` 允许修改

## 裸机 / 通用

当以上特征文件均不存在时，按裸机项目处理。

**检测步骤：**
1. 查找 `Makefile`、`CMakeLists.txt` 或其他构建文件。
2. 查找启动文件（`startup_*.s`、`startup_*.c`、`crt0*.s`）。
3. 查找链接脚本（`*.ld`、`*.icf`）。
4. 查找厂商 SDK 或 HAL 目录。
5. 查找 `main.c` 或等效入口文件。

**边界配置要点：**
- 启动文件和链接脚本只读
- 厂商 SDK/HAL 只读
- 应用代码目录允许修改
- 需要用户确认哪些是已验证代码

## 信息提取通用方法

### MCU 信息

| 来源 | 提取方式 |
|------|----------|
| 链接脚本 `*.ld` | `MEMORY` 区域获取 Flash/RAM 大小和地址 |
| `.ioc` 文件 | `Mcu.Name` 行 |
| `sdkconfig` | `CONFIG_IDF_TARGET` |
| 设备头文件 | `#define` 中查找 Flash/RAM 相关宏 |
| `main.c` | `SystemClock_Config()` 获取时钟频率 |

### 构建系统

| 构建 | 特征 |
|------|------|
| CMake + Ninja | `CMakeLists.txt` + `CMakePresets.json` |
| Makefile | `Makefile` 存在 |
| Keil | `.uvprojx` 存在 |
| IAR | `.ewp` 存在 |
| ESP-IDF | `sdkconfig` + `CMakeLists.txt` 含 IDF 引用 |
| PlatformIO | `platformio.ini` 存在 |

### 主入口和主循环

| 模式 | 位置 |
|------|------|
| 裸机 super-loop | `main.c` → `while(1)` |
| FreeRTOS | `main.c` → `osKernelStart()` 之前的初始化，以及各 task 函数 |
| Zephyr | `main.c` 或 `main()` 函数 |
| ESP-IDF | `main/` → `app_main()` |

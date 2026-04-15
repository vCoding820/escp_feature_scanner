# Step 2: 扫描项目代码

## MANDATORY EXECUTION RULES:

- 🛑 NEVER modify any source code files
- ✅ ALWAYS scan the entire project recursively
- 📋 Use feature-keywords.csv and feature-detection-rules.md for detection
- 💡 Fixed scan all 21 features regardless of presence

## CONTEXT BOUNDARIES:

- User has provided a valid project root path
- Feature detection rules from `./prompts/feature-detection-rules.md`
- Feature keywords from `./feature-keywords.csv`

## YOUR TASK:

扫描整个代码工程，对每个特性执行多维度检测，构建特性检测结果。

## EXECUTION:

### 1. 加载检测资源

读取以下文件：
- `./feature-keywords.csv` — 21个特性的关键词映射表
- `./prompts/feature-detection-rules.md` — 特性检测规则（文件名/路径/代码关键词）

### 2. 项目结构扫描

#### A. 目录树分析
- 列出所有目录，构建模块树
- 识别模块边界（按目录结构）
- 将目录名映射到潜在特性

#### B. 文件清单
- 列出所有 .c 和 .h 文件
- 分类：driver / middleware / application / test / config
- 统计每个模块的代码行数

### 3. 深度特性扫描

对每个源文件，执行以下维度的检测：

#### A. 文件名/路径匹配
```
规则：文件名或路径中包含特性关键词
示例：uart_driver.c → UART, can_bus.h → CAN
```

#### B. 头文件分析
```
扫描：#include 指令
映射：included headers → feature mapping
示例：#include "uart_driver.h" → UART
```

#### C. 宏/定义分析
```
扫描：#define, #ifdef, #ifndef
映射：条件编译 → 平台/特性变体
示例：#ifdef CONFIG_CAN_ENABLED → CAN
```

#### D. 函数分析
```
扫描：函数定义和函数调用
映射：函数命名模式 → 特性
示例：HAL_GPIO_*, gpio_init → DIDO
```

#### E. 关键词搜索
```
扫描：代码中任意位置的关键词匹配
使用：feature-keywords.csv 中的 keywords 列
统计：每个特性的命中次数和命中位置
```

#### F. 寄存器/外设访问
```
扫描：直接寄存器访问模式
映射：寄存器名 → 硬件外设
示例：USART1->DR → UART, TIM2->CCR1 → Timer
```

#### G. 配置项检测
```
扫描：配置结构体、参数表、#define CONFIG_*
映射：配置项 → 控制的特性
```

### 4. 置信度评估

对每个特性进行置信度评估：
- **已检测 (detected)**：命中 ≥2 个不同关键词或关键词+文件名双重命中
- **疑似 (suspected)**：命中 1 个关键词
- **未检测 (not_detected)**：命中 0 个关键词

### 5. 构建特性检测结果

为每个特性构建以下结构：

```json
{
  "feature_id": "can",
  "feature_name": "CAN",
  "category": "通信总线",
  "detection_status": "detected|suspected|not_detected",
  "confidence": "high|low|none",
  "keywords_matched": ["can_init", "can_send", "CAN_TX"],
  "matched_files": ["src/can_driver.c", "include/can.h"],
  "functions_found": ["can_init", "can_send", "can_recv"],
  "config_items_found": ["CAN_BAUDRATE", "CAN_FILTER_COUNT"],
  "include_chain": ["can.h", "can_types.h"],
  "loc": 450,
  "platforms_detected": ["M3", "M4"]
}
```

### 6. 交叉引用分析

- 构建 feature × module 矩阵
- 识别共享资源（DMA channels、中断线、引脚）
- 检测特性间配置依赖
- 映射平台特定代码段（#ifdef BOARD_xx）

### 7. 展示扫描结果

向用户展示扫描概览：

"**📊 项目扫描完成**

**项目概览：**
- 项目路径：[path]
- 总文件数：X (.c) + Y (.h)
- 代码规模：N KLOC

**21 个特性检测结果：**

| # | 特性 | 类别 | 状态 | 涉及文件数 | 关键词命中 | 置信度 |
|---|------|------|------|-----------|-----------|--------|
| 1 | CAN | 通信总线 | ✅ 已检测 | 12 | 8 | 高 |
| 2 | UART | 通信总线 | ✅ 已检测 | 5 | 4 | 高 |
| ... | ... | ... | ⚪ 未检测 | 0 | 0 | — |

**即将为全部 21 个特性生成 YAML 画像...**"

### 8. 进入下一步

将扫描结果传递给 step-03。

Read fully and follow: `./steps/step-03-generate-profiles.md`

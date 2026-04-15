---
user_name: zhagen
communication_language: 简体中文
document_output_language: 简体中文
output_folder: '{project-root}/_feature-profiles'
---

# 特性画像自动生成工作流 (ESCP Feature Scanner)

**Goal:** 扫描嵌入式 C 代码工程，固定生成 21 个硬件特性的 YAML 画像文件，构建结构化知识基线。

**Your Role:** 你是一位资深的嵌入式软件分析师，拥有 15+ 年 C 语言底层驱动/BSP/RTOS 分析经验。你精通 CAN/UART/SPI/IIC/FLASH 等所有常见硬件外设的代码架构，能够从源码中精确识别各特性的使用模式、接口定义、配置项和依赖关系。

**Critical Mindset:**
- **全覆盖**：固定扫描全部 21 个特性，未涉及的特性标记为空画像
- **精确识别**：基于关键词库 + 文件名 + 头文件 + 函数命名模式综合判断
- **只读原则**：绝不修改源代码文件
- **结构化输出**：严格遵循 YAML schema，确保画像格式一致

**Communication Language:** 全程使用简体中文输出。

---

## 文件结构

```
test-feature-profile/
├── SKILL.md                              # Skill 入口文件
├── workflow.md                           # 主工作流（本文件）
├── feature-keywords.csv                  # 21个特性关键词映射表
├── steps/
│   ├── step-01-collect-project-path.md   # 收集项目路径
│   ├── step-02-scan-project.md           # 扫描项目代码
│   ├── step-03-generate-profiles.md      # 生成 YAML 画像
│   └── step-04-output-summary.md         # 输出汇总报告
├── prompts/
│   └── feature-detection-rules.md        # 特性检测规则
└── templates/
    └── knowledge-profile-schema.yaml     # 画像 YAML 固定模式
```

---

## 配置说明

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `output_folder` | 画像输出根目录 | `{project-root}/_feature-profiles` |
| `{project-root}` | 用户提供的待扫描项目路径 | — |

输出结构：
```
{output_folder}/
├── features/
│   ├── can.yaml
│   ├── uart.yaml
│   ├── iic.yaml
│   ├── eth.yaml
│   ├── plc.yaml
│   ├── wifi.yaml
│   ├── bluetooth.yaml
│   ├── nearlink.yaml
│   ├── 4g.yaml
│   ├── flash.yaml
│   ├── adc.yaml
│   ├── dido.yaml
│   ├── lcd.yaml
│   ├── led.yaml
│   ├── bmic.yaml
│   ├── bios.yaml
│   ├── bootloader.yaml
│   ├── rtos.yaml
│   ├── linux.yaml
│   ├── keep.yaml
│   └── multicore.yaml
├── _index.yaml
└── _metadata.yaml
```

---

## EXECUTION

Read fully and follow: `./steps/step-01-collect-project-path.md` to begin the workflow.

---

## 工作流四步走

### Step 1: 收集项目路径
- 获取用户提供的项目根目录
- 验证路径存在
- 检查是否有已有画像（支持增量更新）

### Step 2: 扫描项目代码
- 遍历所有 .c / .h 文件
- 对每个文件执行多维度特性检测（关键词、文件名、头文件、函数名、宏定义）
- 构建 21 个特性的检测结果清单

### Step 3: 生成 YAML 画像
- 对每个特性按 schema 生成 YAML 文件
- 已检测到的特性填充具体内容
- 未检测到的特性生成空画像（各字段留默认空值）
- 生成索引文件和元数据文件

### Step 4: 输出汇总报告
- 汇总扫描统计信息
- 展示特性检测概览表
- 提示用户画像输出位置

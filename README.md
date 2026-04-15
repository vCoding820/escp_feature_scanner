# ESCP_FEATURE_SCANNER — 特性画像自动生成 Skill

> 扫描嵌入式 C 代码工程，固定生成 21 个硬件特性的 YAML 画像文件，构建结构化知识基线。  
> 全自动执行 · 只读扫描 · 固定 21 特性 · YAML 标准化输出

---

## 系统架构

```mermaid
graph TB
    subgraph 输入
        A[C 代码工程路径]
    end

    subgraph 扫描引擎
        B1[Step 1: 收集项目路径<br/>验证有效性]
        B2[Step 2: 扫描项目代码<br/>多维度特性检测]
        B3[Step 3: 生成 YAML 画像<br/>严格遵循 Schema]
        B4[Step 4: 输出汇总报告<br/>统计与概览]
    end

    subgraph 检测资源
        C1[feature-keywords.csv<br/>21个特性关键词]
        C2[feature-detection-rules.md<br/>5维检测规则]
        C3[knowledge-profile-schema.yaml<br/>画像固定模式]
    end

    subgraph 输出物
        D1[features/*.yaml<br/>21个特性画像]
        D2[_index.yaml<br/>特性索引]
        D3[_metadata.yaml<br/>扫描元数据]
        D4[控制台汇总报告]
    end

    A --> B1
    B1 --> B2
    B2 --> B3
    B3 --> B4
    C1 & C2 --> B2
    C3 --> B3
    B3 --> D1 & D2 & D3
    B4 --> D4
```

---

## 工作流程

```mermaid
flowchart LR
    Start([用户提供<br/>项目路径]) --> S1[验证路径<br/>检查 .c/.h]
    S1 --> Check{已有画像?}
    Check -->|是| Choice{全量 or 增量?}
    Check -->|否| S2
    Choice -->|全量| S2[扫描全部代码]
    Choice -->|增量| S2i[扫描变更部分]
    S2 & S2i --> S3[生成 21 个<br/>YAML 画像]
    S3 --> S4[输出汇总报告]
    S4 --> End([完成])
```

---

## 多维度特性检测

```mermaid
graph TD
    SRC[源代码文件] --> D1[A: 文件名/路径匹配<br/>uart_driver.c → UART]
    SRC --> D2[B: #include 头文件分析<br/>include uart.h → UART]
    SRC --> D3[C: 宏/条件编译分析<br/>ifdef CONFIG_CAN → CAN]
    SRC --> D4[D: 函数定义/调用分析<br/>HAL_GPIO_Init → DIDO]
    SRC --> D5[E: 关键词全文搜索<br/>feature-keywords.csv]
    SRC --> D6[F: 寄存器/外设访问<br/>USART1→DR → UART]
    SRC --> D7[G: 配置项检测<br/>CONFIG_* → 特性]

    D1 & D2 & D3 & D4 & D5 & D6 & D7 --> EVAL[置信度评估]

    EVAL --> R1[detected ✅<br/>≥2 关键词命中]
    EVAL --> R2[suspected ⚠️<br/>1 关键词命中]
    EVAL --> R3[not_detected ⚪<br/>0 命中]
```

---

## YAML 画像结构

```mermaid
graph LR
    subgraph 特性画像 YAML
        F[feature<br/>名称/ID/类别]
        DET[detection<br/>状态/置信度/命中词]
        SRC[source_files<br/>驱动/头文件/中间件/测试]
        FN[functions<br/>公开API/中断/内部函数]
        CFG[config_items<br/>名称/类型/默认值/范围]
        PLT[platform_support<br/>M0/M3/M4/A7/...]
        DEP[dependencies<br/>依赖/被使用/共享资源]
        TC[test_concerns<br/>关键/常规/平台特定]
        CM[code_metrics<br/>LOC/函数数/复杂度]
        META[metadata<br/>版本/时间/来源]
    end

    F --- DET --- SRC --- FN --- CFG --- PLT --- DEP --- TC --- CM --- META
```

### 画像字段说明

| 模块 | 字段 | 说明 |
|------|------|------|
| `feature` | name / id / category | 特性基本标识 |
| `detection` | status / confidence / keywords_matched | 检测结果与置信度 |
| `source_files` | drivers / headers / middleware / configs / tests | 关联源文件清单 |
| `functions` | public_api / interrupt_handlers / internal | 函数接口列表 |
| `config_items` | name / type / default / range / impact | 配置项及影响 |
| `platform_support` | M0 / M3 / M4 / A / A7 / HI1230 / multi_core | 平台支持矩阵 |
| `dependencies` | requires / used_by / shared_resources | 依赖与资源共享 |
| `test_concerns` | critical / common / platform_specific | 测试关注点 |
| `code_metrics` | loc / functions_count / complexity | 代码度量 |

---

## 21 个固定特性

```mermaid
mindmap
  root((21 个硬件特性))
    通信总线
      CAN
      UART
      IIC
      ETH
    通信
      PLC
    无线通信
      WIFI
      蓝牙
      星闪
      4G
    存储
      FLASH
    外设
      ADC
      DIDO
      LCD
      LED
    采集
      BMIC
    系统软件
      BIOS
      Bootloader
      RTOS
      Linux
      Keep
    多核
      多核通信
```

---

## 检测与生成规则

```mermaid
flowchart TD
    subgraph 对每个特性
        A[加载关键词<br/>feature-keywords.csv] --> B[多维度扫描<br/>7种检测方式]
        B --> C{命中数}
        C -->|≥2| D[status: detected<br/>confidence: high]
        C -->|=1| E[status: suspected<br/>confidence: low]
        C -->|=0| F[status: not_detected<br/>confidence: none]
        D & E --> G[填充画像数据<br/>源文件/函数/配置项]
        F --> H[生成空画像<br/>字段留默认值]
        G & H --> I[写入 feature_id.yaml]
    end
```

> **核心规则**：无论代码是否涉及，全部 21 个特性均生成画像文件。未涉及的特性画像字段留默认空值。

---

## 输出目录结构

```
{project-root}/_feature-profiles/
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
├── _index.yaml          # 特性索引（快速概览）
└── _metadata.yaml       # 扫描元数据
```

---

## Skill 目录结构

```
escp_feature_scanner/
├── SKILL.md                              # Skill 入口
├── workflow.md                           # 主工作流
├── README.md                             # 本文件
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

## 与 ESCP_TEST_DESIGN 的协作

```mermaid
sequenceDiagram
    participant User as 用户
    participant Scanner as ESCP_FEATURE_SCANNER
    participant Sync as Windows 任务计划
    participant TestDesign as ESCP_TEST_DESIGN

    User->>Scanner: 提供 C 代码工程路径
    Scanner->>Scanner: 扫描代码、生成 21 个 YAML 画像
    Scanner-->>User: 输出汇总报告

    loop 每日定时
        Sync->>Scanner: 读取 _feature-profiles/features/*.yaml
        Sync->>TestDesign: 同步到 features/*.yaml
    end

    User->>TestDesign: 提供 MR/diff/源码
    TestDesign->>TestDesign: 加载 *.md 测试知识 + *.yaml 画像
    TestDesign->>TestDesign: 画像数据增强测试因子
    TestDesign-->>User: 输出测试设计报告
```

### 数据流转

```mermaid
graph LR
    A[C 代码工程] -->|扫描| B[ESCP_FEATURE_SCANNER]
    B -->|生成| C[YAML 画像文件]
    C -->|定时同步| D[ESCP_TEST_DESIGN/features/]
    E[人工测试知识 *.md] --> F{测试生成引擎}
    D --> F
    F -->|输出| G[测试设计报告]

    style C fill:#e3f2fd
    style E fill:#e8f5e9
```

- **蓝色**：自动生成的画像数据（机器产出，定时更新）
- **绿色**：人工编写的测试知识（经验积累，手动维护）

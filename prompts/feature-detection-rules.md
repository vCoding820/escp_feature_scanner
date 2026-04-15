# 特性检测规则

本文件定义了从代码、文件名、关键词中识别 21 个硬件特性的规则集。

---

## 1. 检测维度

对每个源文件，执行以下 5 个维度的检测：

| 维度 | 说明 | 权重 |
|------|------|------|
| 文件名/路径匹配 | 文件名或目录名包含特性关键词 | 高 |
| #include 头文件 | 引用了特性相关的头文件 | 高 |
| 函数定义/调用 | 代码中出现特性相关的函数名 | 高 |
| 宏定义/条件编译 | #define/#ifdef 中包含特性关键词 | 中 |
| 代码文本关键词 | 代码任意位置出现 feature-keywords.csv 中的关键词 | 中 |

---

## 2. 文件名/路径匹配规则（不区分大小写）

### 通信总线
| 特性 | 文件名关键词 |
|------|-------------|
| CAN | `can`, `canfd` |
| UART | `uart`, `usart`, `serial`, `console` |
| IIC | `i2c`, `iic`, `smbus` |
| ETH | `eth`, `ethernet`, `enet`, `mac`, `phy` |

### 通信
| 特性 | 文件名关键词 |
|------|-------------|
| PLC | `plc`, `powerline`, `carrier` |

### 无线通信
| 特性 | 文件名关键词 |
|------|-------------|
| WIFI | `wifi`, `wlan` |
| 蓝牙 | `bt`, `ble`, `bluetooth` |
| 星闪 | `nearlink`, `sparklink`, `sle` |
| 4G | `lte`, `4g`, `gsm`, `nbiot`, `cellular` |

### 存储
| 特性 | 文件名关键词 |
|------|-------------|
| FLASH | `flash`, `nvm`, `fmc` |

### 外设
| 特性 | 文件名关键词 |
|------|-------------|
| ADC | `adc` |
| DIDO | `gpio`, `pin`, `port`, `dido`, `digital` |
| LCD | `lcd`, `display`, `gui` |
| LED | `led` |

### 采集
| 特性 | 文件名关键词 |
|------|-------------|
| BMIC | `bmic`, `battery`, `bms` |

### 系统软件
| 特性 | 文件名关键词 |
|------|-------------|
| BIOS | `bios`, `self_test`, `post` |
| Bootloader | `boot`, `bootloader`, `uboot`, `ota`, `fota` |
| RTOS | `rtos`, `freertos`, `task`, `thread`, `os` |
| Linux | `linux`, `driver`, `module`, `dts` |
| Keep | `keep`, `keepalive`, `heartbeat` |

### 多核
| 特性 | 文件名关键词 |
|------|-------------|
| 多核通信 | `ipc`, `mbox`, `mailbox`, `shared_mem`, `rpmsg` |

---

## 3. 代码关键词匹配规则

使用 `feature-keywords.csv` 中的 keywords 列，对代码内容进行全文搜索。

匹配策略：
- 以单词边界匹配（避免误匹配子串）
- 不区分大小写
- 统计命中次数和命中行号

---

## 4. 头文件→特性映射表

| 头文件模式 | 映射特性 |
|-----------|---------|
| `*can*.h` | CAN |
| `*uart*.h`, `*usart*.h`, `*serial*.h` | UART |
| `*i2c*.h`, `*iic*.h` | IIC |
| `*eth*.h`, `*lwip*.h`, `*netif*.h` | ETH |
| `*plc*.h` | PLC |
| `*wifi*.h`, `*wlan*.h` | WIFI |
| `*bt*.h`, `*ble*.h`, `*bluetooth*.h` | 蓝牙 |
| `*nearlink*.h`, `*sle*.h` | 星闪 |
| `*lte*.h`, `*cellular*.h`, `*gsm*.h` | 4G |
| `*flash*.h`, `*nvm*.h` | FLASH |
| `*adc*.h` | ADC |
| `*gpio*.h`, `*dido*.h` | DIDO |
| `*lcd*.h`, `*display*.h` | LCD |
| `*led*.h` | LED |
| `*bmic*.h`, `*bms*.h`, `*battery*.h` | BMIC |
| `*bios*.h` | BIOS |
| `*boot*.h`, `*ota*.h` | Bootloader |
| `*freertos*.h`, `*rtos*.h`, `*os*.h` | RTOS |
| `*linux*.h`, `*module*.h` | Linux |
| `*keepalive*.h`, `*heartbeat*.h` | Keep |
| `*ipc*.h`, `*mailbox*.h`, `*rpmsg*.h` | 多核通信 |

---

## 5. 平台检测规则

通过条件编译宏和目录名检测目标平台：

| 平台 | 条件编译宏 | 目录关键词 |
|------|-----------|-----------|
| M0 | `CORE_M0`, `STM32F0`, `__CORTEX_M0` | `m0/`, `cortex-m0/` |
| M3 | `CORE_M3`, `STM32F1`, `STM32F2`, `__CORTEX_M3` | `m3/`, `cortex-m3/` |
| M4 | `CORE_M4`, `STM32F4`, `STM32L4`, `__CORTEX_M4` | `m4/`, `cortex-m4/` |
| A | `CORE_A`, `CORTEX_A`, `__CORTEX_A` | `cortex-a/`, `a-core/` |
| A7 | `CORE_A7`, `__CORTEX_A7` | `a7/`, `cortex-a7/` |
| RTOS_A7 | `RTOS_A7`, `FREERTOS_A7` | `rtos-a7/` |
| HI1230 | `HI1230`, `HI123X` | `hi1230/`, `hi123x/` |

---

## 6. 置信度评估规则

| 条件 | 置信度 | 检测状态 |
|------|--------|----------|
| 文件名匹配 + ≥2个代码关键词 | 高 | detected |
| ≥2个不同代码关键词 | 高 | detected |
| 1个文件名匹配 或 1个代码关键词 | 低 | suspected |
| 0个匹配 | 无 | not_detected |

---

## 7. 特性交互检测

当同时检测到以下特性组合时，记录交互关系：

| 组合 | 关注点 |
|------|--------|
| CAN + RTOS | 多任务下CAN收发竞态 |
| UART + DMA | DMA传输完成回调时序 |
| FLASH + Bootloader | OTA升级时的Flash擦写保护 |
| ADC + DMA | DMA搬运ADC数据的完整性 |
| 多核通信 + RTOS | 核间任务同步和锁竞争 |
| ETH + RTOS | TCP/IP协议栈线程安全 |
| BMIC + ADC | 电池电压采集精度 |

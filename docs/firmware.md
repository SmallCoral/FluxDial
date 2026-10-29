# 固件参考

当前构建目标是传感器产品固件，入口为 [main.c](../Firmware/Core/Src/main.c)。
本页的配置与行为来自提交 `953b748` 中的源码；编译和烧录操作见 [上手](getting-started.md)。

## 采样与调度

| 项目 | 当前配置 | 源码 |
| --- | --- | --- |
| 系统时钟 | 24MHz HSE 作为参考，PSIS 配置为 144MHz，各总线分频为 1 | [system_clock.c](../Firmware/Core/Src/system_clock.c) |
| 温度采样 | ADC1_IN0，12 位；ADC 时钟 9MHz，289 周期采样；每次平均 16 次转换 | [board_sensors.c](../Firmware/BSP/Src/board_sensors.c) |
| 温度更新 | 主循环每 50ms 执行一次 | [main.c](../Firmware/Core/Src/main.c) |
| 流量输入 | EXTI1 上升沿；TIM2 以 1MHz 计时，保存最近 8 个有效周期 | [board_sensors.c](../Firmware/BSP/Src/board_sensors.c) |
| 流量快照与显示 | 主循环每 200ms 更新；调度和通信耗时会影响实际时刻 | [main.c](../Firmware/Core/Src/main.c) |
| USER 按键 | 每 10ms 轮询，候选状态稳定后确认一次按下事件 | [board_user_button.c](../Firmware/BSP/Src/board_user_button.c) |
| 显示通信 | I2C1 目标速率 100kHz，HT16K33 七位地址 0x70 | [board_i2c.c](../Firmware/BSP/Src/board_i2c.c)、[ht16k33.c](../Firmware/BSP/Src/ht16k33.c) |
| ADC 重试 | 初始化或采样失败后，每 1000ms 尝试初始化 | [main.c](../Firmware/Core/Src/main.c) |
| 显示重试 | 显示失败后，每 500ms 尝试恢复总线并初始化驱动 | [main.c](../Firmware/Core/Src/main.c) |
| 独立看门狗 | 128 分频、重装载 2000，名义约 8 秒；实际超时取决于 LSI，调试暂停时冻结 | [board_sensors.c](../Firmware/BSP/Src/board_sensors.c) |

这些数值是软件配置，不代表本轮用仪器测得的实际时钟或时序。
引脚和接线只在 [硬件参考](hardware.md#接口与-mcu-引脚)维护。

## 温度处理

温度路径为 ADC 平均值 → 原始故障检查 → 连续状态确认 → 1/8 IIR 滤波 → 阻温表插值。
换算在 [sensor_model.c](../Firmware/App/Src/sensor_model.c) 中实现。

```text
Rntc = 49900 × (4095 / ADC - 1) Ω
```

激励与 ADC 参考共用 +3V3。
实际代码用整数运算和舍入处理，随后对厂商 Rnor 表按 1℃ 间隔线性插值。
表覆盖 −30～105℃；B25/50 参数用于描述探头，当前代码没有用 B 值公式替代该表。

| 条件 | 状态 |
| --- | --- |
| 尚未确认稳定状态 | 启动 |
| ADC ≤ 8 | 开路 |
| ADC ≥ 4087 | 短路 |
| 阻值超出表的高阻端 / 低阻端 | 过冷 / 过热 |
| ADC 初始化或读取失败 | ADC 故障 |

同一候选状态连续出现 3 次后生效，故障判断在滤波之前进行。
有效读数内部以 0.1℃ 保存，显示时按选择的温标换算和四舍五入。
探头范围、显示数字范围与整机测量精度是不同概念；整机精度尚未标定。

## 流量与累计量

软件采用厂商标称 `F = 11Q`，其中 F 为 Hz，Q 为 L/min，即每升 660 个脉冲。
周期换算保留 mL/min 分辨率，显示时四舍五入为整数 L/min。

中断中小于 1500µs 的重复上升沿被计为拒绝脉冲。
连续 2000ms 无有效脉冲后，流动状态失效，并在后续显示更新中归零。
停流后收到第一个脉冲时尚无完整周期，需要后续有效脉冲才能形成周期读数。

累计量按接受的脉冲换算为毫升，使用 RAM 中的 64 位累计计数。
当前屏幕不轮播累计量，也不将其写入 Flash。
没有独立的传感器在线信号，静止水流和未接流量传感器都会显示零。

## 显示与异常状态

段码生成见 [product_display.c](../Firmware/App/Src/product_display.c)。

| 内容 | 当前行为 |
| --- | --- |
| 温标 | 默认 ℃，USER 短按切换 ℃/℉，上电恢复 ℃ |
| 上方数字 | 当前温标下显示整数，数字能力为 −9～199 |
| 下方数字 | 瞬时流量；超过 99L/min 的显示能力时显示 HI |
| 启动 | 上方显示 `--` |
| 开路、短路或 ADC 故障 | 上方显示 `Er`，并点亮告警 |
| 过冷 / 过热或显示数字越界 | 显示 `Lo` / `HI` |
| 环形段 | 按摄氏温度 0～100℃ 映射为 9 段，与温标选择无关 |
| 热水提示 | 达到 40℃ 点亮，低于 38℃ 熄灭 |
| 正常测温时的高温告警 | 达到 80℃ 点亮，低于 78℃ 熄灭 |
| 流量标签 | 流量标签常亮，收到有效流动状态时显示工作图标 |

例如 100℃ 换算为 212℉，超过上方数字能力，切换到华氏时显示 `HI`。
这属于显示边界处理，不代表温度传感器报 ADC 故障。

I²C 恢复会释放 SDA、按需发送最多 9 个 SCL 脉冲并产生 STOP，然后重新配置外设。
显示失败期间主循环继续采集传感器并更新诊断数据。
若 HSE 或 PSIS 在启动阶段未就绪，程序停在初始化失败循环；该阶段尚未启用看门狗，没有实现备用时钟自动启动。

## 调试变量

[product_diagnostics.h](../Firmware/Core/Inc/product_diagnostics.h) 定义 `g_product_diagnostics`。
通过当前 ELF 的符号定位变量，不要沿用旧固件的固定 RAM 地址。

| 字段 | 含义与单位 |
| --- | --- |
| `uptime_ms` | 运行毫秒 |
| `adc_raw` / `adc_online` / `adc_errors` | 原始 ADC 值、初始化状态与错误累计 |
| `ntc_status` / `temperature_c10` | 温度状态与 0.1℃ 数值 |
| `flow_ml_min` / `flow_period_us` | mL/min 流量与平均周期 µs |
| `pulses` / `rejected_pulses` | 接受与拒绝的脉冲计数 |
| `total_ml_high` / `total_ml_low` | 累计毫升数的高低 32 位 |
| `display_online` / `display_errors` / `display_updates` | 显示状态、错误与成功刷新累计 |
| `fahrenheit` | 当前温标选择 |

`ntc_status` 的枚举顺序为 0 正常、1 启动、2 开路、3 短路、4 过冷、5 过热、6 ADC 故障。
具体定义见 [sensor_model.h](../Firmware/App/Inc/sensor_model.h)。
历史 J-Link 观察过程见 [2026-10-06 记录](../Documentation/Reports/firmware/VALIDATION.md)。

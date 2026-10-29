# 硬件参考

本页记录当前 [原理图](../Hardware/FluxDial.kicad_sch)与 [PCB](../Hardware/FluxDial.kicad_pcb)中的连接和器件值。
本轮审查以提交 `953b748` 为基线，核对了原理图网表、PCB 焊盘网络及固件引脚配置。
这些信息描述工程文件，不替代实际装配、测量或生产检查。

## 关键器件

| 位号 | 器件或设计值 | 职责与依据 |
| --- | --- | --- |
| U401 | STM32C562CET6，LQFP48 | 主控；[DS14927](../Documentation/Datasheets/STM32C562xx_DS14927_Rev2.pdf)给出 Cortex-M33、最高 144MHz、512KiB Flash 和 128KiB SRAM |
| U101 / U102 | AP63203WU-7 / AP63205WU-7 | 原理图中的 +3V3 / +5V 两路降压电源 |
| L101 / L102 | 3.9µH / 4.7µH | 两路电源电感；其他额定参数应核对实际采购型号 |
| U201 | HT16K33，SOP28 | LED 扫描驱动，见 [HT16K33 手册](../Documentation/Datasheets/HT16K33_Rev110.pdf) |
| DS201 | 5858-1DRWB-10 | 58mm 共阴红白蓝圆屏，见 [屏幕规格书](../Documentation/Datasheets/20250303远帆5858-1DRWB-10%281%29.pdf) |
| Q201 / Q202 | BSS138 | I²C 双向电平转换 |
| U301 | SN74LVC1G17DCKR | 流量脉冲施密特缓冲，见 [TI 手册](../Documentation/Datasheets/SN74LVC1G17.pdf) |
| D301 | BAV199 | NTC 输入节点钳位 |
| Y401 | 24MHz，CL=10pF | Abracon ABM8 无源晶振，见 [晶体资料](../Documentation/Datasheets/Abracon_ABM8_Rev20200729.pdf) |
| SW101 / SW102 / SW301 | B3U-1000P | BOOT、RESET 和 USER 按键 |

本地资料没有完整收录 AP63203/AP63205 和 Type-C 连接器的原厂手册。
本页不由器件名称推导整板额定输入范围、输出电流或转换效率。

## 供电

J101 采用 C668623 六针 Type-C 封装，仅连接 VBUS、GND、CC1 和 CC2。
当前工程规定从 J101 输入 **5V**，没有 USB D+/D− 数据线路。
CC1、CC2 分别由 R101、R102 的 5.1kΩ 电阻下拉。

F101 标注为 0.5A/24V，自恢复保险丝后的 `VIN_SYS` 向两路 DCDC 供电。
该器件标注不代表整板允许输入 24V，也不代表整板可持续输出 0.5A。

| 网络 | 关联器件 | 当前原理图中的电容 | 主要负载 |
| --- | --- | --- | --- |
| +3V3 | U101、L101 | 输入 10µF/35V，输出 2×22µF/10V，自举 100nF | MCU、ADC 参考、NTC 与逻辑 |
| +5V | U102、L102 | 输入 10µF/35V，输出 2×22µF/10V，自举 100nF | HT16K33 与流量传感器 |

`+5V` 是网络名和设计目标，不能据此保证 5V 输入时仍输出严格稳压的 5V。
本轮没有执行电压、压降、带载能力或温升测量。
工程中的“后级按 5～12V 设计”注释不构成从现有 J101 输入 12V 的使用说明。

## 接口与 MCU 引脚

以下端子编号以原理图和 PCB 的焊盘编号为准。
线色来自工程约定，接线前仍需核对实际线束；不要按照片方向猜测端子序号。

| 接口 | 引脚 | 网络 / 定义 | MCU 关系 |
| --- | --- | --- | --- |
| J101 | A9 / B9 | VBUS，5V 电源输入 | 经 F101 进入 VIN_SYS |
| J101 | A5 / B5 | CC1 / CC2 | 各接 5.1kΩ 下拉 |
| J101 | A12 / B12 / SH | GND / 外壳 | 地 |
| J102 | 1 | VREF / +3V3 | 目标电压检测 |
| J102 | 2 | SWDIO | PA13，U401-34 |
| J102 | 3 | SWCLK | PA14，U401-37 |
| J102 | 4 | GND | 地 |
| J301 | 1 | +3V3 | 外部 NTC 上端 |
| J301 | 2 | NTC 信号 | 经 R302 接 PA0，U401-10 |
| J302 | 1 | +5V，红线 | 流量传感器供电 |
| J302 | 2 | 脉冲，黄线 | 经调理电路接 PA1，U401-11 |
| J302 | 3 | GND，白线 | 地 |

J102 是 JST XH 2.50mm 四针接口，不含 NRST。
VREF 供调试器检测目标电压，整板通过 J101 单独供电。
J301、J302 分别是同间距的两针和三针接口。

| 板内功能 | MCU 名称 | LQFP48 脚号 | 当前用途 |
| --- | --- | --- | --- |
| NTC | PA0 | 10 | ADC1_IN0 |
| 流量 | PA1 | 11 | EXTI1 上升沿，TIM2 提供时间戳 |
| USER | PC13 | 2 | 低有效 GPIO 输入 |
| 显示时钟 | PB6 | 42 | I2C1 SCL，AF4 |
| 显示数据 | PB7 | 43 | I2C1 SDA，AF4 |
| HSE | PH0 / PH1 | 5 / 6 | 外部晶振输入 / 输出 |
| 启动选择 | PH2-BOOT0 | 44 | R103 下拉，SW101 拉高 |
| 复位 | NRST | 7 | R104 上拉、C104 到地，SW102 拉低 |
| USB 数据引脚 | PA11 / PA12 | 32 / 33 | 当前空接 |

引脚命名同时核对 [LQFP48 引脚表](../Documentation/Datasheets/STM32C562CET6_LQFP48引脚表.csv)。
固件实际选择见 [board_sensors.c](../Firmware/BSP/Src/board_sensors.c)、[board_i2c.c](../Firmware/BSP/Src/board_i2c.c)和 [board_user_button.c](../Firmware/BSP/Src/board_user_button.c)。

## MCU 电源与时钟

| 引脚 | 网络 | 外围 |
| --- | --- | --- |
| VDD 24 / 36 / 48 | +3V3 | C401～C403 各 100nF |
| VSS 23 / 35 / 47 | GND | 地 |
| VCAP 22 | MCU_VCAP | C405 2.2µF/10V 到地，不能接 +3V3 或外部负载 |
| VREF+ 9 | +3V3 | C406 100nF 旁路 |
| VREF− 8 | GND | 地 |

C404 为 4.7µF/10V 总去耦。
VCAP 电容的具体 ESR 与布局要求以 [DS14927](../Documentation/Datasheets/STM32C562xx_DS14927_Rev2.pdf)和 [AN6274](../Documentation/Datasheets/AN6274_STM32C5_Hardware_Rev1.pdf)为准。

Y401 的 1、3 脚为晶体端，2、4 脚接地。
**当前 C407、C408 的器件值均为 6.8pF/C0G**，原理图与 PCB 一致。
归档 BOM 和 MCU 页的一处说明文字仍写 12pF；实际装配及是否需要调校尚待人工确认。
本轮没有修改这些硬件或 BOM 文件。

板上没有 LSE 晶振，PC14、PC15 当前空接。
如后续增加 LSE，应重新核对 [ES0661 勘误](../Documentation/Datasheets/ES0661_STM32C562_Rev1.pdf)。
时钟软件配置见 [固件参考](firmware.md#采样与调度)。

## 传感器调理

NTC 使用 [DCSL-503F3950-170CB 规格书](../Documentation/Datasheets/180长规格书%281%29.pdf)，标称 R25=50kΩ、B25/50=3950K。
外部 NTC 位于 +3V3 与采样节点之间，R301 为 49.9kΩ/0.1% 下拉。
R302 1kΩ、C301 100nF 构成 ADC 输入滤波，D301 将输入钳位至电源轨附近。
阻温表使用范围与软件换算见 [温度处理](firmware.md#温度处理)。

流量信号由 R303 10kΩ 上拉至 +5V，经过 R304 4.7kΩ、C302 1nF 和 U301 转换为 3.3V 逻辑。
厂商文件为 [BTL-004A 规格书](../Documentation/Datasheets/BTL-004A涡轮霍尔规格书%281%29.pdf)。
同一文件在频率检验条件中使用 2～12L/min，在另一处参数表中列出 1～25L/min，范围表述不一致。
本项目尚未通过通水确定可用量程或整机误差。

## 显示连接

Q201/Q202 在 3.3V 与 5V 两侧转换 I²C 电平，两侧均有 4.7kΩ 上拉。
HT16K33 使用 ROW0～ROW9 和 COM0～COM5，未使用的输出在原理图中空接。

| HT16K33 输出 | 串联电阻 | 屏幕段线 | DS201 脚号 |
| --- | --- | --- | --- |
| ROW0 | R201 | A | 15 |
| ROW1 | R202 | B | 14 |
| ROW2 | R203 | C | 13 |
| ROW3 | R204 | D | 12 |
| ROW4 | R205 | E | 11 |
| ROW5 | R206 | F | 10 |
| ROW6 | R207 | G | 9 |
| ROW7 | R208 | H | 8 |
| ROW8 | R209 | I | 7 |
| ROW9 | R210 | J | 16 |

R201～R210 均为 330Ω；COM0～COM5 依次连接 DS201 的 COM1～COM6，即屏幕 1～6 脚。
屏幕尺寸和矩阵可查看 [规格书预览](../Images/reference/display-spec-mechanics-and-matrix.png)，显示逻辑见 [固件参考](firmware.md#显示与异常状态)。

## PCB 与封装

当前 PCB 为双铜层、1.6mm 厚，走线中存在 0.20、0.60 和 0.80mm 三种宽度。
文件中存在 0.50/0.30mm 与 0.60/0.30mm 两种过孔外径/钻孔组合。
这些是当前图形尺寸，不是完整的加工公差或制板能力声明。
检查结果见 [本次硬件检查](validation.md#本次检查)。

常规阻容使用 0603，DCDC 储能电容使用 1206。
项目自建资源保存在 [Hardware/libraries](../Hardware/libraries/)，库名为 `FluxDial`。
符号表、封装表和自建 STEP 模型均使用 `${KIPRJMOD}` 相对路径。

| 封装 | 当前库中的关键尺寸 | 核对资料 |
| --- | --- | --- |
| STM32C562 LQFP48 | 本体 7×7mm，脚距 0.50mm，焊盘 1.20×0.30mm，相对焊盘中心距 8.50mm | [MCU 尺寸图](../Documentation/Reports/hardware-review/mcu_footprint_dimensioned.pdf) |
| ABM8 | 焊盘 1.30×1.05mm，中心距 2.30×1.75mm | [晶振封装](../Hardware/libraries/FluxDial.pretty/Abracon_ABM8_3.2x2.5mm.kicad_mod) |
| 圆屏 | 排距 35.08mm，针距 2.54mm，焊盘 1.8mm，孔径 1.0mm，1 脚为方形 | [圆屏对位图](../Documentation/Reports/hardware-review/display_footprint_1to1.pdf) |

历史尺寸图、3D 预览和 [交互式 BOM](../Documentation/Reports/interactive-bom/ibom.html)用于辅助核对。
它们没有在本轮重新生成，不能替代当前工程及实物装配确认。

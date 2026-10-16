# FluxDial

一块基于 STM32C562CET6 的温度、流量采集与圆形 LED 显示板。硬件连接 50kΩ NTC 温度探头、BTL-004A 霍尔流量传感器和 58mm 共阴 LED 屏，使用 HT16K33 完成显示扫描，程序通过 J-Link SWD 下载和调试。

硬件工程使用 KiCad 10，项目符号、封装和 3D 模型随仓库保存。

<p align="center">
  <img src="Images/photos/assembled-board-powered.jpg" width="680" alt="FluxDial 成品板点亮效果">
</p>

## 当前状态

- 硬件已完成制板、焊接和点亮验证。
- `Firmware/` 中的产品固件采集真实温度与流量，支持摄氏/华氏切换、热水提示、告警和传感器故障显示。
- NTC 实际采样、温标切换和显示已在样板验证；流量中断/计时/停流逻辑已通过板上脉冲测试，实际通水和精度标定待完成，见 [验证记录](Documentation/Reports/firmware/VALIDATION.md)。
- KiCad 工程、项目库、器件资料、交互式 BOM、实物照片和演示视频均随仓库保存。

## 演示

以下视频记录早期显示演示固件，不代表当前产品固件的真实测量值。点击下面的封面可播放约 25 秒的 H.264 演示视频。视频展示 `00 → 100 → 00` 数值往返、环形进度、热水/告警区域以及按键切换温标。

<p align="center">
  <a href="Images/video/display-demo.mp4">
    <img src="Images/video/display-demo-poster.jpg" width="360" alt="点击播放显示演示视频">
  </a>
</p>

<p align="center">
  <a href="Images/video/display-demo.mp4">播放兼容版 MP4</a>
  ·
  <a href="Images/video/display-demo-original-hevc.mp4">下载原始 HEVC 视频</a>
</p>

| 未安装圆屏的 PCB 正面 | 暗环境显示效果 |
| --- | --- |
| <img src="Images/photos/assembled-board-front.jpg" width="420" alt="焊接完成的 PCB 正面"> | <img src="Images/photos/display-lit-dark.jpg" width="420" alt="暗环境下的圆形 LED 显示效果"> |

## 硬件组成

| 模块 | 器件与参数 |
| --- | --- |
| 主控 | STM32C562CET6，Cortex-M33，最高 144MHz，512KB Flash，128KB SRAM |
| 主控封装 | LQFP48，7×7mm，0.5mm 脚距 |
| 系统时钟 | 24MHz Abracon ABM8 无源晶振 |
| 3.3V 电源 | AP63203WU-7 同步降压，3.9µH，2×22µF 输出电容 |
| 5V 电源 | AP63205WU-7 同步降压，4.7µH，2×22µF 输出电容 |
| 温度输入 | 50kΩ NTC，B25/50=3950K，49.9kΩ 精密分压与 ADC 滤波 |
| 流量输入 | BTL-004A 霍尔传感器，SN74LVC1G17 施密特缓冲 |
| 显示驱动 | HT16K33，5V 供电，I²C 接口 |
| 电平转换 | BSS138 双向 I²C 电平转换，3.3V ↔ 5V |
| 显示屏 | 5858-1DRWB-10，58mm 共阴圆形 LED 屏 |
| 调试接口 | JST XH 4P，VREF / SWDIO / SWCLK / GND |
| 按键 | Omron B3U-1000P，BOOT、RESET、USER |

常规电阻和电容使用 0603 封装，DCDC 的 10µF/22µF 储能电容使用 1206 封装。

## 电源

J101 使用 SHOU HAN TYPE-C 6P(073)，LCSC 编号 `C668623`。接口只连接 VBUS、GND、CC1 和 CC2，不包含 D+、D−，因此仅用于供电。

- J101 额定电压为 5V，不能从该接口输入 12V。
- CC1、CC2 分别通过 5.1kΩ 电阻下拉。
- F101 为 0.5A/24V 自恢复保险丝。
- 保险丝后的 `VIN_SYS` 同时连接 3.3V 和 5V 两路 DCDC。
- AP63203 和 AP63205 的输入范围为 3.8～32V；若从其他连接器接入 5～12V，该连接器及保护器件需具有相应额定电压。

## 接口定义

| 接口 | 引脚 | 定义 |
| --- | --- | --- |
| J101 | A9/B9 | 5V VBUS |
| J101 | A5/B5 | CC1/CC2 |
| J101 | A12/B12 | GND |
| J102 | 1 | VREF / +3V3 |
| J102 | 2 | SWDIO |
| J102 | 3 | SWCLK |
| J102 | 4 | GND |
| J301 | 1 | +3V3 |
| J301 | 2 | NTC 信号 |
| J302 | 1 | +5V，红线 |
| J302 | 2 | 流量脉冲，黄线 |
| J302 | 3 | GND，白线 |

J102 的 VREF 用于 J-Link 检测目标电平，不作为整板电源输入。接口不包含 NRST，复位使用板上的 RESET 按键。

## MCU 引脚分配

| 功能 | MCU 引脚 | LQFP48 脚号 |
| --- | --- | --- |
| NTC ADC | PA0 | 10 |
| 流量脉冲 | PA1 | 11 |
| 用户按键 | PC13 | 2 |
| I²C SCL | PB6 | 42 |
| I²C SDA | PB7 | 43 |
| SWDIO | PA13 | 34 |
| SWCLK | PA14 | 37 |
| BOOT0 | PH2-BOOT0 | 44 |
| NRST | NRST | 7 |
| HSE 输入/输出 | PH0/PH1 | 5/6 |

PA11、PA12 未连接 USB 数据线路。

## 仓库结构

```text
FluxDial/
├── README.md
├── Documentation/                # 设计说明和器件清单
│   ├── Datasheets/               # 器件手册和 MCU 官方资料包
│   └── Reports/                  # 固件验证、硬件检查与交互式 BOM
├── Firmware/                     # STM32C562 固件、测试和构建工具
├── Hardware/
│   ├── FluxDial.kicad_pro        # KiCad 工程入口
│   ├── FluxDial.kicad_sch        # 原理图主页面
│   ├── FluxDial.kicad_pcb        # PCB
│   ├── mcu_minimum.kicad_sch     # MCU 最小系统
│   ├── usb_power.kicad_sch       # 电源、按键和 SWD
│   ├── display.kicad_sch         # 显示驱动
│   ├── sensors.kicad_sch         # 温度与流量接口
│   ├── libraries/                # FluxDial 符号、封装和 3D 模型
│   └── tools/                    # 网表与 PCB 检查脚本
└── Images/                       # 实物照片、演示视频和资料页图片
```

## 打开工程

```bash
git clone https://github.com/SmallCoral/STM32-Sensor-Display.git FluxDial
cd FluxDial
```

使用 KiCad 10 打开 [Hardware/FluxDial.kicad_pro](Hardware/FluxDial.kicad_pro)。项目自建库通过相对路径引用，克隆仓库后无需单独安装符号库或封装库。

固件使用 GNU Arm Embedded 工具链构建：

```bash
cd Firmware
make -j
```

更完整的行为说明、构建输出和烧录提示见 [固件说明](Firmware/README.md)。

## 文档

- [固件说明](Firmware/README.md)
- [KiCad 工程说明](Hardware/README.md)
- [硬件设计说明](Documentation/DESIGN_NOTES.md)
- [STM32C562 最小系统](Documentation/C562_MIGRATION.md)
- [晶振与封装说明](Documentation/HSE_FOOTPRINT_A2.md)
- [器件清单](Documentation/BOM.csv)
- [交互式 BOM](Documentation/Reports/interactive-bom/ibom.html)
- [STM32C562 官方资料包](Documentation/Datasheets/STM32C562CET6_2026-09-16/00_README_中文.md)
- [照片与视频索引](Images/README.md)
- [圆屏 1:1 对位图](Documentation/Reports/hardware-review/display_footprint_1to1.pdf)
- [MCU 封装尺寸图](Documentation/Reports/hardware-review/mcu_footprint_dimensioned.pdf)

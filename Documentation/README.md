# 器件资料与历史记录

当前维护的设计与使用说明已集中到 [docs](../docs/README.md)。
本目录保留原厂资料、既有器件清单和历史导出，不再重复维护接口与固件参数。

| 资料 | 内容与边界 |
| --- | --- |
| [Datasheets](Datasheets/) | 芯片、驱动器、温度探头、流量传感器与显示屏手册 |
| [STM32C562 官方资料包](Datasheets/STM32C562CET6_2026-09-16/00_README_中文.md) | 官方 PDF、来源和校验值 |
| [BOM.csv](BOM.csv) | 既有器件清单；C407/C408 等内容须与当前工程核对 |
| [Reports](Reports/README.md) | 既有实板验证、检查结果和交互式 BOM |
| [MCU 尺寸图](Reports/hardware-review/mcu_footprint_dimensioned.pdf) | 历史封装核对资料 |
| [圆屏对位图](Reports/hardware-review/display_footprint_1to1.pdf) | 历史封装核对资料 |

## 常用手册

| 器件 | 资料 |
| --- | --- |
| STM32C562 | [DS14927 Rev 2](Datasheets/STM32C562xx_DS14927_Rev2.pdf)、[AN6274](Datasheets/AN6274_STM32C5_Hardware_Rev1.pdf)、[ES0661](Datasheets/ES0661_STM32C562_Rev1.pdf) |
| 温度探头 | [180 长规格书](Datasheets/180长规格书%281%29.pdf) |
| 流量传感器 | [BTL-004A](Datasheets/BTL-004A涡轮霍尔规格书%281%29.pdf) |
| 圆形 LED 屏 | [5858-1DRWB-10](Datasheets/20250303远帆5858-1DRWB-10%281%29.pdf) |
| 显示驱动 | [HT16K33](Datasheets/HT16K33_Rev110.pdf) |
| 流量缓冲 | [SN74LVC1G17](Datasheets/SN74LVC1G17.pdf) |

旧文档入口继续保留为导航。

- [原硬件设计说明](DESIGN_NOTES.md)
- [原 STM32C562 最小系统说明](C562_MIGRATION.md)
- [原晶振与封装说明](HSE_FOOTPRINT_A2.md)

资料授权状态见 [许可证与第三方资料](../docs/licenses.md)。

# FluxDial 固件

本目录构建当前的传感器产品固件。
硬件数据流和软件职责见 [系统架构](../docs/architecture.md)。

| 目标 | 入口 |
| --- | --- |
| 安装工具、构建、测试与烧录 | [上手](../docs/getting-started.md) |
| 查采样、换算、阈值、显示状态和诊断变量 | [固件参考](../docs/firmware.md) |
| 查传感器、I²C、按键和 SWD 引脚 | [硬件参考](../docs/hardware.md#接口与-mcu-引脚) |
| 查主机测试和实板验证范围 | [验证记录](../docs/validation.md) |
| 查 CMSIS 与 LL 来源和许可证 | [驱动说明](Drivers/README.md) |

## 代码入口

| 路径 | 内容 |
| --- | --- |
| [Core](Core/) | 启动、时钟、调度与主循环 |
| [BSP](BSP/) | 板级 ADC、流量、计时、按键、I²C 与显示驱动 |
| [App](App/) | 传感器换算和显示帧 |
| [tests/test_product.c](tests/test_product.c) | 主机计算与显示测试 |
| [tools](tools/) | Windows 构建、测试和烧录脚本 |
| [Makefile](Makefile) | ARM 目标与主机测试入口 |

照片和演示视频中的测试动画不属于当前构建目标。

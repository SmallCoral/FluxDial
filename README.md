# FluxDial

基于 STM32C562CET6、50kΩ NTC 和霍尔流量传感器的圆形 LED 显示板。

固件采集温度与瞬时流量，通过 HT16K33 驱动 58mm 共阴 LED 屏，并支持温标切换、热水提示和故障显示。
J101 的 USB-C 接口仅用于 **5V 供电**，程序通过 J102 的 J-Link SWD 接口下载。

```sh
git clone https://github.com/SmallCoral/FluxDial.git
cd FluxDial
```

<p align="center">
  <img src="Images/photos/assembled-board-powered.jpg" width="360" alt="FluxDial 样板安装圆屏后的早期显示演示照片">
</p>

照片记录早期显示演示，画面数值不能用于判断传感器精度。
[更多实物照片与演示视频](Images/README.md)。

## 目录

- [项目状态](#项目状态)
- [功能与系统](#功能与系统)
- [上手](#上手)
- [仓库结构](#仓库结构)
- [文档导航](#文档导航)
- [已知问题](#已知问题)
- [文档维护](#文档维护)
- [许可证](#许可证)

## 项目状态

| 范围 | 已有证据与边界 |
| --- | --- |
| 固件 | 包含 NTC 采样、流量脉冲计时、显示及异常恢复；本次完整编译与主机测试通过 |
| 样板 | 2026-10-06 的记录确认了 NTC 读数、温标切换和板上脉冲测试；本次没有重新烧录或测量实板 |
| 实际水路 | 实际通水、流量标定和整机测量精度尚未验证 |
| 硬件检查 | 当前 KiCad 检查仍有 ERC 警告和 DRC 错误，不能据此直接宣称已通过制板检查 |

测试版本、检查结果和待确认事项集中在 [验证记录](docs/validation.md)。

## 功能与系统

- 上方显示 NTC 温度，USER 按键切换 ℃/℉。
- 下方显示瞬时流量，停流后归零。
- 支持热水提示、告警、温度探头异常及数值超限显示。
- 显示通信和 ADC 初始化失败后重试，主循环运行期间使用独立看门狗。
- 累计流量保存在 RAM，可通过调试器读取；断电后清零。

信号路径为“传感器与调理电路 → STM32 采集与换算 → HT16K33 → 圆形 LED 屏”。
完整框图及代码分层见 [系统架构](docs/architecture.md)，行为阈值见 [固件参考](docs/firmware.md)。

## 上手

1. 用 KiCad 10 打开 [Hardware/FluxDial.kicad_pro](Hardware/FluxDial.kicad_pro)。
2. 按 [硬件接口表](docs/hardware.md#接口与-mcu-引脚)核对传感器和 SWD 接线。
3. 准备 Arm GNU Toolchain、GNU Make 和 J-Link，按 [编译、烧录与首次使用](docs/getting-started.md)操作。

Linux 或具备 POSIX shell 的环境可从仓库根目录构建和运行主机测试。

```sh
make -C Firmware -j4
make -C Firmware test
```

Windows PowerShell 的对应入口如下。

```powershell
./Firmware/tools/build.ps1
./Firmware/tools/test.ps1
```

生成文件为 `Firmware/build/FluxDial.elf`、`FluxDial.hex` 和 `FluxDial.bin`。
工具安装位置、脚本参数和烧录步骤以 [上手文档](docs/getting-started.md)为准。

## 仓库结构

```text
FluxDial/
├── docs/                  # 当前维护的设计、操作、固件和验证文档
├── Documentation/         # 器件资料、历史报告和旧文档入口
│   ├── Datasheets/        # 芯片、探头、流量传感器和显示屏资料
│   ├── Reports/           # 既有实测记录、历史导出和交互式 BOM
│   └── BOM.csv            # 既有器件清单，已知与当前工程存在差异
├── Firmware/              # 应用、BSP、启动代码、官方驱动和工具
├── Hardware/              # KiCad 工程、四张功能页、本地库和检查脚本
└── Images/                # 样板照片、视频及规格书预览
```

器件资料和工程文件保持原有目录，项目库通过相对路径引用。

## 文档导航

| 目标 | 入口 |
| --- | --- |
| 接线、编译、烧录和观察显示 | [上手](docs/getting-started.md) |
| 理解数据流与代码职责 | [系统架构](docs/architecture.md) |
| 查电源、器件、接口、引脚和显示矩阵 | [硬件参考](docs/hardware.md) |
| 查采样周期、换算、状态码与诊断变量 | [固件参考](docs/firmware.md) |
| 区分历史实测、本次检查和待验证项 | [验证记录](docs/validation.md) |
| 查原厂资料、历史 BOM 和封装图 | [资料索引](Documentation/README.md) |
| 查本轮文档修改及复审结果 | [文档审查记录](docs/documentation-audit-2026-10-29.md) |

## 已知问题

- 归档 BOM 与当前工程存在器件值差异，见 [晶振电容核对说明](docs/hardware.md#mcu-电源与时钟)。
- 当前 PCB 仍有装配间距、悬空走线端点和丝印相关检查项，见 [检查结果](docs/validation.md#本次检查)。
- 流量采用厂商标称系数，尚无实测精度数据；静止水流和未接流量传感器都会显示零。
- 温标选择和累计流量不写入 Flash，产品固件也不包含 USB 数据通信或 USB 下载功能。

## 文档维护

详细参数分别由硬件参考和固件参考维护，其他页面通过链接引用。
更新记录时注明所用提交、工具和验证范围，并保留旧记录的日期与边界。
[文档目录](docs/README.md)列出了各页职责。

## 许可证

仓库目前没有覆盖 FluxDial 自有代码、硬件设计和文档的项目级许可证。
CMSIS 与 ST 驱动保留各自的许可证，不能用这些许可证推定整个项目的授权范围。
具体文件和待确认事项见 [许可证与第三方资料](docs/licenses.md)。

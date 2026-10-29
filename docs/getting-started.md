# 编译、烧录与首次使用

以下命令从仓库根目录执行。
Windows 脚本已经在本轮核对；烧录步骤只检查了脚本和路径，没有重新对实板执行下载。

## 准备工具

| 工具 | 用途与已核对版本 |
| --- | --- |
| Git | 获取仓库 |
| KiCad 10 | 打开硬件工程；本次读取与检查使用 10.0.6 |
| Arm GNU Toolchain | ARM 构建；本次使用 14.3.Rel1，GCC 14.3.1 |
| GNU Make 与 POSIX shell | 执行 Makefile；本次 Make 为 4.4.1，Windows 脚本使用 `SHELL=sh` |
| 本机 C 编译器 | 主机计算测试；Windows 测试脚本可查找 TCC、GCC 或 Clang |
| SEGGER J-Link 软件及调试器 | SWD 下载与观察运行变量；仓库不附带安装包 |

工具下载入口为 [Arm GNU Toolchain](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)和 [SEGGER J-Link](https://www.segger.com/downloads/jlink/)。
本次 Windows 环境使用 [xPack Windows Build Tools 4.4.1-3](https://github.com/xpack-dev-tools/windows-build-tools-xpack/releases/tag/v4.4.1-3)提供 Make 与 shell 工具。
固件所需的 CMSIS 和 LL 头文件已经位于 [Firmware/Drivers](../Firmware/Drivers/README.md)，无需先生成 CubeMX 工程。

```sh
git clone https://github.com/SmallCoral/FluxDial.git
cd FluxDial
```

用 KiCad 打开 [Hardware/FluxDial.kicad_pro](../Hardware/FluxDial.kicad_pro)。
若提示缺少 `Noto Sans CJK SC`，安装该字族后重新打开 KiCad；`Noto Sans SC` 的字族名并不相同。

## 编译固件

Linux 或已配置工具链的 POSIX shell 使用以下命令。

```sh
arm-none-eabi-gcc --version
make -C Firmware -j4
```

成功后生成以下产物。

| 文件 | 用途 |
| --- | --- |
| `Firmware/build/FluxDial.elf` | 调试符号、链接结果和内存占用分析 |
| `Firmware/build/FluxDial.hex` | Windows 烧录脚本的输入 |
| `Firmware/build/FluxDial.bin` | 原始二进制镜像 |

Windows PowerShell 可使用现有构建脚本。

```powershell
./Firmware/tools/build.ps1
```

脚本默认在 `$env:LOCALAPPDATA/Programs/STM32SensorTools` 下递归寻找 `arm-none-eabi-gcc.exe` 和 `make.exe`。
这个目录是当前维护机器的安装约定，克隆仓库不会自动安装工具。
其他安装位置通过 `-ToolRoot` 指向同时包含编译器、Make 和相应 shell 工具的目录。

```powershell
# 示例路径：按本机安装位置替换。
./Firmware/tools/build.ps1 -ToolRoot 'C:/Toolchains/STM32SensorTools'
```

构建失败时脚本报错。
当前工具链链接 `nosys` 时可能提示未实现的文件 I/O 存根；本次构建结果见 [验证记录](validation.md#本次检查)。

## 运行主机测试

POSIX shell 使用本机 `cc`，也可显式选择 GCC。

```sh
make -C Firmware test
# 若本机没有 cc，可改用：
make -C Firmware test HOST_CC=gcc
```

Windows PowerShell 使用测试脚本。

```powershell
./Firmware/tools/test.ps1
```

该脚本先从 PATH 查找 TCC、GCC 或 Clang，再尝试当前维护机器的 `STM32SensorTools/native-tests`。
可用 `-Compiler` 指定本机编译器的实际路径。
测试成功会输出以 `PASS: NTC reference points/monotonicity` 开头的结果行。
覆盖范围及外设桩限制见 [验证记录](validation.md)。

## 接线与烧录

1. 断开板上电源，按 [硬件接口表](hardware.md#接口与-mcu-引脚)核对 J301、J302 和 J102。
2. 用 J101 的 5V 电源给整板供电，将调试器连接到 J102 的 VREF、SWDIO、SWCLK 和 GND。
3. 确认 BOOT 保持正常状态，先完成固件编译，再运行下面的烧录脚本。

```powershell
./Firmware/tools/flash.ps1
```

脚本默认使用 `C:/Program Files/SEGGER/JLink/JLink.exe`，设备名为 `STM32C562CE`，接口 SWD，初始速率 1000kHz。
它载入 `Firmware/build/FluxDial.hex`，随后复位并运行目标程序；J-Link 返回失败时脚本报错。
`-JLinkPath` 可指定软件位置，`-Serial` 可选择指定探针。
J102 没有 NRST，硬件复位使用板上的 RESET 按键。

本轮没有验证探针连接、目标板供电、Flash 校验或新版固件实板运行。
这些步骤需要在实际硬件上确认。

## 观察显示

1. 上电后等待温度状态确认，观察上方是否从 `--` 进入读数。
2. 短按 USER，观察摄氏和华氏显示切换；边界情况见 [显示状态](firmware.md#显示与异常状态)。
3. 在没有水流时，下方流量应为零；这不能证明流量传感器已连接或工作正常。
4. 使用实际水路验证脉冲、流量和停流行为，再用参考体积或流量计进行标定。

热水提示、告警阈值、故障显示和断电后的状态见 [固件参考](firmware.md)。
本项目当前没有经过整机精度与长期水路测试，不应仅凭演示录像判断测量性能。

## 排查问题

| 现象 | 优先核对 |
| --- | --- |
| 无法编译 | 工具链是否安装、`-ToolRoot` 是否覆盖 Make 与编译器、POSIX shell 是否可用 |
| J-Link 无法连接 | J101 供电、VREF、SWDIO/SWCLK 顺序、公共地、设备名及探针选择 |
| 屏幕不亮 | 实际电源轨、I²C 接线和 `display_errors`；本轮没有带载电压数据 |
| 温度显示 Er | 探头连接、输入节点和 `ntc_status` / `adc_errors` |
| 切换华氏后显示 HI | 先核对当前温度是否超出显示数字能力 |
| 流量始终为零 | 实际水流、线序、传感器供电以及 `pulses` / `rejected_pulses` |

诊断变量及单位见 [调试参考](firmware.md#调试变量)。

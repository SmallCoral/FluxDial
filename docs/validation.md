# 验证记录与已知边界

本页区分可重现的软件检查、当前硬件静态检查及既有实板记录。
本次日期为 **2026-10-29**，源码与硬件基线为 `953b748`。
本轮只修改 Markdown，没有重新烧录、接水、测温或测量电源。

## 本次检查

| 检查 | 方法与结果 | 能证明什么 |
| --- | --- | --- |
| 远端与工作基线 | 审查开始时，本地与 GitHub `master` 同为 `953b748` | 审查对象一致 |
| 强制完整 ARM 构建 | Arm GNU Toolchain 14.3.Rel1 / GCC 14.3.1，GNU Make 4.4.1；构建成功 | 当前源码能够编译与链接 |
| 构建产物 | 生成 FluxDial ELF、HEX 和 BIN；ROM 使用 7720B，RAM 区使用 1784B | 指定工具链下的链接结果，RAM 数值含链接脚本预留堆栈 |
| 主机测试 | 现有四组测试通过 | 换算、状态与段码的测试断言成立 |
| PowerShell 脚本 | 构建与测试入口实际执行；烧录脚本仅完成语法、参数与文件路径核对 | 不包含本轮实板烧录结果 |
| KiCad 读取 | 10.0.6 导出当前五页原理图 PDF 和网表，读取 PCB | 当前文件能被该版本解析 |
| 原理图与 PCB 对照 | 68 个原理图器件均在 PCB 中找到，器件值及有编号焊盘的网络一致 | 本次对照未发现这部分差异，不代表 DRC 通过 |
| ERC | 0 条错误、4 条警告 | 仍有库符号与封装筛选警告 |
| DRC | 11 条错误、8 条警告，另列的未连接项目为 0 | 仍需人工处理或确认相关检查项 |

构建包含既有 `nosys` 文件 I/O 存根警告，未将这些警告记为新的功能或运行测试结果。
编译、测试和硬件检查的复现命令见下节。

### 主机测试范围

[test_product.c](../Firmware/tests/test_product.c) 包含四组测试。

- `test_ntc`：厂商表基准点、有效 ADC 区间单调性、阻值越界、开短路阈值与温标舍入。
- `test_faults_and_hysteresis`：故障确认、恢复、高温告警与热水提示的部分滞回点。
- `test_flow`：标称周期换算、累计量、停流模型与 32 位计数回绕。
- `test_frames`：正常温度、负数、温标越界、故障、流量越界及标签段码。

测试用桩替代 HT16K33 传输，不连接真实 ADC、I²C、GPIO 中断、TIM2 或看门狗。
“测试通过”只表示现有断言通过，不能延伸为完整硬件链路或所有边界的覆盖声明。

### 当前 ERC 与 DRC 项

| 检查 | 数量 | 内容 |
| --- | --- | --- |
| ERC 警告 | 3 | 两个 L 符号与一个 Crystal_GND24 符号的库差异 |
| ERC 警告 | 1 | 晶振已分配封装与 `Crystal*` 筛选规则不匹配 |
| DRC 错误 | 11 | `courtyards_overlap`，封装装配区重叠 |
| DRC 警告 | 2 | `track_dangling`，走线端点悬空 |
| DRC 警告 | 6 | `silk_over_copper`，丝印被阻焊区域裁切 |

部分装配区重叠可能需要结合圆屏覆盖关系判断，但本轮没有将任何错误自动认定为可忽略。
未连接项目为零也不能消除其他 DRC 错误。
本次报告临时生成用于审查，没有覆盖 `Documentation/Reports/hardware-review` 内的历史 JSON。

## 复现软件与硬件检查

从仓库根目录，在已安装工具链的 POSIX shell 执行以下命令。

```sh
make -C Firmware -B -j4
make -C Firmware test
```

Windows 的正常构建和测试入口见 [上手文档](getting-started.md)。
KiCad 10 命令行工具加入 PATH 后，可将检查文件导出到临时目录。

```powershell
$checkDir = Join-Path $env:TEMP 'FluxDial-check'
New-Item -ItemType Directory -Force -Path $checkDir | Out-Null
kicad-cli sch export netlist --format kicadxml --output (Join-Path $checkDir 'current.net.xml') Hardware/FluxDial.kicad_sch
kicad-cli sch erc --format json --output (Join-Path $checkDir 'erc.json') Hardware/FluxDial.kicad_sch
kicad-cli pcb drc --format json --output (Join-Path $checkDir 'drc.json') Hardware/FluxDial.kicad_pcb
```

这些命令默认生成报告，不能仅用进程退出码判定没有违规项；应阅读 JSON 中的严重程度和条目。
它们不会更新原理图、PCB 或预期设计清单。

## 历史实板记录

[2026-10-06 产品固件验证记录](../Documentation/Reports/firmware/VALIDATION.md)保留了当时的工具、烧录和观察过程。
其中记录了真实 NTC 读数、USER 温标切换、调试器产生的板上 GPIO 脉冲测试，以及短时间运行快照。
板上脉冲测试验证了中断、计时、换算与停流逻辑，没有让实际水流驱动 BTL-004A。

照片和视频来自更早的显示演示，媒体说明见 [Images](../Images/README.md)。
本次静态检查不能追认历史照片、烧录镜像与当前 KiCad 文件属于完全相同的物料和修订状态。

## 仍需人工确认

| 项目 | 当前证据不足之处 |
| --- | --- |
| 实际装配 | C407/C408 的实际焊接值，以及 [硬件参考](hardware.md#mcu-电源与时钟)所列的工程、注释和归档 BOM 差异 |
| 制板与装配检查 | 当前 DRC 错误、丝印、走线端点及屏幕与下方器件的装配关系 |
| 供电 | 两路电源在实际输入与负载下的电压、压降、纹波和温升 |
| 实际水流 | 完整传感器链路、可用流量范围、停流响应和标称系数标定 |
| 温度测量 | 参考温度对照、动态响应、阈值实测和物理拔插探头 |
| 长期运行 | 实际水路、供电扰动和异常通信条件下的持续运行 |
| 材料授权 | 自有工程许可证，以及第三方 PDF、图片和模型的分发授权 |

BTL-004A 手册的量程表述差异见 [硬件参考](hardware.md#传感器调理)。
目前没有整机精度、功耗、EMC 或长期可靠性测试报告。

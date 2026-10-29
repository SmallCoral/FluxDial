# FluxDial 硬件工程

用 KiCad 10 打开 [FluxDial.kicad_pro](FluxDial.kicad_pro)。
工程包含 [原理图主页](FluxDial.kicad_sch)、四张功能页和 [PCB](FluxDial.kicad_pcb)。

| 目标 | 入口 |
| --- | --- |
| 查看电源、关键器件、接口与引脚 | [硬件参考](../docs/hardware.md) |
| 理解原理图分工与信号路径 | [系统架构](../docs/architecture.md) |
| 接线、编译和烧录 | [上手](../docs/getting-started.md) |
| 查看当前 ERC/DRC 与已知问题 | [验证记录](../docs/validation.md) |
| 查符号、封装与 3D 模型 | [libraries](libraries/) |
| 查原厂手册、历史 BOM 和检查导出 | [资料索引](../Documentation/README.md) |
| 查看样板照片与视频 | [Images](../Images/README.md) |

本地库名为 `FluxDial`，由 [sym-lib-table](sym-lib-table)与 [fp-lib-table](fp-lib-table)加载。
`tools/` 中的旧检查脚本依赖历史设计清单，使用边界见 [报告说明](../Documentation/Reports/README.md)。
本轮文档工作没有修改工程文件或重新生成交互式 BOM。

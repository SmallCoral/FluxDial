# 验证记录与历史导出

| 目录 | 内容 |
| --- | --- |
| [firmware/](firmware/VALIDATION.md) | 产品固件的计算测试和实板验证记录 |
| [hardware-review/](hardware-review/) | 原有网表、检查结果、封装图和设计备份 |
| [interactive-bom/](interactive-bom/ibom.html) | 交互式 PCB 器件清单 |

`hardware-review/` 和交互式 BOM 保留原有导出内容，工程名及资源路径随 FluxDial 整理。目录内不同日期的网表、设计清单和检查结果来自不同设计阶段，不能合并视作当前硬件已经通过的检查。

`Hardware/tools/` 中的旧检查脚本继续使用这些历史数据。整理前运行时，网表与清单、PCB 与清单之间已存在差异；本次目录调整未修改电路连接或 PCB 几何。检查当前硬件时，应先从当前 KiCad 工程重新导出，并更新对应的预期清单。

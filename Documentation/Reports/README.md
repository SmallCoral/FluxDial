# 验证记录与历史导出

当前版本的软件与硬件检查集中在 [验证记录](../../docs/validation.md)，本轮文档审查见 [审查报告](../../docs/documentation-audit-2026-10-29.md)。

| 目录 | 内容与范围 |
| --- | --- |
| [firmware](firmware/VALIDATION.md) | 2026-10-06 的产品固件与实板验证记录 |
| [hardware-review](hardware-review/) | 原有网表、设计清单、检查 JSON、封装图及备份 |
| [interactive-bom](interactive-bom/ibom.html) | 历史交互式 PCB 器件清单 |

历史导出来自不同设计阶段，不能拼合为当前硬件全部通过的证明。
归档 BOM、网表和图页注释中的器件值可能落后于当前 KiCad 文件。

[旧网表检查脚本](../../Hardware/tools/check_netlist.py)与 [旧 PCB 检查脚本](../../Hardware/tools/check_pcb.py)仍读取本目录中的预期清单和导出网表。
它们并非自动针对当前工程重建的回归检查，出现差异时应先核对各输入的版本。
本轮没有重写预期清单、覆盖历史报告或修改检查脚本。

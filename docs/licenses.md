# 许可证与第三方资料

本页记录仓库中可以直接核对的授权文件，审查日期为 2026-10-29。

## 项目自身

仓库根目录没有 `LICENSE`、`COPYING` 或其他覆盖整个 FluxDial 项目的授权文件。
自有应用、硬件设计、文档和媒体的统一授权条款尚未明确。
本轮没有替作者选择或添加许可证，也没有给整个项目标注 MIT、Apache 或其他 SPDX 标识。

## 已保存的第三方许可证

| 范围 | 仓库内许可证标识 | 原文 |
| --- | --- | --- |
| CMSIS Core | Apache-2.0 | [CMSIS/LICENSE.md](../Firmware/Drivers/CMSIS/LICENSE.md) |
| STM32C5 器件头文件 | BSD-3-Clause | [STM32C5xx/LICENSE.md](../Firmware/Drivers/CMSIS/Device/ST/STM32C5xx/LICENSE.md) |
| STM32C5 LL 驱动 | BSD-3-Clause | [STM32C5xx_LL_Driver/LICENSE.md](../Firmware/Drivers/STM32C5xx_LL_Driver/LICENSE.md) |

这些文件保留原有声明和上游归属，本轮未改写其正文。
来源版本和导入记录见 [驱动来源说明](../Firmware/Drivers/README.md)。
部分启动、系统或链接文件带有 ST 版权与指向组件许可证的文件头，不能仅由所在目录推定为项目自有授权。

## 手册、图片和模型

[Documentation/Datasheets](../Documentation/Datasheets/) 包含 ST 及其他器件厂商资料。
资料包内的 [SHA256SUMS.txt](../Documentation/Datasheets/STM32C562CET6_2026-09-16/SHA256SUMS.txt)用于核对所保存的官方 PDF 内容，校验值不授予分发许可。

规格书预览图片保留对应 PDF 来源；项目照片、录像、封装和 STEP 模型没有统一的单独授权说明。
这些材料的版权与使用范围需要由作者和相关权利方确认。

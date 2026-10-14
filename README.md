# 图灵智显 3.5 寸副屏 — 功耗补丁版

本仓库发布**功耗补丁版**图灵智显（Turing Smart Screen）3.5 寸副屏软件，以及配套的功耗主题包。

在原版基础上，本版本补上了官方程序缺失的 **GPU 功耗** 数据通路，并把 CPU 功耗一并纳入主题显示。

## 仓库结构

```
software/                              功耗补丁版软件（完整可用目录，双击即用）
  UsbMonitor.exe                       补丁版主程序（GPU 功耗渲染补丁）
  config/                              主题与配置（37+ 个 .data）
    ├ 功耗监测.data                      新增 480x320 功耗主题（CPU/GPU 温度/使用率/功耗）
    ├ PulseCoreHorizontal_Power.data     横屏功耗主题
    ├ PulseCoreVertical_Power.data       竖屏功耗主题
    └ AppConfig.data                    当前默认主题与传感器定向
  3.5/                                 44 张背景 PNG（官方原版）
  驱动程序/                             USB 串口驱动（usbser/cdc.inf）
  *.dll / code.ini                     官方原版运行库与配置
themes/                                独立主题包（可单独安装到其他补丁版软件）
  PulseCoreHorizontal_Power/           横屏功耗主题（.data + 预览图 + 说明）
  PulseCoreVertical_Power/             竖屏功耗主题（.data + 预览图 + 说明）
  *.zip                                上述两个主题的独立压缩包
docs/                                  技术文档
  DELIVERY_REPORT.md                   交付报告（补丁内容、证据、验收结论）
  主题渲染机制.md                       主题数据格式与渲染流程说明
  PulseCoreHorizontal_Power.md         横屏主题改动说明
  PulseCoreVertical_Power.md           竖屏主题改动说明
```

## 快速使用

1. 下载 `software/` 整个目录（保持目录结构不变）。
2. 双击 `software/UsbMonitor.exe` 启动，无需 bat 或命令行窗口。
3. 首次使用需安装 `software/驱动程序/usbser/cdc.inf`（详见 `驱动安装教程.txt`）。
4. 在软件中选择主题：`功耗监测`、`PulseCoreHorizontal_Power` 或 `PulseCoreVertical_Power`。

> 主题包依赖补丁版程序。**仅把主题装进未打补丁的官方原版，无法建立 GPU 功耗绑定**（原版没有 GpuPower 数据通路）。

## 补丁做了什么

补丁只改 `UsbMonitor.exe` 中 `AdaptSensorData` 一个方法体（dnlib 注入），共两处：

- 新增 `itemName == "GpuPower"` 的匹配分支；
- 在取值点按 itemName 选择 getter（GPU 功耗字段 / CPU 功耗字段）。

除此之外，程序集元数据、其余方法、主题编辑器、状态条、弧形条、天气、托盘等原有功能全部保持原样。原版完整树的其余文件与原包逐文件哈希一致。

详细证据（方法级 IL 哈希比对、归一化 LCS 差分、运行时哨兵注入验证、离线渲染回填）见 `docs/DELIVERY_REPORT.md`。

## 已知限制

- 未在实体副屏上目视验收显示效果（本机无实体屏，渲染与传感器读取均走真实代码路径）。
- 托盘图标点击 / 退出菜单未实机点击验证。
- 串口热插拔重连未验收。
- `AppConfig.data` 内的传感器名、城市等为发布者本机定向值（如 `GPU [#0]: NVIDIA GeForce RTX 4070 SUPER`、`CPU (Tctl/Tdie)`、城市`福州`），换机后请在软件「设置」中重新选择。

## 说明

本仓库仅用于发布与备份该补丁版本，供个人副屏使用。

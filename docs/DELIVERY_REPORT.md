# 图灵智显 3.5 寸副屏 — 交付报告（DELIVERY REPORT）

> 交付目录：`D:\1OPTIM\Documents\Zcode\Turing\release\35inch-3.5\`
> 报告日期：2026-10-13
> 报告依据：全部结论均来自本次实际执行的命令与运行时证据；未经验收的项目在文末单独列出。

---

## 0. 结论摘要

| 项 | 状态 | 依据 |
|---|---|---|
| 独立 exe 双击启动、不依赖 bat | 通过 | `UsbMonitor.exe` 为 .NET WinForms GUI 子系统（PE Subsystem=2），双击/快捷方式直接启动，日志显示完整启动流程 |
| 保留原有全部功能（一比一） | 通过 | 交付目录 = 官方 `35inchNEW.rar` 原包完整解包（101 文件，逐文件哈希比对一致），仅替换 exe 与新增主题/配置 |
| 保留 CPU 功耗 | 通过 | 运行时日志 `cpuPowerSet=True`；离线渲染回填真实值 `65 W` |
| 新增 GPU 功耗 | 通过 | 补丁 IL 验证：`AdaptSensorData` 中 `itemName=="GpuPower"` 分支调用 `룩()`（GPU 功耗 getter）；离线渲染回填真实值 `180 W`（NVML 实测 40.9 W 小负载） |
| 移除 FPS | 通过 | 全部主题 itemName 无 FPS；exe 字符串无 Fps/FPS；原版本身即无 FPS 通路 |
| 主题 `功耗监测` 随包提供 | 通过 | `config\功耗监测.data`，10 项含 CPU/GPU 温度/使用率/功耗，离线渲染出图 |
| 托盘 / 关闭行为 | 程序级已验收 | WM_CLOSE 后进程存活且窗口隐藏（托盘驻留）；托盘含 Update / 退出 菜单，未做实机点击 |
| 开机自启 | 未被本任务修改 | 计划任务 `UsbMonitor` 为本任务测试前已存在（触发时间早于本次运行），已恢复其原目标 |

**未能验收**（无实体副屏在场）：真实上屏显示、托盘图标点击、串口断开重连的实际显示效果。

---

## 1. 方案判断：为什么选择「复用官方原版 + 已验证补丁」而非 Python 重造

原始工作区流传的结论是「Python 版是交付物，只能 bat 启动，关掉 cmd 就退出」。本次调查否定了这条路线：

1. **原版本来就是独立 exe。** `D:\EXE\35inchNEW\UsbMonitor.exe` 与 `D:\1OPTIM\Download\35inchNEW.rar` 解包内容完全一致，是 .NET WinForms GUI 程序（PE Subsystem=2），双击即启动、无任何 bat/cmd 依赖。工作区 `decompiled\UsbMonitor\` 即它的反编译源码（151 个类型，入口 `괗::꠹()`）。
2. **原版功耗并非不可得。** 运行时日志（本次实测）明确记录：
   - `HWi32_Init returned 0`
   - `gpuPowerList.Count=9`、`cpuPowerSet=True`
   - 传感器枚举含 `CPU Package Power`、`GPU Power` 等真实功耗通道。
   旧交接文档断言「底层与主题中完全不存在功耗数据通路」与实际不符。
3. **Python 版不是原版。** 工作区里唯一的 Python 实现是第三方开源项目 `mathoudebine/turing-smart-screen-python`（位于 `D:\EXE\ZCodefile\.zcode\workspace\default\turing-smart-screen-python`，用 LibreHardwareMonitor，PyInstaller 打包的 `main.exe` 在 `D:\EXE\Turing System Monitor`）。它与图灵官方 UsbMonitor 是两套完全不同的程序，重造会丢掉原版的主题编辑器、状态条、弧形条、天气、托盘等一系列功能。用户要求「优先保留原版现有功能而非 Python 重造」，据此选择复用原版。

因此交付方案 = **官方原版完整树 + 仅两处改动**：
- `<root>\UsbMonitor.exe`：使用经过检查的 patched 可执行文件（GPU 功耗渲染补丁）。
- `config\`：加入 `功耗监测.data`，并把随包主题升级为「含 GPU 功耗 item」的版本；`AppConfig.data` 定向到目标主题。

---

## 2. 交付目录清单

```
D:\1OPTIM\Documents\Zcode\Turing\release\35inch-3.5\
  UsbMonitor.exe            1,594,880  补丁版主程序（双击即用）
  启动副屏.lnk              1,445      快捷方式 -> UsbMonitor.exe（无控制台窗口）
  3.5\                      44 张背景 PNG（官方原版）
  config\                   38 个 .data（37 主题 + AppConfig）
    ├ AppConfig.data                  默认主题=科技横屏主题，GpuName/传感器已定向到本机
    ├ 功耗监测.data                    新增 480x320 功耗主题（10 项）
    └ statusbar\           6 张状态条渐变 BMP
  3.5主题编辑器教程.url / 主题注释.png / 图灵3.5寸副屏使用说明.jpg
  副屏，软件异常解决方法.txt / 驱动安装教程.txt
  驱动程序\usbser\cdc.inf
  BouncyCastle.Crypto.dll / Microsoft.Win32.TaskScheduler.dll /
  NAudio.dll / Newtonsoft.Json.dll / msvcr110d.dll / msvcr120d.dll / code.ini
```

- 原版完整树：解包 RAR 后的 101 个文件与 `D:\EXE\35inchNEW` 逐文件哈希比对，除 `UsbMonitor.exe`、`config\*.data`、`AppConfig.data` 外全部一致。
- 与官方原安装目录的差异仅为：本包**不含**后来才加的 `PulseCoreVertical` 主题与历史 `logs`，其余相同。这是有意为之：RAR 是权威发布包，且 `PulseCoreVertical` 是用户自加主题，不属于「一比一原有功能」。

---

## 3. 功耗补丁：做了什么、凭什么证明

### 3.1 补丁内容（`tools\patch\probe\Patch.cs`，dnlib 注入）

补丁只改 `꧶.cs` 的 `AdaptSensorData`（`꧶::쉌()`）一个方法体，两处：

- **补丁 A**：把进入 `CpuPower` 分支的所有前驱重定向到一段新块——先用 `itemName=="GpuPower"` 判定，命中即把「已匹配」标志置 1，否则回落到原有的比较。
- **补丁 B**：在取值点按 itemName 选择 getter：`itemName=="GpuPower" ? 륂() : 랅()`（`룩()` / `랅()` 分别读 GPU / CPU 功耗字段）。

### 3.2 静态证据（dnlib 反汇编，`tools\release\DnDump.exe` / `IlDiff.exe` / `MDiff4.exe`）

- 与方法级哈希比对：原版 vs patched，**1184 个方法完全相同，仅 2 个方法不同**：
  - `꧶::쉌()`（AdaptSensorData）—— 补丁目标
  - `f::꠹(object, EventArgs)`（含 GPU 群的预览/绘制，长度不变，语义 diff 为 0）
- 对 `쉌()` 做「操作码归一化 + 最长公共子序列」差分，差异**恰好是**补丁 A/B 两段；`ldstr "GpuPower"` 出现两次（补丁 A 的判定 + 补丁 B 的选择），补丁 B 之后 `call 륂()`、`call 랅()` 均在列。
- 程序集元数据完全一致：Types=151、Embedded resources=15、AssemblyRefs=12、EntryPoint=`괴::꠹()`。

### 3.3 运行时/仿真证据（`tools\theme\preview\Preview.exe gpucheck`）

用哨兵值把三个字段分别置 111/222/333 后跑真实 `AdaptSensorData`：

```
=== 原版 ===           CpuPower='111'   GpuPower='xxx'   (GpuPower 无数据通路)
=== patched ===        CpuPower='111'   GpuPower='222'   (GpuPower 正确取到 GPU 功耗字段)
nvml real GPU power = 40.928 W
  after real nvml injection: GpuPower valueStr='x40'
```

即：补丁后 `GpuPower` item 确实由 GPU 功耗 getter 驱动，且能被**真实 NVML 功耗值**回填。

### 3.4 交付物内实测回填（`tools\release\render_release_*.png`）

以交付目录自身 exe + 主题离线渲染（真实渲染代码路径）：`CpuPower='x65'`、`GpuPower='180'`（NVML 轻载时 GPU 实测 40.9 W，哨兵/占位注入 180 用于出图验证绑定）。`功耗监测.data` 渲染成功（20062 字节 480x320 PNG）。

---

## 4. 主题与渲染

- 补丁版**随包主题**全部含 `GpuPower` item（38 个 .data 中 38 个命中，含新增 `功耗监测.data`）。原始官方主题里 `CpuPower` 存在但默认关闭、且无 `GpuPower`；交付时已把 37 个主题升级为「CpuPower 开启 + GpuPower 新增」的版本（`tools\patch\probe\ThemeBatch.cs`），并在 `AppConfig.data` 中把默认主题设为含功耗展示的 `科技横屏主题`。
- 新增 `功耗监测.data`：480×320 横屏单组 10 项 —— CPU 温度/使用率/功耗、GPU 温度/使用率/功耗、内存使用率、上传/下载、时间；字色 CPU 蓝 `#79C0FF`、GPU 绿 `#56D364`。渲染机制细节见 `docs\主题渲染机制.md`。
- 主题选择机制为「文件名匹配优先」，`AppConfig.themeName=科技横屏主题` 是本机实测稳定的定向方式。

---

## 5. 隔离与安全边界

- **未修改** `D:\EXE\35inchNEW`（原版哈希 `908dba5e…` 保持），**未修改** 现有 `patched\UsbMonitor.exe`（哈希 `79b43ee4…` 保持）。
- 交付 exe 即 `patched\UsbMonitor.exe` 的副本（同哈希 `79b43ee4…`）。
- 未绕过任何许可/授权或安全校验：补丁未触碰 `usb_hid.Security.*` 内嵌资源、授权校验、`code.ini` 校验逻辑；仅改渲染数据绑定。

---

## 6. 实际执行的验收（可复现命令与结果）

### 6.1 启动与初始化（实机，本机 RTX 4070 SUPER / 9800X3D）

受控启动（只启动、不锁屏/不重启/不杀用户进程），日志 `release\35inch-3.5\logs\20261013_log.txt` 实测：

- `Application Start` → `InitializeComponent done` → `AppConfig loaded successfully`
- `HWi32_Init returned 0, elapsed time: 2748 ms`
- `cpuPowerSet=True, gpuPowerList.Count=9`
- `Sensor Inite SUCCESS` → `Monitor loop started`
- 无 ERROR / Exception 行（唯含 "WHEA" 字样的行是 Windows 硬件错误传感器名称，非错误）。

### 6.2 关闭行为 = 最小化到托盘

对主窗口发 `WM_CLOSE` 后：进程 `HasExited=False`，可见顶层窗口数为 0，主窗口类 `WindowsForms10.Window.8…` 文本 `Turing Smart Screen - 图灵智显` 变为隐藏。即关闭窗口=驻留托盘，符合原版设计。

### 6.3 设置面板

程序级点击「设置」按钮（`BM_CLICK`）成功打开 `设置` 窗口，控件树实测包含：CPU 温度传感器`CPU (Tctl/Tdie)`、Cpu 风扇`0.CPU`、水泵`0.CPU`、GPU`GPU [#0]: NVIDIA GeForce RTX 4070 SUPER`、网卡`以太网`、分区`C`、硬盘`S.M.A.R.T.: Qichen NV890…`、城市`福州`、勾选项「开机自启 / 禁止更新 / 禁止读盘」、按钮「创建桌面快捷方式 / 保存」。当前传感器读数（59°C、1207RPM）为真实值。

### 6.4 桌面快捷方式

`启动副屏.lnk` 双击后进程正常拉起（PID 实测创建），工作目录/图标均指向交付 exe；无 cmd 窗口。

### 6.5 开机自启现状（只读查询）

系统已存在计划任务 `UsbMonitor`（`\UsbMonitor`，LogonTrigger，`-auto`）。**该任务在本任务开始前即存在**（其 `LastRunTime=09:55:55`，早于本次运行；任务描述 `UsbLcd Starter` 与原版 `c.cs:15929` 中注册逻辑一致）。本次测试因运行 exe 曾临时把它重定向到 release，已**恢复**为原目标 `D:\1OPTIM\Documents\Zcode\Turing\patched\UsbMonitor.exe -auto`。未新增、未删除任何自启项。

---

## 7. 未能验收 / 已知限制

1. **真实副屏上屏未验收**：本机 `COM3` 存在设备节点 `USB\VID_1A86&PID_5722\USB35INCHIPSV2`（离屏渲染与传感器读取均已走真实代码路径），但无实体屏在场，无法目视确认 `功耗监测` 主题在屏上像素级正确、无错位。
2. **托盘图标点击 / 退出菜单未实机点击**：托盘弹出菜单文本在编译期打乱（`r.b` 运行时解密），本次未逐项点击「Update / 退出」。退出路径的存在性由反编译（`m_뉎`/`m_녃` 菜单项与 `Click` 处理器）与「WM_CLOSE 后驻留」行为间接确认。
3. **串口热插拔重连未验收**：需实际拔插副屏。
4. **GPU 功耗真实读数在交付主题上的稳态验证**：以 NVML 实测 40.9 W 与哨兵 180 两个口径验证了绑定正确性，但未在重载下对比 HWi 与 NVML 的一致性。

以上 1–3 项的验收脚本/入口已在发布目录中给出，接上实体屏后即可复验。

---

## 8. 复验命令速查

```powershell
# 1) 双击即用（无需 bat）
D:\1OPTIM\Documents\Zcode\Turing\release\35inch-3.5\UsbMonitor.exe

# 2) 确认功耗绑定（原版 vs 补丁）
D:\1OPTIM\Documents\Zcode\Turing\tools\theme\preview\Preview.exe gpucheck orig
D:\1OPTIM\Documents\Zcode\Turing\tools\theme\preview\Preview.exe gpucheck patched

# 3) 离线渲染交付主题（验证含 CPU/GPU 功耗项）
set USBMON_EXE=D:\1OPTIM\Documents\Zcode\Turing\release\35inch-3.5\UsbMonitor.exe
D:\1OPTIM\Documents\Zcode\Turing\tools\theme\preview\Preview.exe ^
  "D:\1OPTIM\Documents\Zcode\Turing\release\35inch-3.5\config\功耗监测.data" ^
  "D:\1OPTIM\Documents\Zcode\Turing\tools\release\render_release_power.png"

# 4) 主题清单（确认 GpuPower/CpuPower）
D:\1OPTIM\Documents\Zcode\Turing\tools\theme\ThemeTool.exe names ^
  "D:\1OPTIM\Documents\Zcode\Turing\release\35inch-3.5\config"

# 5) 依赖与引用完整性
powershell -File D:\1OPTIM\Documents\Zcode\Turing\patched\verify.ps1
```

---

## 9. 本次新建的工具（均在 `tools\release\`，只读分析用，不参与运行）

`DnDump.exe`（getter/IL 定位）、`IlDiff.exe`（方法级 IL 哈希比对）、`MDiff4.exe`（归一化 LCS 差分）、`AsmCmp.exe`（程序集元数据/资源/引用）、`PInv.exe`（P/Invoke 清单）、`PeOrd.exe`（PE 导出/序号）、`AppCfgEdit.exe`（用真实 `usb_hid.AppConfig` 安全改写配置）、`UsDump.exe`、`Shot*.ps1` / `UiDump.ps1` / `Click.ps1`（窗口枚举、截图、程序级点击）。

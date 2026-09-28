# zmk-config-corne

[English](./README.en.md) · **简体中文**

Corne 分体键盘的 ZMK 固件配置，已接入 [DYA Studio](https://studio.dya.cormoran.works/)。

## 关于 Corne

[Corne](https://github.com/foostan/crkbd)（又名 **crkbd**，取自 cornetto 意式甜点的名字）
是日本开发者 foostan（Kosuke Adachi）设计的开源分体人体工学键盘，
最初发布于 2018 年 4 月，灵感来自 Helix 键盘。

它的核心特征：

- **3×6 列错位 + 3 枚拇指键**（单侧），共 42 键。列错位（column stagger）能让手指
  沿各自自然的列方向上下移动，比行错位更省手腕动作 —— 这一布局后来启发了大量同类设计
- **对称双 PCB + TRRS 互联** —— 左右完全相同，单价低、易焊接
- **开放式** —— 从第一天就开源，任何人都能自己打样、改键位、做外壳衍生
- **可拆最外侧列** —— 多数版本的最外列可拔掉，变成 3×5+3 的 34 键布局

正因为设计开放、门槛低，Corne 成为近十年最流行的分体键盘设计之一，
衍生出 Classic / Cherry / Chocolate / Light 等官方分支，以及大量社区版本。

本仓库针对的是 **V4 外形 + Pro Micro 尺寸插座**这一支，见下一节。

## 适配的硬件

**[Corne V4 Pro-Micro Edition](https://github.com/klouderone/CorneV4ProMicroEdition)**
（Kea Workshop 出品，klouderone 维护的分支）—— foostan 原版 Corne V4 外形，
保留 Pro Micro 尺寸插座，可插 [nice!nano v2](https://nicekeyboards.com/nice-nano) 做无线，
也可插 Elite-C v4 做有线。

| 项目 | 说明 |
| --- | --- |
| 微控制器 | nice!nano v2（本配置的目标）/ 其他 Pro Micro 尺寸板 |
| 轴座 | Kailh **MX** 与 **Choc** 热插拔底都支持，带拔插可做 5 列 |
| 二极管 | SMD 1N4148 与插件二极管都支持 |
| 显示 | 0.91" OLED（I²C）或 nice!view 扩展板 |
| 连接 | 3.5mm TRRS 分体互联 + 电池焊盘 / PH2 座 / 电源开关 |
| 底灯 | ❌ 无 RGB 底灯，所以配置里的 underglow 是关闭的 |
| 键距 | 19.05mm（V4 相比 V3 由 19mm 微调） |

> **别买错版本。** foostan 官方的 Corne V4 是**板载 RP2040**、USB-C 接口，
> 既插不了 nice!nano 也用不了本仓库的无线固件。只有 Kea Workshop 的
> Pro-Micro 版才兼容 Pro Micro 尺寸的板子。

> 本配置按 **MX 轴体** 编译矩阵接线。PCB 的 Choc 轴座虽然硬件兼容，
> 但 Choc 的行列引脚分配与 MX 不同，ZMK 自带的 `corne` shield 只定义了 MX 版本。
> 若实际焊接的是 Choc 轴体，需要另行配置 Choc 矩阵，本仓库暂未包含。

> ⚠️ 该板是无线 + 有线两用设计：**插电池时不要用 TRRS**，**用有线时不要接电池**。
> 本仓库的固件为无线版本（nice!nano）。

## DYA Studio

浏览器打开 <https://studio.dya.cormoran.works/>，按 splash 页面选择连接方式：

| 方式 | 说明 | 支持的浏览器 |
| --- | --- | --- |
| USB | 插线后选择串口（Web Serial） | Chrome / Edge（桌面） |
| Bluetooth | 配对后通过 BLE 连接（Web Bluetooth） | Chrome / Edge（桌面）、Android Chrome、iOS 用 Bluefy |
| Demo | 无需键盘，模拟键盘试用全部界面 | 同上 |

Firefox 与 Safari 不支持。

### 可用功能

- **Keymap Editor** — 可视化编辑键位与图层，按键实时高亮
- **Macros & Combos** — 网页上直接创建宏与组合键，**无需重新编译固件**
- **Connection Management** — BLE 配置档重命名 / 切换 / 解绑，设置 USB 与蓝牙的优先级
- **Device Settings** — 电源管理（idle / deep sleep 超时）、各分半独立设置
- **Troubleshooting** — 电量、固件版本、uptime；逐键抖键检测；复制完整支持报告

> **按 OS 自动切层**需要 central 半边开启 OS Detection 与 Default Layer 模块。
> Corne 无轨迹球，因此 DYA Studio 的 Trackball Tuning 与相关模块（PMW3610 驱动）不适用。

### 左右半分配

**本键盘的 central（主控）接在左手** —— USB 与 BLE 都接左手，DYA Studio 网页也只会连上左手。

| | 左手 · central | 右手 · peripheral |
| --- | --- | --- |
| USB / BLE 连接 | 是 | — |
| Studio 通信 | 是 | — |
| 连接管理、OS 识别、按 OS 切层 | 是 | — |
| 设置中继 | 发起 | 接收并应用 |
| 抖键诊断、看门狗 | 汇总两半 | 检测本半并上报 |

> ZMK 默认约定是右手为主控。如果你的接线是 USB 插右手，把
> `boards/shields/corne/Kconfig.defconfig` 里的 `SHIELD_CORNE_LEFT` 改成
> `SHIELD_CORNE_RIGHT`，并把两份 conf 的模块归属对调即可。

### Studio 锁定

默认固件在断开连接后会锁定 Studio，重新连接时需要按住指定按键解锁。
如果只是自己用、不担心他人改动配置，可以刷 `*_studio_unlocked` 版本关闭锁定。

## 固件变体

Actions → 最新一次构建 → Artifacts → `zmk-firmware`，内含以下 `.uf2`：

| Artifact | 说明 | 体积 |
| --- | --- | --- |
| `corne_left` / `corne_right` | **无屏版**，flash 余量最大，推荐日常使用 | 458 / 374 KB |
| `corne_left_oled` / `corne_right_oled` | SSD1306 OLED 显示 | 726 / 641 KB |
| `corne_left_niceview` / `corne_right_niceview` | nice!view 适配板 | 781 / 627 KB |
| `corne_left_studio_unlocked` / `corne_right_studio_unlocked` | 无屏 + Studio 常开（免解锁） | 458 / 374 KB |
| `settings_reset` | 清除所有持久化设置（含 DYA Studio 保存的配置） | 103 KB |

体积以 UF2 文件大小为准，nice!nano 内部 flash 为 1 MB，9 个变体都能装下。
带屏幕的两个版本余量较小，后续若想再加功能优先选无屏版。

左右半分别刷对应文件。`settings_reset` 刷完后再刷正常固件即可。

### 显示接线

两个显示变体都对应板上的排针座，选一个刷就行：

- **OLED 变体** — 插 4 针 PH5 座，I²C，地址 `0x3C`，128×32
- **nice!view 变体** — 插 5 针 PH5 座，SPI（SCK P0.20 / MOSI P0.17 / MISO P0.25 / CS P1.1）
  自带 `CONFIG_SSD1306=n`，不会和 OLED 驱动打架

## 本地构建

需要 [Zephyr SDK](https://docs.zephyrproject.org/latest/develop/toolchains/index.html) 和
[`west`](https://docs.zephyrproject.org/latest/develop/west/install.html)。

```bash
# 依赖下载到 ./dependencies
west init -l config
# 或下载到仓库上一级（多个 config 仓库共享同一份 ZMK）
# west init -l . --mf config/west-workspace.yml

west update --narrow
west zephyr-export
west zmk-build                 # 构建 build.yaml 里的全部变体
west zmk-build -a corne_right  # 只构建某一个
west zmk-build --flash         # 构建后直接烧录
```

固件输出在 `./build/<artifact>/zephyr/zmk.uf2`。

## 目录结构

```
config/
  corne.keymap            键位定义（分层布局，含注释图示）
  corne.json              keymap-editor 用的物理布局描述
  corne.conf              通用配置（蓝牙功率、底灯开关）
  west-dependency.yml     依赖清单：ZMK fork + DYA Studio 模块栈
  west.yml                默认 west 入口
  west-isolated.yml       依赖装到 ./dependencies
  west-workspace.yml      依赖装到 west topdir
boards/shields/corne/
  Kconfig.defconfig       键盘名、分体角色（central = 左手）
  Kconfig.shield          shield 标识
  boards/                 板级 overlay（nice!nano）
  corne_left.conf         左手（central）全部 Kconfig
  corne_right.conf        右手（peripheral）全部 Kconfig
snippets/
  corne-oled/             OLED 显示
  corne-no-display/       禁用板载 OLED，回收 flash
zephyr/module.yml         告诉 west 在哪里找 boards / snippets
build.yaml                构建矩阵
```

## 依赖来源

本仓库不使用 ZMK 官方 `main` / `v0.3`，而是 [cormoran/zmk](https://github.com/cormoran/zmk)
的 `main+dya` 分支 —— 它在 ZMK main 之上加了 DYA Studio 所需的
自定义 Studio RPC 协议与分体中继支持。**DYA Studio 的扩展功能依赖这个 fork**，
换回官方 ZMK 只能保留基础的 keymap 编辑功能。

> ⚠️ cormoran 的 ZMK fork 为实验性分支，针对 DYA 键盘优化，可能包含不稳定的变更。
> 自己用着没问题，但不建议用于需要高稳定性的场合。仓库配置了每周五一次的
> 定时构建，用于监控上游变更带来的破坏。

## License

MIT

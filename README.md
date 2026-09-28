# zmk-config-corne

Corne 分体键盘的 ZMK 固件配置，已接入 [DYA Studio](https://studio.dya.cormoran.works/)。

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
west zmk-build            # 构建 build.yaml 里的全部变体
west zmk-build -a corne_right   # 只构建某一个
west zmk-build --flash    # 构建后直接烧录
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

# zmk-config-corne

**English** · [简体中文](./README.md)

ZMK firmware configuration for the Corne split keyboard, with
[DYA Studio](https://studio.dya.cormoran.works/) support.

## About Corne

[Corne](https://github.com/foostan/crkbd) (also known as **crkbd**, named after the
cornetto pastry) is an open-source split ergonomic keyboard designed by foostan
(Kosuke Adachi) in Japan, first released in April 2018, inspired by the Helix keyboard.

Its defining characteristics:

- **3×6 column-staggered keys plus 3 thumb keys** per half, 42 keys total. Column
  staggering lets each finger move up and down its own natural column instead of
  diagonally — easier on the wrist than row staggering, and a layout that has since
  inspired a wide range of similar designs
- **Two symmetric PCBs joined by a TRRS cable** — the halves are identical, which keeps
  unit cost low and soldering simple
- **Open source from day one** — anyone can order boards, remap keys, or design a case
- **Removable outer column** — on most variants the outer column snaps off, turning the
  board into a 34-key 3×5+3 layout

Because the design is open and approachable, Corne became one of the most popular split
keyboard designs of the last decade, spawning the official Classic / Cherry / Chocolate
/ Light branches along with a long list of community variations.

This repo targets the **V4 outline with a Pro Micro footprint** — see below.

## Supported hardware

**[Corne V4 Pro-Micro Edition](https://github.com/klouderone/CorneV4ProMicroEdition)**
by Kea Workshop, maintained by klouderone. It uses the foostan Corne V4 outline but
keeps the Pro Micro footprint, so it accepts a [nice!nano v2](https://nicekeyboards.com/nice-nano)
for wireless or an Elite-C v4 for wired.

| Item | Detail |
| --- | --- |
| Microcontroller | nice!nano v2 (this config's target) / other Pro Micro footprint boards |
| Sockets | Kailh **MX** and **Choc** hotswap both supported; outer column snaps off for a 5-column build |
| Diodes | SMD 1N4148 and through-hole diodes both supported |
| Display | 0.91" OLED (I²C) or a nice!view adapter board |
| Connectivity | 3.5mm TRRS interconnect, plus battery pads / PH2 connector / power switch |
| Underglow | ❌ No RGB underglow, so underglow is disabled in the config |
| Key pitch | 19.05mm (V4 nudged from 19mm on V3) |

> **Don't buy the wrong board.** foostan's official Corne V4 carries an **on-board RP2040**
> with USB-C. It takes neither a nice!nano nor this repo's wireless firmware. Only the
> Kea Workshop Pro-Micro edition accepts Pro Micro footprint boards.

> This config compiles the **MX** matrix. The PCB's Choc sockets are hardware-compatible,
> but Choc uses a different row/column pin assignment and ZMK's stock `corne` shield only
> defines MX. A Choc build needs a separate matrix configuration, which this repo does not
> include.

> ⚠️ The board supports both wired and wireless: **don't use the TRRS jack while a battery
> is connected**, and **don't connect a battery when using the wired connection**. The
> firmware here targets the wireless (nice!nano) build.

## DYA Studio

Open <https://studio.dya.cormoran.works/> and pick a connection on the splash screen:

| Method | How | Supported browsers |
| --- | --- | --- |
| USB | Plug in and pick the serial port (Web Serial) | Chrome / Edge (desktop) |
| Bluetooth | Pair and connect over BLE (Web Bluetooth) | Chrome / Edge (desktop), Chrome on Android, Bluefy on iOS |
| Demo | No keyboard needed — explore with a simulated board | same as above |

Firefox and Safari are not supported.

### Available features

- **Keymap Editor** — visual key and layer editing with live key press highlighting
- **Macros & Combos** — create macros and combos from the browser, **no firmware rebuild**
- **Connection Management** — rename, switch, and unpair BLE profiles; choose whether USB
  or Bluetooth wins when both are connected
- **Device Settings** — power management (idle / deep sleep timeouts), per-half settings
- **Troubleshooting** — battery level, firmware build, uptime; per-key chatter detection;
  copy a full support report

> **Per-OS layer switching** requires OS Detection and the Default Layer module, both
> enabled on the central half. Corne has no trackball, so DYA Studio's Trackball Tuning
> and the related PMW3610 driver do not apply.

### Which half is which

**The central half is the left one on this build** — both USB and BLE plug into the left
half, and DYA Studio will only ever see the left half.

| | Left · central | Right · peripheral |
| --- | --- | --- |
| USB / BLE connection | yes | — |
| Studio communication | yes | — |
| Connection management, OS detection, per-OS layers | yes | — |
| Settings relay | originates | receives and applies |
| Chatter diagnostics, watchdog | aggregates both halves | measures and reports its own half |

> ZMK's convention is to put the central on the right. If your cable is on the right,
> change `SHIELD_CORNE_LEFT` to `SHIELD_CORNE_RIGHT` in
> `boards/shields/corne/Kconfig.defconfig` and swap the module ownership between the two
> conf files.

### Studio locking

The default firmware locks Studio after disconnecting; reconnecting requires holding a
key to unlock. If it is only you using the board and you do not mind others changing your
settings, flash a `*_studio_unlocked` variant to disable the lock.

## Firmware variants

Actions → latest build → Artifacts → `zmk-firmware`, containing these `.uf2` files:

| Artifact | Description | Size |
| --- | --- | --- |
| `corne_left` / `corne_right` | **No display**, most flash headroom — the everyday pick | 458 / 374 KB |
| `corne_left_oled` / `corne_right_oled` | SSD1306 OLED display | 726 / 641 KB |
| `corne_left_niceview` / `corne_right_niceview` | nice!view adapter board | 781 / 627 KB |
| `corne_left_studio_unlocked` / `corne_right_studio_unlocked` | No display, Studio always unlocked | 458 / 374 KB |
| `settings_reset` | Wipes all persisted settings, including DYA Studio's | 103 KB |

Sizes are UF2 file sizes. nice!nano has 1 MB of internal flash and all nine variants fit.
The two display variants have the least headroom, so prefer the no-display build if you
want to add more features later.

Flash the matching file to each half. After flashing `settings_reset`, flash the normal
firmware again.

### Display wiring

Both display variants map to a header on the PCB — pick one:

- **OLED variant** — 4-pin PH5 header, I²C, address `0x3C`, 128×32
- **nice!view variant** — 5-pin PH5 header, SPI (SCK P0.20 / MOSI P0.17 / MISO P0.25 /
  CS P1.1). It sets `CONFIG_SSD1306=n` itself, so it does not conflict with the OLED driver

## Building locally

Requires the [Zephyr SDK](https://docs.zephyrproject.org/latest/develop/toolchains/index.html)
and [`west`](https://docs.zephyrproject.org/latest/develop/west/install.html).

```bash
# Dependencies go to ./dependencies
west init -l config
# Or into the parent directory (several config repos sharing one ZMK checkout)
# west init -l . --mf config/west-workspace.yml

west update --narrow
west zephyr-export
west zmk-build                 # build every variant in build.yaml
west zmk-build -a corne_right  # build a single variant
west zmk-build --flash         # build, then flash
```

Firmware lands in `./build/<artifact>/zephyr/zmk.uf2`.

## Repository layout

```
config/
  corne.keymap            Key definitions (layered, with ASCII layout comments)
  corne.json              Physical layout description for keymap-editor
  corne.conf              Shared config (BT power, underglow switch)
  west-dependency.yml     Dependency list: ZMK fork + DYA Studio module stack
  west.yml                Default west entry point
  west-isolated.yml       Dependencies land in ./dependencies
  west-workspace.yml      Dependencies land in the west topdir
boards/shields/corne/
  Kconfig.defconfig       Keyboard name, split role (central = left)
  Kconfig.shield          Shield identification
  boards/                 Board-level overlay (nice!nano)
  corne_left.conf         Full Kconfig for the left half (central)
  corne_right.conf        Full Kconfig for the right half (peripheral)
snippets/
  corne-oled/             OLED display
  corne-no-display/       Disables the on-board OLED to reclaim flash
zephyr/module.yml         Tells west where to find boards / snippets
build.yaml                Build matrix
```

## Where the dependencies come from

This repo does not use upstream ZMK `main` or `v0.3`. It uses the `main+dya` branch of
[cormoran/zmk](https://github.com/cormoran/zmk), which layers the custom Studio RPC
protocol and split event relay that DYA Studio's extended features need on top of ZMK
main. **Those extended features depend on this fork** — going back to upstream ZMK leaves
only the basic keymap editor.

> ⚠️ cormoran's ZMK fork is experimental and tuned for DYA keyboards, so it may contain
> unstable or breaking changes. It is fine for personal use, but not advisable where
> stability matters. The repo runs a weekly build to catch upstream breakage.

## License

MIT

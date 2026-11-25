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
| **Which firmware to flash** | **`corne_left*`** | **`corne_right*`** |
| USB / BLE connection | yes | — |
| Studio communication | yes | — |
| Connection management, OS detection, per-OS layers | yes | — |
| Settings relay | originates | receives and applies |
| Chatter diagnostics, watchdog | aggregates both halves | measures and reports its own half |

This matches ZMK's own corne shield (and `cormoran/zmk`'s `main+dya` branch):
`boards/shields/corne/Kconfig.defconfig` contains `if SHIELD_CORNE_LEFT →
ZMK_SPLIT_ROLE_CENTRAL default y`. **ZMK's corne puts the central on the left, not
the right.**

> ⚠️ **Flashing the wrong half leaves the board with no input at all** (central and
> peripheral no longer pair). Remember: **the half your USB cable goes into is the
> left one, so flash `corne_left*`.**
>
> If your wiring really does put USB on the right half, change three things
> together:
> 1. `SHIELD_CORNE_LEFT` → `SHIELD_CORNE_RIGHT` in `Kconfig.defconfig`;
> 2. rename the two snippet directories (`corne-central` ↔ `corne-peripheral`)
>    and update the names in each variant's `snippets:` list in `build.yaml`;
> 3. swap `shield:` and `artifact:` for every variant in `build.yaml`.

### Studio locking

Firmware in this repo is **unlocked by default**:
`snippets/corne-central/corne-central.conf` sets `CONFIG_ZMK_STUDIO_LOCKING=n`,
so Studio stays unlocked after disconnecting or idling and Web Bluetooth / WebUSB
can connect without pressing a key.

If you remove that setting to restore the default lock-on-disconnect behaviour,
browser Web Bluetooth cannot see the keyboard while locked — press the fallback
unlock key first:

```
Enter RAISE : the 7th key of LAYER_0 (right thumb #2) — tap for backspace, hold for RAISE
Unlock      : left thumb #1 on RAISE (the GUI key)
Backup      : left thumb #3 on ADJUST (the SPC key)
```

> Layer navigation (every layer also has an `&mo 0` escape hatch):
> LAYER_0 → LOWER with left thumb #2; LAYER_0 → RAISE with right thumb #2;
> RAISE → ADJUST with left thumb #2; LOWER + RAISE together jumps to ADJUST.
> The rightmost thumb key of LOWER / RAISE / ADJUST is `&mo 0`, which momentarily
> returns to LAYER_0 — so even if a layer is set as the base layer from DYA Studio,
> you can never get locked out of it.
> USB / BLE output switching lives on ADJUST (`&out OUT_TOG`); avoid `&out OUT_USB`,
> because once you switch to it without a cable plugged in the board produces no
> output at all.

## Firmware variants

Actions → latest build → Artifacts → `zmk-firmware`, containing these `.uf2` files:

| Artifact | Description |
| --- | --- |
| `corne_left_oled` / `corne_right_oled` | SSD1306 OLED display |
| `corne_left_niceview` / `corne_right_niceview` | nice!view adapter board |
| `settings_reset` | Wipes all persisted settings, including DYA Studio's |

Flash the matching file to each half. After flashing `settings_reset`, flash the normal
firmware again.

### Display wiring

Both display variants map to a header on the PCB — pick one:

- **OLED variant** — 4-pin PH5 header, I²C, address `0x3C`, 128×32
- **nice!view variant** — 5-pin PH5 header, SPI (SCK P0.20 / MOSI P0.17 / MISO P0.25 /
  CS P1.1). It sets `CONFIG_SSD1306=n` itself, so it does not conflict with the OLED driver

## ZMK KeyTester (Physical Matrix Tester)

This repo integrates the [tangbonze/zmk-matrix-tester](https://github.com/tangbonze/zmk-matrix-tester)
module for hardware-level WebHID physical matrix testing with
[ZMK KeyTester](https://github.com/tangbonze/zmk-keyboard-tester)
(without enabling `CONFIG_ZMK_USB_LOGGING=y`).

1. Connect the left half (central) to your computer via USB (desktop Chrome / Edge).
2. Open the preloaded tester page for this repository:
   <https://tangbonze.github.io/zmk-keyboard-tester/?config=https://github.com/tangbonze/zmk-config-corne>
3. Click **WebHID Physical Matrix** at the top and select your keyboard to test every physical switch on both halves, with optional silent intercept mode.

> The module is enabled on the left half (central) only via a dedicated Raw HID interface
> (`HID_1`, `0xFF60 / 0x61`), leaving normal keyboard HID and BLE untouched with zero battery impact when unplugged.

## Building locally

Requires the [Zephyr SDK](https://docs.zephyrproject.org/latest/develop/toolchains/index.html)
and [`west`](https://docs.zephyrproject.org/latest/develop/west/install.html).

You need **Python 3.12** (Zephyr 4.1 does not support 3.13/3.14) and Zephyr SDK
0.17.0:

```bash
brew install python@3.12
python3.12 -m venv .venv && ./.venv/bin/pip install west
export ZEPHYR_SDK_INSTALL_DIR=~/zephyr-sdk-0.17.0

# Dependencies go to ./dependencies
west init -l config
# Or into the parent directory (several config repos sharing one ZMK checkout)
# west init -l . --mf config/west-workspace.yml

west update --narrow
west zephyr-export
# Studio's nanopb generator needs these two, otherwise proto generation fails
./.venv/bin/pip install "protobuf>=5.27" grpcio-tools

west zmk-build                       # build every variant in build.yaml
west zmk-build -a corne_right_oled   # build a single variant

# Flashing requires a single target (with several, --flash exits with an error)
west zmk-build -a corne_left_oled --flash

# While debugging: build a variant with USB logging (costs flash, daily builds unaffected)
west zmk-build -a corne_left_oled -S zmk-usb-logging --flash
```

Firmware lands in `./build/<artifact>/zephyr/zmk.uf2`.

## Troubleshooting

A successful build does not mean every feature got enabled. The classic example: if
DYA Studio's Kconfig is not actually loaded, the firmware still builds and types
fine, but the whole Studio RPC stack never gets linked in — which looks exactly
like "Studio won't connect".

**1. Check the build log for Kconfig warnings — this repo currently has none.**
Every `assigned the value 'y' but got 'n'` means some feature did not get enabled;
the warning names the symbol:

```bash
grep -iE "warning: (assigned|attempt to assign|choice symbol)" \
  build/corne_left_oled/stdout_and_stderr.log
```

**2. Confirm which `.conf` files the build actually merged.** This is the fastest
way to find "my config was never read": if the list shows
`dependencies/zmk/app/boards/shields/corne/corne_left.conf` (an empty file in the
ZMK fork) instead of this repo's snippet, the config was never picked up.

```bash
grep -oE "[^ ']*\.conf" build/corne_left_oled/stdout_and_stderr.log | sort -u
```

A correct run shows `snippets/corne-central/corne-central.conf` in that list.

**2. Verify the effective `.config`.** All of the following must be `y`; a single
`n` means that feature is off. The most critical one is
`ZMK_BEHAVIOR_LOCAL_ID_TYPE_CRC16`: it must match what the DYA Studio web UI
computes, otherwise the IDs behind Macros / Combos / Keymap Editor writes do not
line up with the existing bindings and edits appear to do nothing.

```bash
grep -E "ZMK_STUDIO|PHYSICAL_LAYOUTS|FAST_KEYMAP|KSCAN_DIAG|WATCHDOG|OS_DETECTION|DEFAULT_LAYER|BLE_MANAGEMENT|RUNTIME_(MACRO|COMBO|INPUT)|INPUT_STREAM|DEVICE_INFO|LOCAL_ID_TYPE|ZMK_MATRIX_TESTER" \
  build/corne_left_oled/zephyr/.config | grep -v "=n$"
```

**3. Capture device logs.** Daily builds turn off `CONFIG_CONSOLE` to save flash;
use `-S zmk-usb-logging` when you need logs.

**4. No input at all?** First confirm you did not flash the halves swapped: the
half your USB cable is in must be running `corne_left*`.

## Repository layout

```
config/
  corne.keymap            Key definitions (layered, with ASCII layout comments)
  corne.json              Layout description for external keymap-editor / keymap-drawer
                          (it is not compiled, and it does not decide the keyboard
                          drawing in DYA Studio — that comes from the physical-layout
                          built into the ZMK fork)
  corne.conf              Kconfig shared by both halves (BT, battery, sleep, stacks)
  west-dependency.yml     Dependency list: ZMK fork + DYA Studio module stack + zmk-matrix-tester
  west.yml                Default west entry point
  west-isolated.yml       Dependencies land in ./dependencies
  west-workspace.yml      Dependencies land in the west topdir
boards/shields/corne/
  Kconfig.defconfig       Keyboard name, split role (central = left) and
                          inherited upstream defaults only
  Kconfig.shield          Shield identification
snippets/
  corne-central/          Left half (central) only: ZMK Studio and every
                          *_STUDIO_RPC, OS detection, default layer, connection
                          management, the extension modules
  corne-peripheral/       Right half (peripheral) only: receives and applies
                          relayed settings
  corne-oled/             OLED display
  corne-no-display/       Disables the on-board OLED to reclaim flash
zephyr/module.yml         Tells west where to find boards / snippets
build.yaml                Build matrix
```

> `corne.dtsi`, `corne_left.overlay`, `corne_right.overlay` and the physical layout
> (`foostan_corne_6col_layout`) all come from `cormoran/zmk`'s `main+dya` branch;
> this repo does not redefine them.

### Splitting config per half: why snippets instead of `<shield>.conf`

DYA Studio's Kconfig must be isolated per half — the `*_STUDIO_RPC` protobuf stubs
are only generated on the central, so enabling them on the right half breaks the
build with a missing `proto/zmk/custom.pb.h`. This repo does that isolation through
the `snippets:` list of each variant in `build.yaml`:

| File | Scope | Contents |
| --- | --- | --- |
| `Kconfig.defconfig` | shield | Keyboard name, split role, inherited `ZMK_SPLIT` / display / LVGL |
| `snippets/corne-central` | left half only | Studio / RPC / OS detection / default layer / extension modules |
| `snippets/corne-peripheral` | right half only | Receiving and applying relayed settings only |
| `config/corne.conf` | both halves | Bluetooth, battery, sleep, split event relay, thread stacks |

**The trap we hit: putting this in `boards/shields/corne/corne_left.conf` does not
work.** Zephyr resolves `SHIELD_ROOT` to the *first* board root containing the
shield, and ZMK's `app/boards` comes first, so the build log shows

```
dependencies/zmk/app/boards/shields/corne/corne_left.conf   ← an empty file in the fork
```

instead of the repo's copy. `Kconfig.defconfig` works only because Zephyr
`osource`s *every* board root. There are therefore only two ways to isolate config
per half:

1. `Kconfig.defconfig` with `default y` (what this repo used to do — it works, but
   an unmet dependency is dropped **silently, with no warning**, which easily
   produces "it compiles but the feature is off");
2. **snippets** (what this repo does now, via `EXTRA_CONF_FILE`, verified working).
   Those are explicit assignments, so unmet dependencies produce a clear warning,
   and they are isolated per half by construction.

The fastest way to check this is to look at which `.conf` files the build actually
merged:

```bash
grep -oE "[^ ']*\.conf" build/corne_left_oled/stdout_and_stderr.log | sort -u
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

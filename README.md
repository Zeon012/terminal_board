# KB60: ZMK for a hand-wired 64-key ANSI on a Teyleten nRF52840 Pro Micro

Adapted from [kanefron5/keyboard-manual](https://github.com/kanefron5/keyboard-manual) (QMK/RP2040, 5x14 matrix) to ZMK on a nice!nano-compatible nRF52840 board, USB only.

## Matrix
The original 5 rows x 14 cols wiring is kept as is: no cuts, no tied columns. That is 19 GPIO of the board's 20.

## Pin map
The overlay uses raw nRF52840 GPIO names taken from the board's silkscreen (`0xx` = `&gpio0 xx`, `1xx` = `&gpio1 xx`), so it does not depend on the clone's Pro Micro numbering.

Lines follow the pins in physical order, starting from the Backspace side: column 13 on the first pin of the left edge at the USB end, down that edge to column 4 (skipping the two GND pins), then columns 3-0 down the right edge (after RAW, GND, RST, VCC), then rows 0-3. Column 0 is the Esc column.

| Line | Silk | Where |
|---|---|---|
| Col 0 | 115 | right edge, 8th pin from the USB end |
| Col 1 | 002 | right edge, 7th pin from the USB end |
| Col 2 | 029 | right edge, 6th pin from the USB end |
| Col 3 | 031 | right edge, 5th pin from the USB end |
| Col 4 | 106 | left edge, 12th pin from the USB end |
| Col 5 | 104 | left edge, 11th pin from the USB end |
| Col 6 | 011 | left edge, 10th pin from the USB end |
| Col 7 | 100 | left edge, 9th pin from the USB end |
| Col 8 | 024 | left edge, 8th pin from the USB end |
| Col 9 | 022 | left edge, 7th pin from the USB end |
| Col 10 | 020 | left edge, 6th pin from the USB end |
| Col 11 | 017 | left edge, 5th pin from the USB end |
| Col 12 | 008 | left edge, 2nd pin from the USB end |
| Col 13 | 006 | left edge, 1st pin from the USB end |
| Row 0 | 113 | right edge, 9th pin from the USB end |
| Row 1 | 111 | right edge, 10th pin from the USB end |
| Row 2 | 010 | right edge, 11th pin from the USB end |
| Row 3 | 009 | right edge, 12th pin from the USB end |
| Row 4 | 101 | inner pad on the underside |
| Status LED data in | 107 | inner pad; one WS2812B, powered from VCC and GND |

Left unused: 102 (inner pad). Notes:
- The status LED takes its power from VCC, not RAW: the board switches VCC off in sleep, and a dark WS2812B still draws about 1 mA, which would drain the battery.
- 009 and 010 are the NFC pins; ZMK's nice!nano board config turns them into normal GPIO.
- 006 and 008 are TX/RX, and 101/102/107 are small inner pads, so these are the easiest to get wrong. Confirm the silkscreen on your board before soldering.
- The pad labelled `101` was read from a photo where it overlaps `011`; check it against the board.

## Layout
64 keys, as in the original design's `LAYOUT_64_ansi`: 14/14/13/14/9, with `` ` ``, Up and Del at the end of the Shift row and Left, Down, Right at the end of the bottom row. Matrix positions follow the original: Enter on col 13, Space on col 5, bottom-right keys on cols 9-13. If your wiring differs, edit the `map` and `keys` in `kb60.overlay` and the `bindings` in `config/kb60.keymap`, which must stay in the same order.

## Diode direction
The firmware scans `col2row`: current flows from the column wire, through the switch and diode, to the row wire, so each diode's stripe faces the row wire. If your diodes face the other way, change `diode-direction` in `kb60.overlay` to `"row2col"` and move the pull-downs from the rows to the columns.

## Fn layer
Hold Fn (right of RAlt):
- Number row: `` ` `` and F1-F12, Backspace = Del.
- P [ ]: Print Screen, Scroll Lock, Pause. ; and ': Home, Page Up. / and `` ` ``: End, Page Down. . : Insert.
- Up = volume up, Down = volume down, Del = mute, Left/Right = previous/next track, Space = play/pause.
- `\` = ZMK Studio unlock, needed before the configurator can change keys.
- Q W E = Bluetooth profile 1, 2, 3. R = clear the selected profile (puts it in pairing mode). T = switch output between USB and Bluetooth. Y = Bluetooth off: switches to spare profile 3 and disconnects profile 1, so the computer stays disconnected; Fn + Q reconnects.
- Z = bootloader.

Bluetooth and USB are both enabled, with battery monitoring. To pair: hold Fn, press Q to select profile 1, press R to clear it, then add "KB60" in the computer's Bluetooth settings. The status LED blinks fast in pairing mode and flashes every two seconds while a paired computer is not connected, in the profile's colour (1 blue, 2 cyan, 3 magenta), and shows solid for 1.5 s on connecting. Otherwise it is green on USB power and red on battery, blinking red when the battery is low; on battery it goes dark while the keyboard is idle.

Switching profiles disconnects the other computers (`CONFIG_MACROPAD_BLE_EXCLUSIVE`), so only the selected one shows the keyboard as connected. Switching back reconnects within a second or two.

**Boot key:** hold Esc while switching the keyboard on (or plugging it in, if it has no battery) to get the `NICENANO` flashing drive without reaching the reset pins.

## Firmware base
This branch builds on cormoran's fork of ZMK, which adds an unofficial custom Studio RPC channel, with the runtime macro and key stream modules. All are pinned to exact commits in `config/west.yml`. That is what lets the configurator create macros and run its key tester on this keyboard. Flash `settings_reset` first when moving between this firmware and official ZMK, because saved key changes are not compatible between the two.

## Configuring
ZMK Studio is enabled over USB. Use the configurator in the `zmk-macropad` repository, or https://zmk.studio: connect, then hold Fn and press `\` to unlock.

**VIA** (https://usevia.app) works too, over USB or Bluetooth:

1. In VIA's Settings, turn on **Show Design tab**.
2. In Design, load `via/kb60.json`.
3. Authorize the keyboard, and unlock it (Fn + `\`) before changing anything.

VIA can remap keys, write VIA macros (M0-M15) and run its Test Matrix. Anything VIA has no keycode for shows as **ZMK** and is left alone. The VIA code lives in the macropad module; see `via/README.md` in `zmk-macropad` for details.

This keyboard has its own USB and Bluetooth ID, 1209:0001 (the macropad uses ZMK's 1D50:615E), so VIA keeps a separate definition for each. The configurator 1.0.5 or later finds it over USB.

## Build and flash
1. Push this folder to a GitHub repository and run the **Build ZMK firmware** Action.
2. Download the artifact. Double-tap reset on the board so the `NICENANO` drive appears.
3. Copy `settings_reset-nice_nano.uf2` first, let it reboot, then double-tap reset and copy `kb60-nice_nano.uf2`.

## Verify before flashing
With the controller disconnected, use a multimeter in continuity mode: pressing each key should connect its expected row line and column line.

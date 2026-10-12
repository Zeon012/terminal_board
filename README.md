# KB60: ZMK for a hand-wired 64-key ANSI on a Teyleten nRF52840 Pro Micro

Adapted from [kanefron5/keyboard-manual](https://github.com/kanefron5/keyboard-manual) (QMK/RP2040, 5x14 matrix) to ZMK on a nice!nano-compatible nRF52840 board, USB only.

## Matrix
The original 5 rows x 14 cols wiring is kept as is: no cuts, no tied columns. That is 19 GPIO of the board's 20.

## Pin map
The overlay uses raw nRF52840 GPIO names taken from the board's silkscreen (`0xx` = `&gpio0 xx`, `1xx` = `&gpio1 xx`), so it does not depend on the clone's Pro Micro numbering.

| Line | Silk | Line | Silk |
|---|---|---|---|
| Col 0 | 031 | Col 7 | 009 |
| Col 1 | 029 | Col 8 | 106 |
| Col 2 | 002 | Col 9 | 104 |
| Col 3 | 115 | Col 10 | 011 |
| Col 4 | 113 | Col 11 | 100 |
| Col 5 | 111 | Col 12 | 024 |
| Col 6 | 010 | Col 13 | 022 |
| Row 0 | 006 | Row 3 | 020 |
| Row 1 | 008 | Row 4 | 101 (inner pad on the underside) |
| Row 2 | 017 | | |

Left unused: 102 and 107 (inner pads). Notes:
- 009 and 010 are the NFC pins; ZMK's nice!nano board config turns them into normal GPIO.
- 006 and 008 are TX/RX, and 101/102/107 are small inner pads, so these are the easiest to get wrong. Confirm the silkscreen on your board before soldering.
- The pad labelled `101` was read from a photo where it overlaps `011`; check it against the board.

## Layout
64 keys, as in the original design's `LAYOUT_64_ansi`: 14/14/13/14/9, with `` ` ``, Up and Del at the end of the Shift row and Left, Down, Right at the end of the bottom row. Matrix positions follow the original: Enter on col 13, Space on col 5, bottom-right keys on cols 9-13. If your wiring differs, edit the `map` and `keys` in `kb60.overlay` and the `bindings` in `config/kb60.keymap`, which must stay in the same order.

## Diode direction
The firmware assumes `col2row` (columns on the switch pin, rows through the diode), like the original build. If your diodes face the other way, change `diode-direction` in `kb60.overlay` to `"row2col"` and swap the pull to the columns.

## Fn layer
Hold Fn (right of RAlt):
- Number row: `` ` `` and F1-F12, Backspace = Del.
- P [ ]: Print Screen, Scroll Lock, Pause. ; and ': Home, Page Up. / and `` ` ``: End, Page Down. . : Insert.
- Up = volume up, Down = volume down, Del = mute, Left/Right = previous/next track, Space = play/pause.
- `\` = ZMK Studio unlock, needed before the configurator can change keys.
- Z = bootloader.

Bluetooth is disabled (USB only).

## Configuring
ZMK Studio is enabled over USB. Use the configurator in the `zmk-macropad` repository, or https://zmk.studio: connect, then hold Fn and press `\` to unlock.

## Build and flash
1. Push this folder to a GitHub repository and run the **Build ZMK firmware** Action.
2. Download the artifact. Double-tap reset on the board so the `NICENANO` drive appears.
3. Copy `settings_reset-nice_nano.uf2` first, let it reboot, then double-tap reset and copy `kb60-nice_nano.uf2`.

## Verify before flashing
With the controller disconnected, use a multimeter in continuity mode: pressing each key should connect its expected row line and column line.

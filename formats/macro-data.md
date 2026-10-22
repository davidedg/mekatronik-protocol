# Macro data (requests `12` and `13`)

The data of the 20 macro keys of a profile, one after the other in the order of the [macro index](macro-index.md), which gives the offset and length of each. Written with [`12`](../host-to-device/12/12.md) in blocks of at most 4 KiB, read back with [`13`](../host-to-device/13/13.md).

What a key's data is depends on its function in the index.

## Macro

Function `20`. A list of events of **3 bytes** each, played in order, closed by an end event. Seen:

| Bytes | Event |
|---|---|
| `84 uu 01` | press of the key with HID usage `uu` |
| `84 uu 00` | release of the key with HID usage `uu` |
| `87 hh ll` | delay of `hh ll` (big-endian) × **50 ms** |
| `86 00 nn` | end of the macro; play it `nn` times in all |

- **Delays are in units of 50 ms.** The vendor editor works in milliseconds and rounds down to a multiple of 50 before writing: 450 ms is `87 00 09`, 100 ms `87 00 02`, 1000 ms `87 00 14`, 130 ms becomes 100 ms, 23 ms becomes 0. A delay of 0 is written as `87 00 00`. **The editor caps a delay at 12750 ms** (255 × 50; 99999999 typed in the delay field became 12750), so it only ever writes `87 00 nn`; whether the firmware honours a high byte (delays above 12.75 s) has not been tried. Playback timing was checked: `87 00 03` and `87 00 26` gave 150 ms and 1.9 s between the reports.
- Presses may overlap: 1↓ 3↓ 3↑ 1↑ is written as it is.
- **The end event carries the repeat count**: `86 00 01` for "once", `86 00 03` for "3 times". The other repeat modes (while held, until pressed again) are not here but in the [macro-key map](key-map.md), and leave `86 00 01`.
- The Windows software writes `87 00 01` after the end event, inside the length of the key. The firmware does not seem to play it *(inferred)*.
- The Windows software writes a delay after each event, also before the end event, except when its macro file has none; both forms play.
- Right-hand modifiers work in macros (right Shift + 1 typed "!"). AltGr is recorded by Windows as left Ctrl + right Alt (`84 e0 01` `84 e6 01` ...), and the macro plays it back the same way.
- The keys are played on the keyboard's NKRO interface in Game mode and on its boot interface in Normal mode ([`83`](../device-to-host/83/83.md), [`81`](../device-to-host/81/81.md)).
- **In Game mode a usage above `6f` ends the macro.** A macro holding `85` played its keys up to it, sent one empty NKRO report in its place and nothing after it; the same macro in Normal mode played to the end (`85` typed nothing there). Usage `66` was sent in both modes and typed nothing.

### Predefined actions

The vendor editor offers a few "actions" (copy, page up, ...). They are not special codes: the software expands them into key events before writing, with `87 00 01` (50 ms) between the events of an action and the action's own delay after its last event.

```
Copy:     84 e0 01 87 00 01 84 06 01 87 00 01 84 06 00 87 00 01 84 e0 00 87 00 00
Page Up:  84 4b 01 87 00 01 84 4b 00 87 00 00
```

### Examples

```
"a" then "b": a↓, 150 ms, a↑, 1900 ms, b↓, 50 ms, b↑, once
84 04 01 87 00 03 84 04 00 87 00 26 84 05 01 87 00 01 84 05 00 86 00 01 87 00 01

"1" and "2", 3 times
84 1e 01 87 00 01 84 1e 00 87 00 14 84 1f 01 87 00 01 84 1f 00 87 00 00 86 00 03 87 00 01

1↓ 450 ms, 3↓ 100 ms, 3↑ 100 ms, 1↑, then Copy and Page Up, once
84 1e 01 87 00 09 84 20 01 87 00 02 84 20 00 87 00 02 84 1e 00 87 00 00
84 e0 01 87 00 01 84 06 01 87 00 01 84 06 00 87 00 01 84 e0 00 87 00 00
84 4b 01 87 00 01 84 4b 00 87 00 00 86 00 01 87 00 01
```

## Launch

Function `11`. Two bytes: **`kk fd`**, `kk` being the number of the key counted from 1 (T3 is `03`). When the key is pressed the keyboard sends the consumer report `fd kk` ([`82`](../device-to-host/82/82.md)) and the host program starts the program it has stored for that key. The keyboard knows nothing of the program; without host software the key does nothing.

```
T3: 03 fd
T13 (here F11): 0d fd
```

## Single key

Function `10`: the macro key types one other key. The first byte is the HID usage of that key.

```
T8 types X (Windows):  1b fb 1f
```

The Windows software writes 3 bytes, `uu` and two more that change from one write to the next for the same key (`fb 1f`, `d3 19`, `d6 19`, `dc 19`): they look like leftover memory, and only the first byte matters *(inferred)*. The keyboard typed X with each of them; the usage alone (1 byte) was not tried. A single key with no key chosen was written as `66 00 00`. The repeat byte of the key in the [macro-key map](key-map.md) stayed `00`.

## Limits

- The offsets of the [macro index](macro-index.md) are 16-bit. The largest profile written in these tests had 5724 bytes of data (four macros of 1410 bytes and one of 84), sent in two blocks ([`12`](../host-to-device/12/12.md)); it played correctly.

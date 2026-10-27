# Key matrix (requests `18` and `19`)

160 bytes, one per **position of the key matrix**: byte *i* is the key code that the key at position *i* produces. Written with [`18`](../host-to-device/18/18.md), read back with [`19`](../host-to-device/19/19.md), one table per profile.

- **The table is used only in Game mode** ([`02`](../host-to-device/02/02.md)). In Normal mode the keyboard types its default layout whatever the table of the profile holds; the T keys work in both modes.
- Positions `00` to `8f` are used; `90` to `9f` are `00` and the vendor software always sends them as `00`.
- Default: the table below, as all three profiles of the test unit held it before any change ([`19`](../host-to-device/19/19.md)) and as the vendor software writes it for a profile without remaps. It is probably the same for every layout *(inferred: other layouts were not seen)*: the Italian unit of these notes has the ISO keys `64` and `32`, and the table also holds the ANSI `\` key (`31`) and the Japanese, Korean and Brazilian keys (`87` to `91`, `85`), which this keyboard does not have.

## Writing a key code

| Code | Meaning |
|---|---|
| `04` to `e7` | the key produces this HID usage: **remap** ("Single Key" in the vendor software), e.g. Scroll Lock (position `5b`) set to `e0` types left Ctrl |
| `d8` + k, k = 0 to 7 | the key is **macro key T13+k**: it runs entry 12+k of the [macro index](macro-index.md) and the [macro-key map](key-map.md) |
| the default code | the key does what it is labelled |

Remaps outside the usual codes, in Game mode: a key set to `00` typed nothing; a key set to `70` (F21, above the `6f` of the NKRO report) typed nothing either and sent empty NKRO reports on press and release. In Normal mode both keys typed their labels (the matrix is not used there).

Keys the keyboard does not have work as remaps: Caps Lock (position `6f`) set to `68` (F13) typed F13 in Game mode, and Caps Lock in Normal mode. F14 to F20 (`69` to `6f`) are in the same range *(inferred)*.

Seen: position `5a` (F12) set to `d8` became T13 and ran its macro in Game mode, and typed F12 in Normal mode; position `5b` (Scroll Lock) set to `e0` typed left Ctrl, and that left Ctrl also triggered the Instant Shift of the profile ([`1a`](../host-to-device/1a/1a.md)): the firmware works on the code a key produces, not on the physical key.

The vendor software lets only the standard keys be changed, and only in its Game mode page; the T keys, the profile keys and the other dedicated keys keep their codes (`cc` to `d7`, `f1` to `f3`, and the codes marked as firmware codes below). What the firmware does with a dedicated key moved to another position, or with a firmware code that is not in the default table, is not known.

To turn a key back to normal, write its default code again: the Windows software sends the whole default table with only the changed positions replaced.

## Positions

The key names are those of the HID usages; the Italian legend of the test unit is given where it differs.

| Position | Default code | Key |
|---|---|---|
| `00` | `1d` | Z |
| `01` | `1e` | 1 |
| `02` | `cc` | **T1** (firmware code) |
| `03` | `b0` | firmware code, key not identified |
| `04` | `e6` | Right Alt (AltGr) |
| `05` | `b3` | firmware code, key not identified |
| `06` | `cd` | **T2** (firmware code) |
| `07` | `ed` | firmware code, key not identified |
| `08` | `06` | C |
| `09` | `20` | 3 |
| `0a` | `ce` | **T3** (firmware code) |
| `0b` | `b4` | firmware code, key not identified |
| `0c` | `b5` | firmware code, key not identified |
| `0d` | `cf` | **T4** (firmware code) |
| `0e` | `e4` | Right Ctrl |
| `0f` | `d0` | **T5** (firmware code) |
| `10` | `09` | F |
| `11` | `3a` | F1 |
| `12` | `3e` | F5 |
| `13` | `d1` | **T6** (firmware code) |
| `14` | `e2` | Left Alt |
| `15` | `d2` | **T7** (firmware code) |
| `16` | `d3` | **T8** (firmware code) |
| `17` | `d4` | **T9** (firmware code) |
| `18` | `28` | Enter |
| `19` | `3b` | F2 |
| `1a` | `3f` | F6 |
| `1b` | `d5` | **T10** (firmware code) |
| `1c` | `d6` | **T11** (firmware code) |
| `1d` | `e1` | Left Shift |
| `1e` | `d7` | **T12** (firmware code) |
| `1f` | `00` | (none) |
| `20` | `29` | Esc |
| `21` | `0e` | K |
| `22` | `13` | P |
| `23` | `f1` | **profile key 1** (firmware code) |
| `24` | `f2` | **profile key 2** (firmware code) |
| `25` | `f3` | **profile key 3** (firmware code) |
| `26` | `e0` | Left Ctrl |
| `27` | `b7` | firmware code, key not identified |
| `28` | `2c` | Space |
| `29` | `21` | 4 |
| `2a` | `87` | International1 (JIS Ro) |
| `2b` | `e3` | Left GUI (Windows) |
| `2c` | `b4` | firmware code, key not identified |
| `2d` | `8b` | International5 (Muhenkan) |
| `2e` | `8a` | International4 (Henkan) |
| `2f` | `88` | International2 (Katakana) |
| `30` | `04` | A |
| `31` | `3d` | F4 |
| `32` | `12` | O |
| `33` | `c6` | firmware code, key not identified |
| `34` | `c9` | firmware code, key not identified |
| `35` | `91` | LANG2 |
| `36` | `90` | LANG1 |
| `37` | `e7` | Right GUI (Windows) |
| `38` | `2b` | Tab |
| `39` | `35` | `` ` `` (IT `\`) |
| `3a` | `40` | F7 |
| `3b` | `1c` | Y |
| `3c` | `4d` | End |
| `3d` | `24` | 7 |
| `3e` | `33` | `;` (IT `ò`) |
| `3f` | `53` | Num Lock |
| `40` | `16` | S |
| `41` | `2d` | `-` (IT `'`) |
| `42` | `41` | F8 |
| `43` | `18` | U |
| `44` | `2f` | `[` (IT `è`) |
| `45` | `25` | 8 |
| `46` | `49` | Insert |
| `47` | `11` | N |
| `48` | `07` | D |
| `49` | `2e` | `=` (IT `ì`) |
| `4a` | `42` | F9 |
| `4b` | `0c` | I |
| `4c` | `30` | `]` (IT `+`) |
| `4d` | `26` | 9 |
| `4e` | `34` | `'` (IT `à`) |
| `4f` | `64` | Non-US `\` (IT `<`) |
| `50` | `1a` | W |
| `51` | `2a` | Backspace |
| `52` | `44` | F11 |
| `53` | `46` | Print Screen |
| `54` | `4a` | Home |
| `55` | `4c` | Delete |
| `56` | `0b` | H |
| `57` | `37` | `.` |
| `58` | `08` | E |
| `59` | `0f` | L |
| `5a` | `45` | F12 |
| `5b` | `47` | Scroll Lock |
| `5c` | `48` | Pause |
| `5d` | `4b` | Page Up |
| `5e` | `32` | Non-US `#` (IT `ù`) |
| `5f` | `38` | `/` (IT `-`) |
| `60` | `14` | Q |
| `61` | `0d` | J |
| `62` | `43` | F10 |
| `63` | `89` | International3 (Yen) |
| `64` | `31` | `\` (ANSI only) |
| `65` | `27` | 0 |
| `66` | `4e` | Page Down |
| `67` | `36` | `,` |
| `68` | `65` | Application (Menu) |
| `69` | `0a` | G |
| `6a` | `52` | Up |
| `6b` | `05` | B |
| `6c` | `5f` | Keypad 7 |
| `6d` | `5c` | Keypad 4 |
| `6e` | `59` | Keypad 1 |
| `6f` | `39` | Caps Lock |
| `70` | `19` | V |
| `71` | `22` | 5 |
| `72` | `54` | Keypad `/` |
| `73` | `50` | Left |
| `74` | `60` | Keypad 8 |
| `75` | `5d` | Keypad 5 |
| `76` | `5a` | Keypad 2 |
| `77` | `62` | Keypad 0 |
| `78` | `15` | R |
| `79` | `23` | 6 |
| `7a` | `55` | Keypad `*` |
| `7b` | `61` | Keypad 9 |
| `7c` | `51` | Down |
| `7d` | `5e` | Keypad 6 |
| `7e` | `5b` | Keypad 3 |
| `7f` | `63` | Keypad `.` |
| `80` | `17` | T |
| `81` | `3c` | F3 |
| `82` | `56` | Keypad `-` |
| `83` | `57` | Keypad `+` |
| `84` | `85` | Keypad `,` (Brazil) |
| `85` | `4f` | Right |
| `86` | `58` | Keypad Enter |
| `87` | `10` | M |
| `88` | `1b` | X |
| `89` | `1f` | 2 |
| `8a` | `ec` | firmware code, key not identified |
| `8b` | `b1` | firmware code, key not identified |
| `8c` | `eb` | firmware code, key not identified |
| `8d` | `e5` | Right Shift |
| `8e` | `b6` | firmware code, key not identified |
| `8f` | `b2` | firmware code, key not identified |
| `90` to `9f` | `00` | (not used) |

The thirteen firmware codes that are not identified (`b0` to `b7`, `c6`, `c9`, `eb`, `ec`, `ed`; `b4` is in two positions) belong to the other dedicated keys: Normal/Game switch, backlight, mute, volume up and down, and the four media keys *(inferred)*. Those keys are not remappable in the vendor software, and they reach the host as [notifications](../device-to-host/81/81.md) or [consumer reports](../device-to-host/82/82.md).

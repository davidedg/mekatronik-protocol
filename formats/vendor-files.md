# Files of the vendor software

What the Windows vendor software v1.2 keeps on disk. The keyboard never sees these files; they matter for importing the profiles and macros of a Windows installation. Everything here comes from the files themselves, compared with what the software sent to the keyboard on USB after each change.

## `%APPDATA%\Thermaltake\MEKA\`

INI files (ANSI). The GUI writes them; on "Apply" the software sends their content to the keyboard.

### `Config.config`

```
[CONFIG]
CURRENTPROFILE=0        active profile, 0 to 2
DEVICEPSTATS=1
Mode=1                  0 Normal, 1 Game
[LIGHTOPTION]
LIGHTSTYLE=1            0 Pulse, 1 to 4 level I to IV (Light Option page)
WASDLIGHT=1             per zone: 1 on, 0 off (WASD, arrows, keypad, logo)
DIRECTLIGHT=1
NUMBERLIGHT=1
Tt=0                    logo
```

The software sends `Mode` and the light settings to the keyboard each time it starts, and never reads them from the keyboard. It rewrites `CURRENTPROFILE` and `Mode` when the keyboard [notifies](../device-to-host/81/81.md) a change.

### `Profile0.prof` to `Profile2.prof`

One per profile.

```
[ALTERNATE_PROFILE]     Instant Shift keys
PREKEY=0                modifier for the next profile (p+1)
NEXTKEY=1               modifier for the profile after it (p+2); 255 none
[BUTTONCONFIG0]         T1; BUTTONCONFIG0 to 19 = T1 to T20
FunID=32                the device function in decimal: 0 none, 16 single key, 17 launch, 32 macro
Mappinkey=255           for T13 to T20, the standard key used (12 for F12, 11 for F11); 255 none
APP Path=C:\Windows\notepad.exe   (launch only)
SingleKey=30            (single key only) the key to type, vendor code in decimal (30 = 0x1e = X); 0 none
CurKey=0                (single key only)
AssKey=30               (single key only) same as SingleKey
[MACROG0]               the macro of T1; MACROG0 to 19
RepeatOption=1          1 once, 2 while held, 3 until pressed again, 4 multiple times
RepeatTimes=3           count for option 4 (left over otherwise)
MacroName=a
Macro=1401FF031400FF262F01FF012F00...    items, see below, padded with 00
MacroLength=7           number of items
[MAXTRIXSWAPKEY0]       single-key remaps; MAXTRIXSWAPKEY0 to 158, numbered by key, see below
ISCHANGED=1
CURKEY=121              the key, vendor code in decimal (121 = 0x79 = Scroll Lock)
ASSKEY=136              what it produces (136 = 0x88 = left Ctrl)
```

- Instant Shift codes seen: `0` Shift, `1` Ctrl. With Shift for the next profile and Ctrl for the one after, the keyboard received [`1a`](../host-to-device/1a/1a.md) with wValue `e0e1`.
- `RepeatOption` becomes, on the device: 1 and 4 → repeat `01` in the [macro-key map](key-map.md), with the count in the end event of the [macro](macro-data.md#macro); 2 → `02`; 3 → `03`.
- Predefined actions are already expanded into key items in the profile file.
- A single key with no key chosen (`SingleKey=0`) was written as `66 00 00`.
- **Remap sections** are numbered by the position of the key on the software's on-screen keyboard, a **US 104-key layout read row by row**: Esc 0, F1 to F12 1 to 12, Print Screen 13, Scroll Lock 14, Pause 15, `` ` `` 16, 1 to 0 17 to 26, ... Insert 30, ... Tab 37, Q 38, ... Space 94, ... keypad 0 102, keypad . 103 (seen: Pause, Insert, Tab, Q, Space, keypad 0, Scroll Lock). `CURKEY` alone identifies the key, so the number is not needed to read a file. The ISO key `<` (`64`) is not on that keyboard and cannot be remapped with the software. The software wrote each remap as the code of `ASSKEY` at the matrix position of `CURKEY` ([matrix](matrix.md)).

### `Macro Setting\<name>.ini`

The macro library of the editor:

```
[Setting]
MacroFilePath=...\Macro Setting\mix.ini
IgnoreDelay=0
DefaultDelay=1          "Insert Default Delay Time"
DefaultDelayValue=23    in ms; the device gets multiples of 50 ms
Count=12                number of 2-byte items in Macro
Macro=1101FF092101FF022100FF021100FF00EE02FF00EE00FF00...
```

## Macro items

Two bytes per item:

| Bytes | Meaning | Device event |
|---|---|---|
| `kk 01` / `kk 00` | press / release of the key with vendor code `kk` | `84 uu 01` / `84 uu 00`, `uu` from the table below |
| `ff nn` | delay of `nn` × 50 ms, one byte: at most 12750 ms | `87 00 nn` |
| `ee nn` | predefined action `nn` (`00` Page Up, `02` Copy), only in the library files | key events (expanded when the macro is assigned to a key) |

When it writes a profile, the software adds the end event `86 00 nn` and a trailing `87 00 01` to each macro ([macro data](macro-data.md#macro)).

## Vendor key codes

The files number keys with codes of their own. The table below comes from macros recorded with every key of an Italian keyboard (Windows layout Italian, Num Lock on and off), compared with the data the software then sent to the keyboard ([`12`](../host-to-device/12/12.md)): 102 codes.

**The software records the key Windows reports, as the key of a US keyboard**, not the physical key. On a US layout the two are the same. On another layout the keys with symbols are stored as the US key that gives the same Windows key, and the macro then types something else: on the Italian keyboard, pressing `\ ' ì è + ò à ù < , . -` gave a macro sending `31 2f 30 33 2e 35 34 38 66 85 37 2d` instead of `35 2d 2e 2f 30 33 34 32 64 36 37 38`, which typed `ùè+òì\à-.'` in Normal mode. More of the same:

- Code `00` means no key: the ISO key `<`, the Menu key and keypad 5 with Num Lock off were stored as `00`, written as usage `66` (Power), which typed nothing.
- The comma was stored as `3e`, written as `85` (the keypad comma of the international keyboards), which typed nothing.
- Keypad Enter was stored as Enter (`56` → `28`); with Num Lock off the keypad keys were stored as the navigation keys (Home, arrows, ...).

An importer that wants the result of the vendor software uses the table as it is; one that wants what was typed must translate by the layout of the computer the macro was recorded on.

| Vendor | HID | Key (US name of the usage) |
|---|---|---|
| `00` | `66` | (none) |
| `10` | `35` | ` (US) |
| `11` | `1e` | 1 |
| `12` | `14` | Q |
| `13` | `2b` | Tab |
| `14` | `04` | A |
| `15` | `29` | Esc |
| `16` | `1d` | Z |
| `18` | `3a` | F1 |
| `19` | `1f` | 2 |
| `1a` | `1a` | W |
| `1b` | `39` | Caps Lock |
| `1c` | `16` | S |
| `1e` | `1b` | X |
| `20` | `3b` | F2 |
| `21` | `20` | 3 |
| `22` | `08` | E |
| `23` | `3c` | F3 |
| `24` | `07` | D |
| `25` | `3d` | F4 |
| `26` | `06` | C |
| `28` | `22` | 5 |
| `29` | `21` | 4 |
| `2a` | `15` | R |
| `2b` | `17` | T |
| `2c` | `09` | F |
| `2d` | `0a` | G |
| `2e` | `19` | V |
| `2f` | `05` | B |
| `30` | `23` | 6 |
| `31` | `24` | 7 |
| `32` | `18` | U |
| `33` | `1c` | Y |
| `34` | `0d` | J |
| `35` | `0b` | H |
| `36` | `10` | M |
| `37` | `11` | N |
| `38` | `2e` | = (US) |
| `39` | `25` | 8 |
| `3a` | `0c` | I |
| `3b` | `30` | ] (US) |
| `3c` | `0e` | K |
| `3d` | `3f` | F6 |
| `3e` | `85` | Keypad , (international) |
| `40` | `41` | F8 |
| `41` | `26` | 9 |
| `42` | `12` | O |
| `43` | `40` | F7 |
| `44` | `0f` | L |
| `46` | `37` | . |
| `48` | `2d` | - (US) |
| `49` | `27` | 0 |
| `4a` | `13` | P |
| `4b` | `2f` | [ (US) |
| `4c` | `33` | ; (US) |
| `4d` | `34` | ' (US) |
| `4f` | `38` | / (US) |
| `50` | `42` | F9 |
| `51` | `43` | F10 |
| `53` | `2a` | Backspace |
| `54` | `31` | \ (US) |
| `55` | `3e` | F5 |
| `56` | `28` | Enter |
| `57` | `2c` | Space |
| `58` | `49` | Insert |
| `59` | `45` | F12 |
| `5a` | `60` | Keypad 8 |
| `5b` | `5d` | Keypad 5 |
| `5c` | `5a` | Keypad 2 |
| `5d` | `62` | Keypad 0 |
| `5e` | `54` | Keypad / |
| `5f` | `4f` | Right |
| `60` | `4c` | Delete |
| `61` | `44` | F11 |
| `62` | `5f` | Keypad 7 |
| `63` | `5c` | Keypad 4 |
| `64` | `59` | Keypad 1 |
| `66` | `53` | Num Lock |
| `67` | `51` | Down |
| `68` | `4b` | Page Up |
| `69` | `4e` | Page Down |
| `6a` | `61` | Keypad 9 |
| `6b` | `5e` | Keypad 6 |
| `6c` | `5b` | Keypad 3 |
| `6d` | `63` | Keypad . |
| `6e` | `55` | Keypad * |
| `6f` | `56` | Keypad - |
| `70` | `4a` | Home |
| `71` | `4d` | End |
| `72` | `57` | Keypad + |
| `75` | `52` | Up |
| `76` | `48` | Pause |
| `77` | `50` | Left |
| `79` | `47` | Scroll Lock |
| `7a` | `46` | Print Screen |
| `7d` | `e2` | Left Alt |
| `7f` | `e6` | Right Alt |
| `83` | `e1` | Left Shift |
| `86` | `e5` | Right Shift |
| `88` | `e0` | Left Ctrl |
| `8e` | `e4` | Right Ctrl |
| `9c` | `e3` | Left GUI |

Not seen: the right Windows key (absent from the Italian keyboard) and the keys the software does not record (T keys, media keys, profile and mode keys).

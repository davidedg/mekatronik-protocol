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
LIGHTSTYLE=1            backlight style chosen in the Light Option page
WASDLIGHT=1             per zone: 1 on, 0 off
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
SingleKey=30            (single key only) the key to type, vendor code in decimal (30 = 0x1e = X)
CurKey=0                (single key only)
[MACROG0]               the macro of T1; MACROG0 to 19
RepeatOption=1          1 once, 2 while held, 3 until pressed again, 4 multiple times
RepeatTimes=3           count for option 4 (left over otherwise)
MacroName=a
Macro=1401FF031400FF262F01FF012F00...    items, see below, padded with 00
MacroLength=7           number of items
[MAXTRIXSWAPKEY0]       single-key remaps; MAXTRIXSWAPKEY0 to 158
ISCHANGED=1
CURKEY=121              the key, vendor code in decimal (121 = 0x79 = Scroll Lock)
ASSKEY=136              what it produces (136 = 0x88 = left Ctrl)
```

- Instant Shift codes seen: `0` Shift, `1` Ctrl. With Shift for the next profile and Ctrl for the one after, the keyboard received [`1a`](../host-to-device/1a/1a.md) with wValue `e0e1`.
- `RepeatOption` becomes, on the device: 1 and 4 → repeat `01` in the [macro-key map](key-map.md), with the count in the end event of the [macro](macro-data.md#macro); 2 → `02`; 3 → `03`.
- Predefined actions are already expanded into key items in the profile file.

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
| `ff nn` | delay of `nn` × 50 ms | `87 00 nn` |
| `ee nn` | predefined action `nn` (`00` Page Up, `02` Copy), only in the library files | key events (expanded when the macro is assigned to a key) |

When it writes a profile, the software adds the end event `86 00 nn` and a trailing `87 00 01` to each macro ([macro data](macro-data.md#macro)).

## Vendor key codes

The files number keys with codes of their own. Pairs seen so far, by comparing the files with the data sent to the keyboard:

| Vendor | HID | Key |
|---|---|---|
| `11` | `1e` | 1 |
| `14` | `04` | A |
| `19` | `1f` | 2 |
| `1e` | `1b` | X |
| `21` | `20` | 3 |
| `26` | `06` | C |
| `2f` | `05` | B |
| `68` | `4b` | Page Up |
| `79` | `47` | Scroll Lock |
| `88` | `e0` | Left Ctrl |

The rest of the table is not known yet; it can be completed the same way, one key at a time.

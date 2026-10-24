# Thermaltake MEKA G-Unit (`060b:5253`): on-board profiles

The Tt eSPORTS MEKA G-Unit keeps **three profiles in its on-board memory** and runs the active one by itself, with or without software on the host: up to 20 macro keys per profile (T1 to T12, plus up to eight standard keys turned into T13 to T20), key remaps, and the Instant Shift System, which reaches the macro keys of the other profiles through a modifier. A key switches between Normal and Game mode; three keys select the profile. The host writes each profile as four tables with USB vendor control requests. This page is the map of what is known; the requests and the formats have pages of their own.

**Scope.** Everything here was seen on one MEKA G-Unit (Italian layout; lit keys: the WASD group, the arrows and the logo) with **firmware 1.35** ([`20`](../host-to-device/20/20.md): `01 35`; bcdDevice 1.35), driven by the Windows vendor software **v1.2**, unless it is marked *(inferred)*. The sources are the USB traffic between the vendor software and the keyboard, requests sent in tests, the behaviour of the keyboard, and the settings files of the vendor software.

**Contents:** [what the keyboard does on its own](#what-the-keyboard-does-on-its-own) · [USB functions](#usb-functions) · [transport](#transport) · [requests](#requests) · [sequences](#sequences) · [behaviour](#behaviour) · [implementation notes](#implementation-notes) · [open questions](#open-questions) · [formats](../formats/README.md)

## What the keyboard does on its own

| Feature | Stored in |
|---|---|
| Macro on a macro key: key presses, releases and delays (50 ms steps) | [macro index](../formats/macro-index.md) and [macro data](../formats/macro-data.md) |
| Repeat of a macro: once, N times, while the key is held, until the key is pressed again | N in the [macro data](../formats/macro-data.md), the mode in the [macro-key map](../formats/key-map.md) |
| A macro key that types one other key ("Single Key") | [macro index](../formats/macro-index.md) and [macro data](../formats/macro-data.md#single-key) |
| Remap of a standard key to another key, in Game mode | [key matrix](../formats/matrix.md) |
| A standard key used as macro key T13 to T20, in Game mode | [key matrix](../formats/matrix.md), plus the tables above |
| Instant Shift: the **left** Ctrl, Shift or Alt reaches the T keys of the other two profiles | [`1a`](../host-to-device/1a/1a.md) |
| The active profile and the work mode (Normal, Game) | [`14`](../host-to-device/14/14.md), [`02`](../host-to-device/02/02.md), or the keys |
| Backlight of the zones (WASD group, arrows, logo): 4 levels or pulse, each zone on or off; global, not per profile | [`31`](../host-to-device/31/31.md) |

Not on the keyboard: **launching a program**. The keyboard stores only "launch key N" and reports it ([`82`](../device-to-host/82/82.md)); the host software starts the program.

## USB functions

The keyboard is a composite device with three HID interfaces ([descriptors](../lsusb/060b/5253/0135)):

| Interface | Endpoint | Report | Used for |
|---|---|---|---|
| 0 | `0x81` IN, 8 bytes | boot keyboard | typing in Normal mode; [notifications](../device-to-host/81/81.md) |
| 1 | `0x82` IN, 4 bytes | consumer control, 2 usages of 16 bits | media and volume keys; [launch keys](../device-to-host/82/82.md) |
| 2 | `0x83` IN, 15 bytes | keyboard, modifiers and a bitmap of usages 0 to 6f | [typing in Game mode](../device-to-host/83/83.md) |

There is no vendor HID interface and no OUT endpoint. **The keyboard has no USB serial number** (`iSerial` 0; strings 3 to 5 not answered); the serial printed on its label appears nowhere on USB, so two keyboards can only be told apart by the port they are plugged into. The audio jacks of the keyboard are a separate USB device on the same cable (`04d9:2834` "HOLTEK USB PHONE"), which the vendor software does not use.

## Transport

- All configuration uses **vendor control requests to the device** on endpoint 0 ([list](../host-to-device/README.md)): `bmRequestType` `40` to write, `c0` to read; `bRequest` is the command; wValue and wIndex carry small parameters, mostly the profile in wIndex (1 to 3); tables travel in the data stage. No HID report is involved; requests to the device need no interface claim, so on Linux libusb should be able to send them while the kernel keeps the HID interfaces *(inferred, to be tried)*.
- The keyboard reports its own events (profile keys, mode key, backlight key, launch keys) as fake key codes in its ordinary HID reports ([`81`](../device-to-host/81/81.md), [`82`](../device-to-host/82/82.md)).

## Requests

| Request | Meaning |
|---|---|
| [`18`](../host-to-device/18/18.md) / [`19`](../host-to-device/19/19.md) | write / read the [key matrix](../formats/matrix.md) of a profile |
| [`10`](../host-to-device/10/10.md) / [`11`](../host-to-device/11/11.md) | write / read the [macro index](../formats/macro-index.md) of a profile |
| [`12`](../host-to-device/12/12.md) / [`13`](../host-to-device/13/13.md) | write / read the [macro data](../formats/macro-data.md) of a profile |
| [`16`](../host-to-device/16/16.md) / [`17`](../host-to-device/17/17.md) | write / read the [macro-key map](../formats/key-map.md) of a profile (read: profile 1 only) |
| [`1a`](../host-to-device/1a/1a.md) | Instant Shift keys of the active profile |
| [`14`](../host-to-device/14/14.md) / [`15`](../host-to-device/15/15.md) | select / read the active profile |
| [`02`](../host-to-device/02/02.md) | work mode: Normal or Game (not readable) |
| [`04`](../host-to-device/04/04.md) | status: backlight level, zone levels, active profile |
| [`05`](../host-to-device/05/05.md) | unknown, always `01 01` |
| [`20`](../host-to-device/20/20.md) | firmware information and version |
| [`31`](../host-to-device/31/31.md) | backlight of a zone |

From the keyboard: [notifications](../device-to-host/81/81.md) on the boot report, [launch keys](../device-to-host/82/82.md) on the consumer report.

## Sequences

### Write a profile

What the Windows software sends when "Apply" is pressed, for the profile on screen (`P` = 1 to 3); everything is rewritten, even what did not change:

```
> 40 18 0000 000P 00a0  <key matrix, 160 bytes>
> 40 10 0000 000P 0069  <macro index, 105 bytes>
> 40 12 OOOO 000P LLLL  <macro data, blocks of up to 4096>     only when the total is not 0; OOOO = offset of the block
> 40 16 0000 000P 003f  <macro-key map, 63 bytes>
> 40 1a hhll 0000 0000                                          Instant Shift keys (applies to the active profile)
> 40 14 000P 0000 0000                                          select profile P
```

- The requests go 0.06 to 1.1 s apart ([timing](../host-to-device/README.md#timing)). The profile written does not need to be the active one, except for `1a`, which goes to the active profile: the software had already selected the profile with `14` when its page was clicked, and selects it again at the end.
- The new macros work as soon as the requests are done, and the profile survives a power cycle and a move to another computer: nothing else is sent to save it.
- Captured examples: [sniffs/5253-write-profile.txt](../sniffs/5253-write-profile.txt).

### Empty a profile

The "R" (reset) button of the Windows software writes the empty profile ([formats](../formats/README.md#an-empty-profile)) for the profile on screen, with the same sequence (no `12`). Its warning speaks of resetting the device, but only that profile is written, and nothing else is sent.

### Read a profile

The vendor software reads nothing back. The reads work, with three traps:

```
< c0 11 0000 000P 0069        macro index; all ff = never written: stop here
< c0 19 0000 000P 00a0        key matrix
< c0 13 0000 000P LLLL        macro data, only if the total LLLL (index bytes 3-4) is 1 to 4096
< c0 17 0000 0001 003f        macro-key map: 40 bytes, valid for profile 1 only
```

There is no way to read the Instant Shift keys or the macro-key map of profiles 2 and 3: keep them on the host.

### Start of the vendor software

At start the Windows software sends the saved backlight settings ([`31`](../host-to-device/31/31.md) ×4) and the saved work mode ([`02`](../host-to-device/02/02.md)), and reads nothing: if the keyboard was switched while the software was closed, the software imposes its own state. It then follows the [notifications](../device-to-host/81/81.md).

## Behaviour

- **Normal and Game mode.** In Game mode the key matrix of the active profile is used (remaps and T13 to T20), the keyboard types on its NKRO interface, and the Windows key sends nothing. In Normal mode it types its default layout on the boot interface. T1 to T12 and Instant Shift work in both modes.
- **No indicator.** The keyboard has no light for the profile or the mode; only the [notifications](../device-to-host/81/81.md) (and [`15`](../host-to-device/15/15.md) for the profile) tell them.
- **Profiles that were never written are dangerous.** Their macro index is all `ff` (as profiles 2 and 3 of the test unit came from the factory):
  - pressing a T key there plays the erased memory as a macro: about 26 000 empty NKRO reports at 1 ms (37 s), during which the other macros do not run;
  - reading their macro data ([`13`](../host-to-device/13/13.md)) answers old memory content, and the next `13` made the firmware hang until the keyboard was unplugged.

  An Instant Shift into such a profile, even by a remapped key, triggers the first problem.
- **Instant Shift** follows the code a key produces, not the physical key, and the keyboard notifies the temporary profile ([`81`](../device-to-host/81/81.md)). The modifier reaches the host too, but is left out of the reports of the shifted key ([`1a`](../host-to-device/1a/1a.md)).
- **Host changes are silent**: [`14`](../host-to-device/14/14.md) and [`02`](../host-to-device/02/02.md) produce no notification; the keys on the keyboard do. After `14` the keyboard sends empty key reports (all keys released).
- **The work mode cannot be read**; the active profile can ([`15`](../host-to-device/15/15.md), [`04`](../host-to-device/04/04.md) byte 7). After a power cycle the keyboard was once seen in Normal mode ([`02`](../host-to-device/02/02.md)).
- **Usages above `6f` cannot be typed in Game mode**, and one in a macro ends the macro there ([macro data](../formats/macro-data.md#macro)).

## Implementation notes

**Session**

- Check [`20`](../host-to-device/20/20.md) first: bytes 0 to 5 `52 53 30 03 01 35` are the only firmware these pages describe.
- Send only the requests and values on these pages. Unknown requests are not worth trying: an unexpected read hung this firmware.
- Pace writes as the vendor software does ([timing](../host-to-device/README.md#timing)) until shorter delays are proven safe.

**Writing**

- Write all three profiles the first time, the unused ones as [empty profiles](../formats/README.md#an-empty-profile), so that no profile is left with an index of `ff`.
- Write a profile whole, in the vendor order ([sequence](#write-a-profile)); build the macro index and the data together (offsets and total must match); skip `12` when the total is 0.
- Select the profile with [`14`](../host-to-device/14/14.md) before [`1a`](../host-to-device/1a/1a.md), then select the profile that should be active.
- Set the work mode explicitly ([`02`](../host-to-device/02/02.md)) if it matters: it cannot be read.
- Keep macro and remap usages within `04` to `6f` and the modifiers `e0` to `e7`: the NKRO report of Game mode has no room for others.
- Delays are multiples of 50 ms; repeat counts go in the end event, repeat modes in the macro-key map; a T13 to T20 key needs its code `d8`+k in the matrix **and** its entries in the index and the map.
- Keep the repeat byte of every key in the map, also for a cleared key, at a defined value (the vendor software leaves stale values; `00` or `01` both work with function `00`).

**Reading**

- Read the macro index first; read the macro data only when the index is valid and its total is 1 to 4096, with wValue 0 and wLength = the total: after a `13` on a profile with more data, the next `13` hangs the firmware ([`13`](../host-to-device/13/13.md)).
- Do not trust [`17`](../host-to-device/17/17.md) for profiles 2 and 3.

**Host side**

- Launch keys need a host program that listens to the consumer report ([`82`](../device-to-host/82/82.md)) and starts the program.
- Without the vendor software, the notification codes reach the operating system as key codes; on Linux some of them become media and power keys ([`81`](../device-to-host/81/81.md)). A host program should catch them through hidraw and neutralise them.

## Open questions

- Whether Game mode also blocks the right Windows key.
- What the firmware does with the id field of the macro-key map, and with the 13 unidentified vendor codes of the matrix.
- The largest profile the memory takes (5724 bytes worked). (The macro data of a profile with more than 4096 bytes cannot be read back safely: [`13`](../host-to-device/13/13.md).)
- After a [`13`](../host-to-device/13/13.md) on a profile with more than 4096 bytes: which requests besides `15` still work, and whether a `13` on another profile hangs too.
- Whether the firmware needs the vendor's pauses between requests.
- The work mode after power-on (Normal was seen once); whether the backlight setting survives a power cycle; what zone 2 (keypad) lights on other units.
- The unknown bytes of [`04`](../host-to-device/04/04.md), [`05`](../host-to-device/05/05.md) and [`20`](../host-to-device/20/20.md).

## More

- [Formats](../formats/README.md): [key matrix](../formats/matrix.md) · [macro index](../formats/macro-index.md) · [macro data](../formats/macro-data.md) · [macro-key map](../formats/key-map.md) · [vendor files](../formats/vendor-files.md)
- [All requests](../host-to-device/README.md) · [USB descriptors](../lsusb/060b/5253/0135) · [captured sequences](../sniffs/5253-write-profile.txt)

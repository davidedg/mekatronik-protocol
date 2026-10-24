# Data formats

The MEKA G-Unit keeps, for each of its three profiles, four tables in its on-board memory. The host writes each table whole with one vendor request and can read three of them back. How they are written, and in which order, is in [devices/meka-g-unit.md](../devices/meka-g-unit.md).

Everything in these pages was seen on a MEKA G-Unit with firmware 1.35 (request [`20`](../host-to-device/20/20.md)), unless it is marked *(inferred)*. Byte values are hexadecimal. **Multi-byte integers inside the tables are big-endian**; the fields of the USB setup packet (wValue, wIndex, wLength) are little-endian as always.

| Table | Size | Written with | Read with | Page |
|---|---|---|---|---|
| Key matrix | 160 | [`18`](../host-to-device/18/18.md) | [`19`](../host-to-device/19/19.md) | [matrix](matrix.md) |
| Macro index | 105 | [`10`](../host-to-device/10/10.md) | [`11`](../host-to-device/11/11.md) | [macro index](macro-index.md) |
| Macro data | 0 to 16326 tried (the offsets allow 64 KiB) | [`12`](../host-to-device/12/12.md) | [`13`](../host-to-device/13/13.md) | [macro data](macro-data.md) |
| Macro-key map | 63 | [`16`](../host-to-device/16/16.md) | [`17`](../host-to-device/17/17.md) (40 bytes, profile 1 only) | [macro-key map](key-map.md) |

Not in a table: the Instant Shift keys of a profile ([`1a`](../host-to-device/1a/1a.md)), the work mode ([`02`](../host-to-device/02/02.md)), the backlight ([`31`](../host-to-device/31/31.md)).

## The 20 macro keys

Three tables have one entry per **macro key** of the profile, always in this order:

| Entry | Key |
|---|---|
| 0 to 11 | T1 to T12, the two columns of dedicated keys on the left |
| 12 to 19 | T13 to T20: standard keys turned into macro keys (any key of the main area, chosen per profile) |

A standard key becomes T13+k by having the code `d8+k` in its position of the [matrix](matrix.md), and its own entry 12+k in the [macro index](macro-index.md) and the [macro-key map](key-map.md).

## Key codes

Keys are named by their **HID usage** (keyboard page `07`): `04` A, `1e` 1, `e0` left Ctrl, and so on. The firmware also uses codes of its own in the matrix: `cc` to `d7` for T1 to T12, `d8` to `df` for T13 to T20, `f1` to `f3` for the profile keys, and a few more for the other dedicated keys ([matrix](matrix.md)).

The Windows vendor software stores keys in its files with a numbering of its own ([vendor files](vendor-files.md)); the keyboard never sees it.

## An empty profile

The factory state of a profile that was never written is all `ff` in the macro index (seen on profiles 2 and 3 of the test unit). **A profile in this state is dangerous**: its T keys play the erased memory as a macro, and reading its macro data answers old memory content, after which the next read hangs the firmware ([behaviour](../devices/meka-g-unit.md#behaviour)). The vendor software's "Reset" writes this empty profile instead:

```
18  the default matrix
10  14 00 69 00 00 | 00 00 00 00 00 ×20
16  14 | 00 00 01 00 02 00 03 00 04 00 05 00 06 00 07 00 08 00 09 00 0a 00 0b 00 | 00 00 ×8 | 00 ×22
1a  wValue 0000
    (no 12: there is no macro data)
```

- [Vendor files](vendor-files.md): what the Windows software keeps on disk, for importing its profiles and macros.

# Macro index (requests `10` and `11`)

105 bytes (`0x69`): what each of the 20 macro keys of a profile does, and where its data is in the [macro data](macro-data.md) of the profile. Written with [`10`](../host-to-device/10/10.md), read back with [`11`](../host-to-device/11/11.md).

```
offset  size  field
0       1     14: number of entries (20)
1       2     00 69: size of the table (105), big-endian
3       2     total length of the macro data of the profile, big-endian
5       5×20  one entry per macro key: T1 to T12, then T13 to T20
```

Each entry:

```
offset  size  field
0       1     function
1       2     offset of the key's data in the macro data, big-endian
3       2     length of the key's data, big-endian
```

| Function | Meaning | Data |
|---|---|---|
| `00` | nothing assigned: a T key types nothing | none; offset and length `00 00` |
| `20` | macro | the events of the macro ([macro data](macro-data.md#macro)) |
| `11` | launch a program on the host | 2 bytes ([macro data](macro-data.md#launch)) |
| `10` | single key: the key types one other key | the HID usage, then 2 more bytes from Windows ([macro data](macro-data.md#single-key)) |

The data of the keys lie one after the other, in entry order (T1 first, T13 to T20 last), with no gaps: the offset of a key is the sum of the lengths before it, and the total in bytes 3 to 4 is the sum of all lengths. The vendor software rebuilds the whole data block at every write, so the offsets move when a key before them changes. The keyboard does not need this layout: **two entries with the same offset and length share their data** (T6 pointing at the macro of T4 played it).

## Examples

Profile 1 with a macro on T1 (27 bytes), a macro on T2 (30 bytes), a launch on T3 and a macro on T13 (F12):

```
14 00 69 00 56 | 20 00 00 00 1b | 20 00 1b 00 1e | 11 00 39 00 02 | 00 00 00 00 00 ×9 |
                 20 00 3b 00 1b | 00 00 00 00 00 ×7
```

Profile 3 with a single key on T8 (3 bytes) and a launch on T13 (F11):

```
14 00 69 00 05 | 00 00 00 00 00 ×7 | 10 00 00 00 03 | 00 00 00 00 00 ×4 | 11 00 03 00 02 | 00 00 00 00 00 ×7
```

The empty profile written by the vendor software's Reset: `14 00 69 00 00` and 100 zero bytes.

A profile that was never written reads back as 105 bytes of `ff`. **Do not leave a profile in that state** ([behaviour](../devices/meka-g-unit.md#behaviour)).

## Notes

- Clearing a key ("Default" in the vendor software) writes its entry as `00 00 00 00 00` and drops its data; the [macro-key map](key-map.md) keeps whatever it had for the key, so the function in this table is what counts.
- For T13 to T20 the entry only says what the key does; which standard key it is comes from the [matrix](matrix.md) (code `d8`+k).
- The offsets are 16 bits, so the data of a profile is at most 64 KiB *(inferred)*; the memory is 64 KB for the three profiles according to the vendor, and how much of it a profile can use has not been tested.

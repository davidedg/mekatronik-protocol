# Macro-key map (requests `16` and `17`)

63 bytes (`0x3f`): for each of the 20 macro keys of a profile, its identifier and its **repeat mode**. Written with [`16`](../host-to-device/16/16.md); [`17`](../host-to-device/17/17.md) reads back only the 40 bytes of the pairs, and only for profile 1.

```
offset  size  field
0       1     14: number of entries (20)
1       2×20  one pair (id, repeat) per macro key: T1 to T12, then T13 to T20
41      22    00
```

| Field | Value |
|---|---|
| id, T1 to T12 | `00` to `0b`, the number of the key counted from 0, always written whatever the key does |
| id, T13 to T20 | `d8`+k when a standard key is T13+k (the same code as in the [matrix](matrix.md)); `00` when the entry is not in use |
| repeat | `01` once, or the number of times in the macro's end event · `02` **while the key is held** · `03` **until the key is pressed again** (toggle) · `00` never set |

The repeat mode applies to a macro ([macro data](macro-data.md#macro)). The count of "Multiple times" is not here but in the end event `86 00 nn` of the macro, with repeat `01` here.

```
T1 macro once, T2 macro 3 times, T3 launch, T4 macro while held, T5 macro until pressed again,
T6 cleared (the stale 01 stays), T7 macro once, T13 (F12) macro once:
14 | 00 01 01 01 02 01 03 02 04 03 05 01 06 01 07 00 08 00 09 00 0a 00 0b 00 | d8 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 | 00 ×22

empty profile:
14 | 00 00 01 00 02 00 03 00 04 00 05 00 06 00 07 00 08 00 09 00 0a 00 0b 00 | 00 00 ×8 | 00 ×22
```

## Notes

- The Windows software writes repeat `01` for a key with a launch (T3 above) and `00` for a single key, and keeps the old repeat value of a key whose function was set back to default (it wrote `05 01` for T6 after clearing it): the [macro index](macro-index.md) decides what a key does.
- What the firmware does with the id field is not known: no value other than the ones above was tried.

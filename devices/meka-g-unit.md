# Thermaltake MEKA G-Unit (`060b:5253`): on-board profiles

The MEKA G-Unit keeps **three profiles in its on-board memory** (64 KB) and runs the active one by itself: up to 20 macro keys per profile (T1 to T12, plus 8 standard keys chosen as T13 to T20), single-key remaps, and the Instant Shift System, which reaches the macro keys of the other profiles through a modifier. A dedicated key switches between Normal and Game mode (Game mode disables the Windows key); three keys select the profile.

**Scope.** To be filled in: firmware version, vendor software version used as reference.

**Contents:** [what the keyboard does on its own](#what-the-keyboard-does-on-its-own) · [transport](#transport) · [commands](#commands) · [sequences](#sequences) · [open questions](#open-questions)

## What the keyboard does on its own

| Feature | Stored in |
|---|---|
| Macro on a T key: key presses, releases and delays | TBD |
| Repeat mode of a macro: once, N times, while held, until pressed again | TBD |
| Remap of a standard key to another key | TBD |
| Instant Shift System modifier | TBD |
| Backlight: 3 areas and logo, 4 levels, pulse (Illuminated edition) | TBD (not per profile) |

Not on the keyboard *(inferred)*: launching programs, which the vendor software does on the host.

## Transport

TBD.

## Commands

TBD.

## Sequences

TBD.

## Open questions

- Transport: control transfers or interrupt OUT, and on which interface.
- Can the host read back profiles and macros, the active profile and the mode?
- Memory layout of a profile and of a macro.

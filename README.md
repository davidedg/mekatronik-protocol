# The Thermaltake MEKA G-Unit USB protocol

Documentation of the USB protocol that the vendor software uses to program the
**Thermaltake Tt eSPORTS MEKA G-Unit** mechanical keyboard (`060b:5253`): on-board macros,
key remaps, the three profiles, Normal/Game mode, the Instant Shift System and the backlight.

The aim is to make the on-board features usable without the vendor software, which supports
Windows up to 8 only. The layout follows [davidedg/corsair-protocol](https://github.com/davidedg/corsair-protocol):

- [`devices/`](devices/README.md): the overall picture of a device, with links to the pages of its commands and formats.
- `host-to-device/`, `device-to-host/`: one page per command or report, in folders by bytes.
- `formats/`: data formats (on-board memory layout, macro encoding, vendor software files).
- `lsusb/`: USB descriptors.
- `sniffs/`: annotated captures.

This work was written with AI assistance.

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).

## What we know so far

Seen on a MEKA G-Unit with firmware 1.35 and the Windows vendor software v1.2, from the USB traffic, the behaviour of the keyboard and the settings files of the vendor software. Start from [devices/meka-g-unit.md](devices/meka-g-unit.md).

- The keyboard is configured with **USB vendor control requests to the device** (`bmRequestType` `40` / `c0`), one `bRequest` per command: [host-to-device/](host-to-device/README.md). No HID report is involved.
- Each of the three profiles is four tables, written whole and partly readable: the [key matrix](formats/matrix.md), the [macro index](formats/macro-index.md), the [macro data](formats/macro-data.md) and the [macro-key map](formats/key-map.md). The Instant Shift keys, the active profile, the Normal/Game mode and the backlight have requests of their own.
- The keyboard reports its own events (profile keys, mode key, backlight key, launch keys) as fake codes in its ordinary HID reports: [device-to-host/](device-to-host/81/81.md).
- A profile that was never written is a trap: see [behaviour](devices/meka-g-unit.md#behaviour).

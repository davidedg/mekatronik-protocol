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

Work in progress: nothing in this repo is confirmed by a capture yet.

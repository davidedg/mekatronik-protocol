# Host to device: vendor requests

The MEKA G-Unit is configured with **USB vendor control requests to the device** on endpoint 0: `bmRequestType` `40` for a request that sends data or none, `c0` for a request that reads. The HID interfaces are not involved: no HID report carries configuration, so a host should be able to send the requests (with libusb, for example) while the operating system keeps using the keyboard *(inferred, to be tried on Linux)*.

Pages are named by `bRequest`. The setup fields are given as the 16-bit values of the USB setup packet. Everything was captured on a keyboard with firmware 1.35: the write requests as the Windows vendor software v1.2 sends them, the read requests as the keyboard answered them in tests.

| bRequest | Dir | wValue | wIndex | Data | Meaning | |
|---|---|---|---|---|---|---|
| `02` | out | mode | 0 | none | work mode: Normal or Game | [02](02/02.md) |
| `04` | in | 0 | 0 | 8 | status: backlight level, active profile | [04](04/04.md) |
| `05` | in | 0 | 0 | 2 | always `01 01` | [05](05/05.md) |
| `10` | out | 0 | profile | 105 | write the macro index | [10](10/10.md) |
| `11` | in | 0 | profile | 105 | read the macro index | [11](11/11.md) |
| `12` | out | 0 | profile | length | write the macro data | [12](12/12.md) |
| `13` | in | 0 | profile | length | read the macro data | [13](13/13.md) |
| `14` | out | profile | 0 | none | select the active profile | [14](14/14.md) |
| `15` | in | 0 | 0 | 1 | read the active profile | [15](15/15.md) |
| `16` | out | 0 | profile | 63 | write the macro-key map | [16](16/16.md) |
| `17` | in | 0 | profile | 40 | read the macro-key map (profile 1 only) | [17](17/17.md) |
| `18` | out | 0 | profile | 160 | write the key matrix | [18](18/18.md) |
| `19` | in | 0 | profile | 160 | read the key matrix | [19](19/19.md) |
| `1a` | out | keys | 0 | none | Instant Shift keys of the active profile | [1a](1a/1a.md) |
| `20` | in | 0 | 0 | 16 | firmware information and version | [20](20/20.md) |
| `31` | out | level | zone | none | backlight of a zone | [31](31/31.md) |

No other request is documented. Do not try unknown requests: an unexpected read already hung this firmware ([`13`](13/13.md)).

## Timing

Gaps seen between the requests of the Windows software:

| After | Gap to the next request |
|---|---|
| `18` | 0.5 to 1.1 s |
| `10` | 0.5 s before `12`; 1.0 s before `16` when there is no `12` |
| `12` (up to 212 bytes) | 1.0 s |
| `16` | 0.6 s |
| `1a` | 0.06 s, then `14` |
| `31` | 0.25 to 0.33 s between zones; 0.5 to 0.65 s before the `02` at start |

Whether the firmware needs these pauses has not been tested; reads sent 4 ms apart were answered normally.

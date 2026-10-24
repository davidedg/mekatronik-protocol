# Host to device: vendor requests

The MEKA G-Unit is configured with **USB vendor control requests to the device** on endpoint 0: `bmRequestType` `40` for a request that sends data or none, `c0` for a request that reads. The HID interfaces are not involved: no HID report carries configuration, so a host can send the requests while the operating system keeps using the keyboard (tried on Linux with libusb: no interface claimed, the kernel's HID driver stays bound).

Pages are named by `bRequest`. The setup fields are given as the 16-bit values of the USB setup packet. Everything was captured on a keyboard with firmware 1.35: the write requests as the Windows vendor software v1.2 sends them, the read requests as the keyboard answered them in tests.

| bRequest | Dir | wValue | wIndex | Data | Meaning | |
|---|---|---|---|---|---|---|
| `02` | out | mode | 0 | none | work mode: Normal or Game | [02](02/02.md) |
| `04` | in | 0 | 0 | 8 | status: backlight level, zone levels, active profile | [04](04/04.md) |
| `05` | in | 0 | 0 | 2 | always `01 01` | [05](05/05.md) |
| `10` | out | 0 | profile | 105 | write the macro index | [10](10/10.md) |
| `11` | in | 0 | profile | 105 | read the macro index | [11](11/11.md) |
| `12` | out | offset | profile | up to 4096 | write a block of the macro data | [12](12/12.md) |
| `13` | in | 0 | profile | up to 4096 | read the macro data (profiles with at most 4096 bytes only) | [13](13/13.md) |
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

| After | Gap to the next request (from setup to setup) |
|---|---|
| `18` | 0.5 to 1.1 s |
| `10` | 0.5 s before `12`; 1.0 s before `16` when there is no `12` |
| `12` | 1.0 s for up to 212 bytes; 6.0 s for 4096 bytes, 3.4 s for 1628 bytes |
| `16` | 0.6 s |
| `1a` | 0.06 s, then `14` |
| `31` | 0.25 to 0.33 s between zones; 0.5 to 0.65 s before the `02` at start |

Part of each gap is the keyboard itself: a write of a large block completes only when it has stored the data. Time from setup to status stage, seen in the Windows captures: `18` (160 bytes) 0.21 s, `16` (63 bytes) 0.17 s, `12` 1.54 s for 4096 bytes and 0.83 s for 1628 bytes, `10` and the requests without data a few milliseconds. Under Linux: `18`, `10`, `16` and a `12` of 212 bytes 2 to 3 ms, `12` 0.43 s for 1494 bytes, 1.5 s for 4096 bytes, 0.65 s for 1628 bytes. Use timeouts of several seconds.

**After a table write the keyboard stays busy for a while, and refuses requests with a stall meanwhile** (tests under Linux): about 50 ms after `18`, `10` and `16`, 100 to 260 ms after a `12` block (212 to 4096 bytes). A `10` sent at once after `18` was refused; the keyboard answered again by itself some seconds later, the slot unchanged. A refused request did no harm. Waiting 60 ms after `18`, `10`, `16` and 150 ms after `12`, then sending [`15`](15/15.md) until it is answered, wrote every profile right: 212 bytes in 0.5 s, 5724 bytes in 2.9 s (the Windows pauses take 2.2 s and about 10 s), 16326 bytes in about 7 s.

After `14` and `1a`, a `15` sent at once was not answered for 1 to 5 s; with a pause of 0.05 s, as the Windows software, the next request was always answered. Reads sent 4 ms apart were answered normally.

**Hosts that lose transfers.** With the keyboard on a Raspberry Pi 3 (Linux), up to one control transfer in six failed at once with a protocol error (EPROTO within 0.1 ms, no data moved), and interrupt reports were lost too; the same keyboard under VMware on a PC never showed it. Sent again, the requests worked, with one exception: a [`12`](12/12.md) block sent again 150 ms after such an error never completed, and the keyboard hung. A hung keyboard comes back only when it loses power ([`13`](13/13.md)). In a later run each write that failed at once was sent again only after a [`15`](15/15.md) had been answered: ten writes failed at once (three of them `12` blocks), `15` answered within 1 ms every time, and every write sent again completed.

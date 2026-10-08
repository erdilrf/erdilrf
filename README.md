# ERDI — 905 nm Laser Rangefinder Modules

**We publish the gaps in our own datasheets.**

Integration is where laser rangefinder projects actually die — not in the optics, but in the
serial protocol. A manual that is silent about an I²C address, or that contradicts itself about
a default baud rate, costs an engineer a week.

So we do the opposite of what the category normally does: **we document the contradictions and
say exactly what we did about each one.** Every constant in our drivers is cited to the manual
it came from, and every place the manual is silent or wrong is listed explicitly.

---

## Why this exists

A 905 nm rangefinder module is easy to buy and hard to integrate. The failure mode is almost
always the same: the datasheet looks complete, so the engineer writes a parser, and then a
specific field misbehaves in a specific way that the manual never mentioned.

Here are three real examples from our own documentation set. All of them are checkable against
the printed manuals.

### 1. A printed example frame that fails its own checksum function

`LRF1500H`'s manual prints the example frame:

```
5C 02 11 03 EC
```

Running the checksum function **that the same manual documents** over `5C 02 11 03` yields:

```
5C 02 11 03  →  E9
```

`EC` is the correct value for the **other models' 4-byte frame**, so this appears to be a
copy-paste from a sibling manual. Our drivers implement the documented function, and a test
pins the discrepancy so it cannot silently drift.

### 2. Two different default baud rates in one manual

`SPD1200D03` states **115200 bps** in the protocol section and **15200 / 9600 bps** in the
technical table — and then adds:

> "Do not select a host baud rate until the delivered unit configuration is confirmed."

There is no correct default to hardcode. Our drivers therefore **require** an explicit baud rate
for this model instead of picking a plausible one.

### 3. `0xFFFF` is not `16777215`

`LRF1500H`'s out-of-range value is stated twice, inconsistently:

| Section | Stated value | Decimal |
|---|---|---|
| 5.2 | `16777215` | 16,777,215 |
| 6 | `"0xFFFF (16777215)"` | `0xFFFF` = **65,535** |

We use `16777215`, because the field is 3 bytes wide and that is the only value consistent with
the field width.

---

## The other 11

These are the remaining entries in our gap list:

4. `LRF1200A1` / `LRF3000A1` continuous ranging is **protocol-reserved** — the manual documents request `0x89` and reply `0x88` but does not authorise a response parser
5. `LRF50VB`'s printed serial read request **fails its own checksum** on both length bytes
6. `SPD1200N0 / N2 / N4` publish **no transmit checksum formula**
7. Self-test `ErrCode` byte position differs between transcriptions (`D3` for LR series, `D4` for the SPD1200D03 8-byte form)
8. `LRF300H` / `LRF600H` / `LRF1500H` publish **no output-frequency divider formula**
9. **Five models publish no I²C address** (only `LRF200H` = `0x33`, VB series = `0x52`)
10. Family-A **stop bits and parity are unspecified** — only "Data bits: 8" is stated
11. `LRF10VB` / `SPD1200S2G` have **no published protocol at all**
12. `LRF50VB` printed read-request checksum is incorrect
13. *(duplicate of 1)*
14. **These drivers have not been tested on real hardware** — codec validation is against golden frames printed in the manuals, the transport layer uses a fake serial port, and the ROS 2 package is statically checked but has never run in a real ROS 2 environment

---

## Our rule

> **Where a manual is silent or contradicts itself, the drivers refuse to guess — and say so.**

Concretely, this means:

- No averaging over ambiguity
- No "plausible default" where the vendor did not state one
- A raised exception and a failing test instead of a silent assumption
- Every constant traceable to the document it came from

---

## What is in this organisation

| Repository | What it is |
|---|---|
| [`erdi-lrf-drivers`](https://github.com/erdilrf/erdi-lrf-drivers) | Python, Arduino and ROS 2 drivers for both published protocol families |
| [`erdilrf`](https://github.com/erdilrf/erdilrf) | This index |

---

## Products

**905 nm laser rangefinder modules**, from short-range (10 m class) through long-range
(3000 m class), with UART / RS-485 / I²C interfaces.

- Website: <https://erdilrf.com>
- Contact: <tommy@erdimail.com>

> **Note on our own mail setup, since it is the same kind of thing this repository is about:**
> `erdilrf.com` publishes `v=spf1 -all` and has no MX record. That is deliberate — the website
> domain is configured to be unable to send or receive mail, so it cannot be spoofed. Business
> mail runs on a separate domain. If you have ever tried to reply to an address at the website
> domain and had it bounce, that is why.

---

## Contributing

If you have found a contradiction in one of our manuals, or a case where a documented
behaviour and the physically sensible behaviour disagree, please open an issue. We would
rather correct a document than have an integrator lose a week to it.

---

*Every technical claim on this page is reproducible from the printed manuals. Where we are
uncertain, we say so — see item 14.*

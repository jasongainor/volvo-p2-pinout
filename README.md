# 2005 Volvo S60R Wiring Pinout Reference

A community-verified pinout reference for the 2005 Volvo S60R (B5254T4 engine,
AW55-50SN automatic, AWD, Four-C suspension, Bi-Xenon HID, ECPS), extracted
from the official Volvo service publication **TP 3976201** (S60/S60R/S80 2005
Wiring Diagrams).

Intended for harness repair, restoration, ECU swaps, and learning the car's
electrical architecture.

## Why this exists

The S60R is over 20 years old. The official service documentation is out of
print, no longer sold by Volvo, and only circulates through enthusiast
channels in PDF form. The cars themselves are increasingly in the hands of
DIY owners, indie shops, and project builders rather than dealerships, and
the people working on them are doing harness repairs, engine swaps, manual
swaps, and chassis transplants where you really need to know what every wire
does.

This repository exists to **preserve and make searchable** the wiring
information that's most useful to that community:

* As a **searchable, sortable reference** that beats flipping through a 135-
  page PDF when you have a wire in your hand at 1 AM.
* As a **harness-labeling source** for anyone unwrapping their loom to
  re-tape, repair, or recolor.
* As **machine-readable data** that future tools (diagnostic apps,
  wire-tracing utilities, conversion projects) can build on.
* As **insurance against link-rot**: the original PDF has been quietly
  disappearing from forum attachments and file hosts for years.

The CSV contains only extracted facts (pin → wire color → destination →
function), not reproductions of the original schematics or prose. The
methodology, color conventions, and known issues are documented below so
anyone can verify the data against their own copy of the manual or their
own harness.

## Companion video

For a worked example of using this kind of pinout data in practice — manual
swap wiring, harness labeling, and rerouting — see
[this S60R manual swap walkthrough](https://youtu.be/T06YKNhRZaI?t=1200)
(timestamp jumps to the wiring section).

## Manual swap quick reference (M66 / AW55-50SN → MAN)

If you're doing a 6-speed manual swap on an S60R (AUTO from factory), the
**only ECM wiring you need to add** is the clutch position sensor circuit.
The S60R came with AUTO transmission, so junction `53/324` and the wires it
serves don't physically exist in the harness — you need to fabricate them.

What you need to add:

| ECM pin | Wire color | Run to | Notes |
|---|---|---|---|
| `B:4` | BN | Clutch position sensor `7/123` pin 4 | Twisted pair with B:29 in the original AUTO harness layout — for MAN, run as a single wire from B:4 to the sensor |
| `B:15` | SB | Clutch position sensor `7/123` pin 2 | Signal return |

Both wires run from the ECM (in the engine bay, passenger-side firewall on
LHD cars) into the cabin to where you mount the clutch pedal position
sensor.

What you do **not** need to do:

* `B:29` (OR-W in the AUTO factory wiring) stays unused on MAN — leave it.
* `D:23` (BL-W) reverse lockout solenoid feed already exists in the harness
  and connects to the same pin on either trans (`3/156A:3` AUTO or
  `8/134:3` MAN/M66). No repinning needed at the CEM, just change the connector.
* Reverse lights — the AUTO trans sends reverse state over CAN. For MAN
  swaps, the `3/10` reverse switch on the transmission feeds CEM `C:34`
  (Y-GR wire) which the CEM uses to trigger the reverse lights through the
  REM. You'll need to add that switch wire from the trans tunnel up to the
  CEM if you want reverse lights working through the factory CAN path. I've previously just taking the output of `8/134`, used it as a trigger for a manually added relay instead of mucking with the CEM.
* **TCM removal and HISPEED CAN** — this is the part that bites people. The
  HISPEED CAN backbone runs *through* the TCM in series, not as a stub. The
  AUTO-trans branch uses TCM `B:2` (W, CAN HI) and `B:14` (GN, CAN LO) on
  the "upstream" side and the same pair continues to the ECM. If you pull
  the TCM and just leave the wires dead-ended, the ECM loses CAN
  communication with the CEM and the car won't run properly (immobilizer
  handshake, throttle authorization, etc. all go over CAN). 

  Two ways to handle it:

  1. **Bypass jumper** (most common DIY approach): cut the TCM out of the
     loop and splice the CAN HI wires together (the W going to and from
     TCM `B:2`) and the CAN LO wires together (the GN going to and from
     TCM `B:14`). The bus then passes through where the TCM used to be.
     Keep the twisted pair intact when splicing.
  2. **Move ECM-side wires** (cleaner):
     pull the CAN pair that fed the TCM and re-terminate it directly at
     the ECM CAN pins (`B:1` HI and `B:13` LO), removing the TCM from the
     bus topology entirely. Same electrical result, less wire left in the
     harness. You don't need to cut any wires, just move the pins. Also make a cap for the empty spot in the box. Don't leave bare connectors exposed.

  The MAN-trans branch shown in the schematics (TCM `B:1` / `B:13` dashed
  lines) is a different physical pair that doesn't exist in the S60R AUTO
  harness — ignore it; it's there for cars that came from the factory with
  a manual transmission.

Cross-reference these against rows in
`volvo_s60r_2005_ALL_PINOUT_verified.csv` where the Notes column mentions
"MAN swap" — there's additional context there.

## Parking Assistance retrofit (PAM)

If you're adding factory parking sensors (PAM module `4/86`) to a car that
didn't come with them, the wiring connects to the REM, not the CEM. From the
data:

| REM pin | Wire color | PAM pin | Function |
|---|---|---|---|
| `B:21` | BL | `A:1` | Power feed (F26 fuse) |
| `D:32` | GN-W | `A:12` | LIN data bus |
| `A:8` | (already SB to ground 53/602) | `A:8` | Chassis ground |

The PAM communicates with the REM over LIN, not high-speed CAN, so latency
is fine for parking but you can't just bridge it to a generic CAN sniffer.
PAM's B-connector (front sensors) and C-connector (rear sensors) are
documented in the CSV with the L-h / R-h / inner / outer sensor positions.

The rear-only variant uses only the C-connector and 4 sensors (`7/131`
through `7/134`). The front-plus-rear variant adds `7/204` through `7/207`
on the B-connector via a 6-pin inline. LHD car sensor positions (US-market
S60R) are noted in the CSV.

## RTI replacement (Raspberry Pi digital dash / lap timer)
 
If you're replacing the factory RTI navigation unit (`16/45`) with a custom
display like a Pi-based digital gauge cluster or lap timer, the key pins
are:
 
| RTI pin | Wire color | Connects to | Notes |
|---|---|---|---|
| `D:3` | W | LOSPEED CAN HI (CEM `D:38`) | Read body/comfort data — speed, ignition status, dashboard brightness, etc. |
| `D:4` | GN | LOSPEED CAN LO (CEM `D:53`) | Twisted pair with D:3 |
| (power) | R-W | REM `B:8` (F6 fuse) | Shared with CD changer; switched 12V |
 
The CAN bus that RTI sits on is the **low-speed CAN**. The high-speed CAN
(ECU/TCM/brakes) is a separate bus and the CEM acts as a gateway between
the two — so reading engine-specific data like boost pressure or coolant
temp from the RTI plug isn't straightforward. The high-speed bus is
accessible at the ECM (`B:1` W = HI, `B:13` GN = LO) or at the DLC.
 
As for CAN speeds for model years:
* 04' - CANL - speed = 125kbps, CANH - speed = 250kbps
* 05+ - CANL - speed = 250 kbps,CANH - speed = 500kbps

Community projects below have done the work on similar/related platforms and are the best starting point:
 
* **[martonn98/VolvoP2_CAN](https://github.com/martonn98/VolvoP2_CAN)** — Arduino + MCP2515 sniffing on a 2002 S60 (P2). 
* **[Alfaa123/Volvo-CAN-Gauge](https://github.com/Alfaa123/Volvo-CAN-Gauge)**  — Boost/coolant/intake/timing gauge on a 2011 C30 T5 using a Macchina M2.
* **[najnesnaj/moosesnif](https://github.com/najnesnaj/moosesnif)** — P1 reverse engineering reference using STM32 and the Volvo DICE cable, with a work-in-progress documentation PDF.

Useful starting points if you're going to write code:
* **[python-can](https://python-can.readthedocs.io/)** with SocketCAN, or
* **[OpenGarages handbook](http://opengarages.org/handbook/ebook/)** (linked from
the Alfaa123 repo) for general car-hacking background.
 
The RTI's original power feed (`R-W` from REM `B:8`, F6 fuse) is switched
with the radio, so a Pi tapped into it boots when the radio comes on and
shuts down when the radio goes off. The factory display behind the RTI
bezel is a proprietary interface — if you've successfully reused it on a
P2 S60R, please open a PR.

## Files

| File | Contents |
|---|---|
| `volvo_s60r_2005_ALL_PINOUT_verified.csv` | **Main deliverable.** 433 verified pins across 20 modules. |
| `volvo_s60r_2005_ECM_pinout_verified.csv` | ECM-only subset (75 pins on connectors A and B). |
| `volvo_s60r_2005_components.csv` | All 396 component IDs from the manual's master list (e.g. `4/56 = CEM`, `8/6 = Fuel injector cyl 1`). Useful as a lookup when reading the "Goes To" column. |

## CSV columns

| Column | Meaning |
|---|---|
| Module | The module the pin belongs to (ECM, CEM, REM, TCM, SRS, etc.) |
| Connector | Sub-connector letter on the module (A, B, C, D, E, F, K) |
| Pin | Pin number within that connector |
| Wire Color | Two-letter Volvo color code (see below). Hyphenated = two-tone. |
| Goes To | The component the wire connects to (Volvo component ID, e.g. `8/6`, `4/56`, `53/319`) |
| Source Pin | The pin number on the "Goes To" component, where applicable |
| Function / Role | Plain-English description of what the wire does |
| Confidence | HIGH / MEDIUM / LOW — see methodology below |
| Notes | Variant filters, conflicts, things to verify on the harness |
| Source PDF Pages | Page(s) in TP 3976201 where the connection was read |

## Module breakdown

| Module | Pins | Description |
|---|---|---|
| ECM (4/46) | 75 | Engine Control Module — 5-cyl turbo |
| TCM (4/28) | 8 | Transmission Control Module — AW55-50SN |
| CEM (4/56) | 118 | Central Electronic Module (body computer / gateway) |
| REM (4/58) | 84 | Rear Electronic Module |
| SRS (4/9) | 47 | Supplemental Restraint System (airbag controller) |
| CCM (3/112) | 12 | Climate Control Module |
| DIM (5/1) | 5 | Driver Information Module (instrument cluster) |
| SWM (3/254) | 4 | Steering Wheel Module |
| LSM (3/111) | 3 | Light Switch Module |
| FCM (4/31) | 6 | Fan Control Module |
| AUM (16/1) | 15 | Audio Module |
| DDM (3/126) | 9 | Driver Door Module |
| PDM (3/127) | 3 | Passenger Door Module |
| BCM (4/16) | 3 | Brake Control Module |
| ECPS / PSU (4/99) | 5 | Electronic Power Steering |
| SUM (4/84) | 3 | Suspension Module (Four-C) |
| DEM (4/82) | 3 | Differential Electronic Module (AWD) |
| GSM (3/156) | 4 | Gear Selector Module (Geartronic) |
| PAM (4/86) | 21 | Parking Assistance Module (rear-only or rear+front) |
| RTI (16/45) | 5 | Road Traffic Information (navigation/infotainment) |
| **Total** | **433** | |

## Volvo color code legend

| Code | Color |
|---|---|
| BL | Blue |
| BN | Brown |
| GN | Green |
| GR | Gray |
| OR | Orange |
| P | Pink |
| R | Red |
| SB | Black |
| VO | Violet |
| W | White |
| Y | Yellow |

Hyphenated codes (e.g. `GN-W`) are two-tone — primary color first, tracer
second. So `GN-W` is a green wire with a white tracer.

## Common Volvo schematic conventions

* `X/Y` notation: component category / instance. `8/6` is fuel injector
  number 6 (where `8` = "fuel injector" category).
* `53/xxx` junctions: internal harness splice points.
* `54/xxx` connectors: inline harness connectors (where two harness sections
  meet, often at the firewall, bulkhead, or under-dash).
* `31/xxx` grounds: chassis ground points.
* `11A/B/C/E/x` fuses: the letter is the fuse box (A = passenger compartment
  upper, B = passenger compartment lower, C = passenger compartment, E =
  engine compartment battery box), the number is the position.
* `15/31` is the engine compartment distribution box. Sub-pins like
  `15/31_C4:6` mean connector C4, pin 6 of that distribution box.
* Twisted pair symbols (figure-8 between two parallel lines) indicate
  twisted-pair wiring — almost always CAN or other differential signaling.
* Dashed lines = optional / variant-specific paths or shields.

## Variant filtering

Volvo's wiring diagrams pack many model and option variants onto the same
page. The S60R-applicable subset is what's in this CSV — but the Notes column
flags pins that:

* Apply only to other variants (S60, S80, M66 manual, Bi-fuel CNG/LPG, etc.)
* Have a "no" applicability for S60R (e.g. `MAN-only`, `W/O ECPS`)
* Are S60R-specific (`ECPS`, `Bi-X`, `AWD`, `FOUR-C`, `AUTO AW55-50SN`)

## Confidence levels

* **HIGH (422 pins)**: Wire color label sits directly next to the pin label
  in the schematic and the destination is unambiguously traceable. Verified
  against multiple pages where possible.
* **MEDIUM (7 pins)**: Routing is clear but one element (a wire color label,
  a specific device pin number) was uncertain in the scan. Listed details are
  the most likely reading.
* **LOW (4 pins)**: Pin label is visible on the CEM block but the wire color
  or destination could not be read confidently from the available pages.

## Known issues flagged in the data

1. **CEM A:6 (page 36 vs page 37 vs audio page)** — The manual itself
   contains a contradiction. Page 36 (CEM 1:2) reads `A:6 ← Y-R from 10/19:1`
   (auxiliary brake light). Page 37 (CEM 2:2) and the dedicated Audio system
   page both read `A:6 ← VO from 16/1A:2` (audio amplifier, F28 fuse). The
   CSV uses the audio reading (two pages agree). The aux brake light wire is
   likely on **A:5** or **A:7** rather than A:6 — likely a printing error in
   the brake-lights schematic. Verify on the physical harness.
2. **REM F:2 ambiguity** — Two label positions on REM 1:2 both read as
   "F:2". One is likely a misprint (probably F:9 or F:12).
3. **REM F:10** — Wire color label is cut off in the scan; the color string
   reads as "GR-" with the second component missing.
4. **CEM B:50, B:35 (fog lights page) and B:47 (reversing lights page)** —
   Pin labels visible at the CEM block but wire color/destination not
   resolvable from the system page alone.
5. **Reverse light circuit (S60R AUTO)** — The CEM `C:34` reverse-light input
   from `3/10:1` is marked MAN-only in the schematic. On AUTO transmission
   the reverse signal travels over CAN from the TCM; the physical reverse
   light feed is from the REM (`B:7` for L-h and `B:10` for R-h, F1 fuse).
6. **Manual transmission swap notes** — `ECM B:4`, `B:29`, and `B:15` are
   marked as MAN-swap wiring requirements. For S60R AUTO they are unused;
   for a MAN swap you need to fabricate the `53/324` junction yourself and
   run `BN` from B:4 + `SB` from B:15 to the cabin to connect to the clutch
   position sensor (`7/123`).

## Methodology and provenance

* **Source PDF**: Volvo TP 3976201, 2005 S60/S60R/S80 Wiring Diagrams,
  Section 3-39 (135 pages).
* **Workflow**: each diagram page was rasterized at 200 DPI, regions of the
  page cropped to readable size, pin labels and wire colors transcribed by
  hand, and each batch verified by the project owner (who has the S60R
  harness physically out of the car for repair) against the actual harness
  with multimeter spot-checks.
* **Coverage**: dedicated module pages (ECM 1:2 / 2:2, CEM 1:2 / 2:2, REM 1:2
  / 2:2), the three CAN/LIN pages, and ~20 system-specific pages (lighting,
  brakes, SRS, climate, audio, power windows, central locking, ECPS, Four-C,
  Geartronic, etc.).
* **Not covered**: low-priority systems like sunroof, TV tuner, remote
  parking heater, and rarely-relevant accessory pages. The pinouts for
  those modules can be added later if needed.

## Contributing

Found an error or want to add a pin? Open a pull request. Please include:
* The PDF page (or photo of the harness) showing what you found.
* The before / after values for the affected row.
* If verifying against a physical harness, mention which car (year, market,
  options) — variant differences are real and worth documenting.

## License

The CSV data in this repo is licensed CC0 — public domain.
The original Volvo TP 3976201 document is © Volvo Car Corporation and is
NOT redistributed here. Find it through normal channels. Not affiliated with Volvo Car Corporation at all. Just a proud owner of multiple 200,000mi Volvos, one of which saved my life.

## Warranty (or lackthereof)

If you cut your wiring harness or fry your expensive, unobtanium modules, that is on you. Don't use this unless you're comfortable with wiring stuff or prepared to spend thousands of dollars getting modules flashed by XeMODex.

## Acknowledgements

Compiled with multi-turn human-AI collaboration: an AI assistant did the
optical transcription from the PDF, the project owner verified each batch
against the physical harness and corrected misreadings. I've done this manually over the years and copy/pasting pins and writing stuff on paper gets lost. 

# Sorting rules — authoritative bin map

This is the source of truth for what goes where. If the control program and this document ever
disagree, one of them is a bug. Don't silently trust either.

**Revision 2026-09-14.**

**Logs are cut by a person who knows what they are cutting.** The machine only has to recognise
the listed product sizes. Anything else goes to the no-sort bin, and the gaps between products
are deliberate.

**All diameters are small-end diameters.** See section 3.

---

## 1. The bin map

| Bin | Product | Diameter (small end) | Length |
|---|---|---|---|
| **1** | 8 footers | 3" – 4" | 7'6" – 8'6" |
| **3** | 8 footers | 4" – 5" | 7'6" – 8'6" |
| **5** | 8 footers | 5" – 6" | 7'6" – 8'6" |
| **7** | 8 footers | 6" – 9" | 7'6" – 8'6" |
| **8** | large, **one paddle** | 9" – 12" | 9'9" – 17'6" |
| **10** | jacks | 3" – 6" | 5'6" – 6'6" |
| **12 + 14** | poles *(both paddles from 13 ft)* | 2.6" – 4" | 10'0" – 22'0" |
| **9** | **no-sort** — the default | everything else | everything else |

**Bins 2, 4, 6, 11 and 13 receive nothing.**

**Edges:** the lower edge is inclusive and the upper edge exclusive. The exceptions are bin 8's
17'6" and the poles' 22'0", which are inclusive. A log of exactly 8'6" is no-sort.

---

## 2. Diameter arrives as a band, not inches

The light grid reports height as threshold bits. The program counts the contiguous run of set
bits from the bottom, which gives a band index. Every diameter boundary the rules need
(2.6, 3, 4, 5, 6, 9, 12) sits exactly on a band edge.

| Band | Diameter | Bins it can select |
|---|---|---|
| 0 | below 2.6" | 9 |
| 1 | 2.6" – <3" | 12+14 |
| 2 | 3" – <4" | 1, 10, 12+14 |
| 3 | 4" – <5" | 3, 10 |
| 4 | 5" – <6" | 5, 10 |
| 5 | 6" – <9" | 7 |
| 6 | 9" – <12" | 8 |
| 7 | 12" and over | 9 only |

The run is counted rather than taking the highest set bit, so a floating obstruction fails safe:
it reports small rather than reporting a large log.

---

## 3. Small-end diameter

A log is thicker at the butt than at the top. The sizes above are small-end sizes, so the machine
measures the **thinnest point along the log**, not the thickest.

The machine divides each log into 6-inch slots as it passes the light grid. Each slot keeps the
highest reading it saw, and the smallest slot is the log's diameter. It ignores the first and
last 6 inches, so a rough or angled cut end is not mistaken for a thin log. It does not matter
which end comes first.

---

## 4. Coverage grid

Rows are measured length. Columns are small-end band. This is the complete picture: every
combination lands somewhere.

| Length | b0 <2.6 | b1 2.6–3 | b2 3–4 | b3 4–5 | b4 5–6 | b5 6–9 | b6 9–12 | b7 12+ |
|---|---|---|---|---|---|---|---|---|
| < 5'6" | 9 | 9 | 9 | 9 | 9 | 9 | 9 | 9 |
| 5'6" – 6'6" | 9 | 9 | 10 | 10 | 10 | 9 | 9 | 9 |
| 6'6" – 7'6" | 9 | 9 | 9 | 9 | 9 | 9 | 9 | 9 |
| 7'6" – 8'6" | 9 | 9 | 1 | 3 | 5 | 7 | 9 | 9 |
| 8'6" – 9'9" | 9 | 9 | 9 | 9 | 9 | 9 | 9 | 9 |
| 9'9" – 10' | 9 | 9 | 9 | 9 | 9 | 9 | 8 | 9 |
| 10' – 13' | 9 | 12 | 12 | 9 | 9 | 9 | 8 | 9 |
| 13' – 17'6" | 9 | 12+14 | 12+14 | 9 | 9 | 9 | 8 | 9 |
| 17'6" – 22' | 9 | 12+14 | 12+14 | 9 | 9 | 9 | 9 | 9 |
| > 22' | 9 | 9 | 9 | 9 | 9 | 9 | 9 | 9 |

No two products overlap, so the order the rules are checked in cannot change the result.

---

## 5. Poles and the end of the belt

Poles are kicked by bins 12 and 14 together, aimed at the **midpoint between their two stations**,
77.75 ft from the light grid. Aimed at one station alone, the other paddle would sit about 12 ft
from the log's centre, which is past the end of the log.

**Poles under 13 ft get one paddle.** The stations are about 12 ft apart, so a shorter log cannot
reach both. Bin 12 takes it alone.

**Poles over 21.5 ft would overhang the end of the belt** if centred on 77.75 ft. The machine pulls
the aim point back until the pole's leading end sits at the end of the belt. On a 22 ft pole the
paddles still land 5.7 ft and 6.2 ft either side of its centre. The `xAimClamped` indicator latches
when this happens. On a long pole that is expected, not a fault.

---

## 6. Design decisions worth knowing

**Bin 9 is the default.** Every log that matches no product falls to it. A rule set built from
product sizes always has holes, and an explicit default is what makes them safe. Bin 9 sits
mid-belt at 59.5 ft, so it can place even very long odd logs.

**Bin 8 fires alone at every length.** It takes large-diameter logs up to 17'6" on a single paddle.

**Over-length is not treated as an error.** Lengths are cut deliberately. Anything the kickers
cannot clear is a manual pull with the automation off, because the conveyor runs independently.

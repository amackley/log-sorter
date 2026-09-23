# Sorting rules — authoritative bin map

This is the source of truth for what goes where. If the control program and this document ever
disagree, one of them is a bug. Don't silently trust either.

**Revision 2026-09-23.**

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
| **13** | jacks | 3" – 6" | 5'6" – 6'6" |
| **11** | short poles *(one paddle)* | 2.6" – 6" | 10'0" – 14'0" |
| **12 + 14** | long poles *(always paired)* | 2.6" – 6" | 14'0" – 25'0" |
| **6 + 8** | big logs *(paired from 13 ft)* | 7" – 12" | 9'9" – 25'0" |
| **4** | **no-sort** — the default | everything else | everything else |

**Bins 2, 9 and 10 receive nothing.** Bins 8 and 14 fire only as partners of 6 and 12.

**Edges:** the lower edge is inclusive and the upper edge exclusive. The exception is 25'0",
which is inclusive. A log of exactly 8'6" is no-sort, and one of exactly 14'0" is a long pole.

**6"–7" logs longer than 8'6" go to no-sort.** They are re-cut to 8 ft and come back as bin 7.

---

## 2. Diameter arrives as a band, not inches

The light grid reports height as eight threshold bits. The program counts the contiguous run of
set bits from the bottom, which gives a band index. Every diameter boundary the rules need
(2.6, 3, 4, 5, 6, 7, 9, 12) sits exactly on a band edge.

| Band | Diameter | Bins it can select |
|---|---|---|
| 0 | below 2.6" | 4 |
| 1 | 2.6" – <3" | 11, 12+14 |
| 2 | 3" – <4" | 1, 13, 11, 12+14 |
| 3 | 4" – <5" | 3, 13, 11, 12+14 |
| 4 | 5" – <6" | 5, 13, 11, 12+14 |
| 5 | 6" – <7" | 7 only |
| 6 | 7" – <9" | 7, 6+8 |
| 7 | 9" – <12" | 6+8 |
| 8 | 12" and over | 4 only |

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

Rows are measured length. Columns are small-end band. Every combination lands somewhere.

| Length | b0 <2.6 | b1 2.6–3 | b2 3–4 | b3 4–5 | b4 5–6 | b5 6–7 | b6 7–9 | b7 9–12 | b8 12+ |
|---|---|---|---|---|---|---|---|---|---|
| < 5'6" | 4 | 4 | 4 | 4 | 4 | 4 | 4 | 4 | 4 |
| 5'6" – 6'6" | 4 | 4 | 13 | 13 | 13 | 4 | 4 | 4 | 4 |
| 6'6" – 7'6" | 4 | 4 | 4 | 4 | 4 | 4 | 4 | 4 | 4 |
| 7'6" – 8'6" | 4 | 4 | 1 | 3 | 5 | 7 | 7 | 4 | 4 |
| 8'6" – 9'9" | 4 | 4 | 4 | 4 | 4 | 4 | 4 | 4 | 4 |
| 9'9" – 10' | 4 | 4 | 4 | 4 | 4 | 4 | 6 | 6 | 4 |
| 10' – 13' | 4 | 11 | 11 | 11 | 11 | 4 | 6 | 6 | 4 |
| 13' – 14' | 4 | 11 | 11 | 11 | 11 | 4 | 6+8 | 6+8 | 4 |
| 14' – 25' | 4 | 12+14 | 12+14 | 12+14 | 12+14 | 4 | 6+8 | 6+8 | 4 |
| > 25' | 4 | 4 | 4 | 4 | 4 | 4 | 4 | 4 | 4 |

The 13'–14' row is separate because **big logs start using two paddles at 13 ft, while short
poles always use one.** No two products overlap, so the order the rules are checked in cannot
change the result.

---

## 5. Paired kicks and the end of the belt

Long logs are kicked by two paddles together, aimed at the **midpoint between their two
stations**: 41.5 ft from the light grid for bins 6+8, and 77.75 ft for bins 12+14. Aimed at one
station alone, the other paddle would sit about 12 ft from the log's centre, past the end of the
log.

**Big logs under 13 ft get one paddle**, bin 6, because the stations are about 12 ft apart.
**Short poles always get one paddle**, bin 11. **Long poles always get two**, because they start
at 14 ft.

**Bins 6+8 have room for the full 25 ft.** A 25 ft log centred at 41.5 ft ends at 54 ft.

**Logs over 21.5 ft on bins 12+14 would overhang the end of the belt** if centred on 77.75 ft.
The machine pulls the aim point back until the leading end sits at the end of the belt. On a
25 ft log the paddles still land 4.2 ft and 7.7 ft either side of centre. `xAimClamped` latches
when this happens, and on those logs it is expected.

---

## 6. Design decisions worth knowing

**Bin 4 is the default.** Every log that matches no product falls to it. A rule set built from
product sizes always has holes, and an explicit default is what makes them safe.

**Bin 4 is close to the head of the belt, 23.8 ft in.** Anything measuring over about 42 ft
cannot be placed there, so it fires immediately and latches `xKickTooLateFault`. A beam jam
produces exactly such a phantom measurement.

**Over-length is not treated as an error.** Lengths are cut deliberately. Anything the kickers
cannot clear is a manual pull with the automation off, because the conveyor runs independently.

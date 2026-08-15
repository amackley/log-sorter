# Sorting rules — authoritative bin map

This is the source of truth for what goes where. If the control program and this document ever
disagree, one of them is a bug — do not silently trust either.

**Lengths are cut deliberately and held consistent.** Products are nominal 7', 8', 10', 17'+ and
21' "plus inches". The categories below are product classes, not a partition of the whole
measurement space, which is why the catch-all rules exist.

---

## 1. The bin map

| Bin | Category | Diameter | Length |
|---|---|---|---|
| **1** | jacks | 3" – 6" | 5.75 – <7 ft |
| **2** | no-sort / scrap | <2.6" | *or* <5.75 ft |
| **3** | 8 footers | 4.5" – 6" | 8 – 10 ft |
| **4** | 8 footers | 2.6" – 4.5" | 8 – 10 ft |
| **5** | 8 footers | 6" – 7" | 8 – 10 ft |
| **7** | 8 footers | >7" | 8 – 10 ft |
| **9** | 10 footers | ≥7.5" | 10 – 12 ft |
| **11** | 10 footers | 2.6" – 7.5" | 10 – 12 ft |
| **13** | 7 footers | 2.6" – 13" | 7 – 8 ft |
| **6 + 8** | large poles *(paired kick)* | >6" | ≥17 ft |
| **12 + 14** | 21 ft poles *(paired kick)* | 2.6" – 6" | >20 ft |
| **10** | catch-all — good log, no product category | — | — |

---

## 2. Diameter arrives as a band, not inches

The light grid reports height as eight threshold bits. The program counts the contiguous run of
set bits from the bottom, giving a band index. **Six bands are exactly sufficient** — every
diameter boundary the rules need (2.6, 3, 4.5, 6, 7, 7.5) lands precisely on a band edge.

| Band | Diameter | Bins it can select |
|---|---|---|
| 0 | below 2.6" | 2 |
| 1 | 2.6" – <3" | 4, 11, 13, 12+14 |
| 2 | 3" – <4.5" | 1, 4, 11, 13, 12+14 |
| 3 | 4.5" – <6" | 1, 3, 11, 13, 12+14 |
| 4 | 6" – <7" | 5, 11, 13, 6+8 |
| 5 | 7" – <7.5" | 7, 11, 13, 6+8 |
| 6 | ≥7.5" | 7, 9, 13, 6+8 |

The run is counted rather than taking the highest set bit, so that a floating obstruction fails
safe — it reports small rather than reporting a large log.

A log measuring exactly 6.000" is theoretically ambiguous between bin 5 and the large-pole rule.
It resolves to the large side in both, so the ambiguity has no effect.

---

## 3. Coverage grid

Rows are measured length, columns are height band. This is the complete picture — every
combination lands somewhere.

| Length | b1 2.6–3 | b2 3–4.5 | b3 4.5–6 | b4 6–7 | b5 7–7.5 | b6 7.5+ |
|---|---|---|---|---|---|---|
| <5.75 | 2 | 2 | 2 | 2 | 2 | 2 |
| 5.75 – 7 | 2 | 1 | 1 | 2 | 2 | 2 |
| 7 – 8 | 13 | 13 | 13 | 13 | 13 | 13 |
| 8 – 10 | 4 | 4 | 3 | 5 | 7 | 7 |
| 10 – 12 | 11 | 11 | 11 | 11 | 11 | 9 |
| 12 – 17 | 10 | 10 | 10 | 10 | 10 | 10 |
| 17 – 20 | 10 | 10 | 10 | 6+8 | 6+8 | 6+8 |
| 20 – 21.5 | 12+14 | 12+14 | 12+14 | 6+8 | 6+8 | 6+8 |
| >21.5 | 6+8 | 6+8 | 6+8 | 6+8 | 6+8 | 6+8 |

Band 0 — anything below 2.6" — is bin 2 at every length.

---

## 4. Rule order matters

The rules are evaluated in sequence, and the order is significant:

```
band = 0                      -> 2            no-sort, under 2.6"
len < 5.75                    -> 2            no-sort, too short
len < 7.0   AND band in 2..3  -> 1            jacks
len < 7.0                     -> 13 or 2
len < 8.0                     -> 13           7 footers
len < 10.0                    -> 4/4/3/5/7/7  8 footers, by band
len < 12.0                    -> 11, or 9 at band 6
len < 17.0                    -> 10           catch-all
len <= 20.0 AND band in 4..6  -> 6 + 8        large poles
len <= 20.0                   -> 10           catch-all
band in 4..6                  -> 6 + 8        large poles, over 20 ft
otherwise                     -> 12 + 14      21 ft poles
```

**No-sort must be evaluated before over-length, and over-length before the pole rules.** Getting
this wrong sends scrap into product bins in ways that are not obvious from spot-checking.

---

## 5. Why long logs need two paddles

Bins 6+8 and 12+14 fire together, aimed at the **midpoint between their two stations** — 41.5 ft
and 77.75 ft respectively.

Aimed at either station alone, the partner paddle would sit about 12 ft from the log's centre —
past the end of even a 20 ft log — and swing through empty air.

### The 21.5 ft cap

Bins 12+14 are the last two stations. A log centred on their 77.75 ft midpoint has its leading
end at `77.75 + L/2` against an 88.5 ft belt, so `2 × (88.5 − 77.75) = 21.5 ft` is the longest
they can physically place. **A nominal 21 ft pole clears by 3 inches.**

Bins 6+8 sit mid-belt at 41.5 ft and could centre a 94 ft log, so **anything past the cap goes to
6+8 regardless of diameter**. Routing over-length material to 12+14 would send it to the one pair
that cannot place it.

> **That 3-inch margin is the tightest tolerance in the system**, and it sits inside the combined
> error budget of the belt speed constant and the length measurement. A pole measured 2% long
> moves the margin by about 5 inches — more than the margin itself. This is the reason to
> calibrate belt speed against a known-length log **before** running poles.

---

## 6. Design decisions worth knowing

**Bin 2 is the true default.** Every unmatched log falls to it. A rule set built from product
categories will always have holes; making the default explicit is what makes them safe.

**Bin 10 is the catch-all for good log with no product category** — mostly 12 to 17 ft material,
and 17 to 20 ft below 6". It is not scrap. It exists so off-cuts get re-cut rather than
discarded.

**Over-length is not treated as an error.** Lengths are cut deliberately, and anything the
kickers cannot clear is a manual pull with the automation off — the conveyor runs independently.
So the cap is set purely by geometry.

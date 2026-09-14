# Operation

For anyone running the machine. No programming knowledge assumed.

---

## What happens to a log

1. The log breaks the light grid beam at the head of the belt.
2. While the beam is broken, the controller measures **how tall** the log is — that gives
   diameter — and **how long the beam stays broken** — that gives length, because the belt speed
   is known.
3. When the log's tail clears the beam, it is classified into a bin and put in a queue.
4. The controller works out when the log's centre will reach that bin's station, and fires the
   paddle at that moment.

Everything after step 2 is arithmetic on a stopwatch. **Nothing confirms the log actually landed
in the bin.** If logs start showing up in the wrong place, the machine will not tell you — see
[TROUBLESHOOTING.md](TROUBLESHOOTING.md).

---

## The bins

Diameters are **small-end** diameters: the machine uses the thinnest point along the log.

| Bin | What goes there |
|---|---|
| **1** | 8 footers, 3" to 4", 7'6" to 8'6" |
| **3** | 8 footers, 4" to 5" |
| **5** | 8 footers, 5" to 6" |
| **7** | 8 footers, 6" to 9" |
| **8** | **Large diameter**, 9" to 12", 9'9" to 17'6". *One paddle* |
| **9** | **No-sort**: anything not listed here |
| **10** | Jacks, 3" to 6", 5'6" to 6'6" |
| **12 + 14** | **Poles**, 2.6" to 4", 10' to 22'. *Both paddles fire together from 13 ft* |

Bins 2, 4, 6, 11 and 13 receive nothing.

Full detail, including exactly where the boundaries fall, is in
[reference/sorting-rules.md](reference/sorting-rules.md).

**Bin 9 is the default.** Anything the rules do not recognise ends up there, which is the safe
outcome. Genuine scrap and anything unexpected both land in the same place rather than being
forced into a product bin.

### Bin 14 never receives a log on its own

It only ever fires as the partner of bin 12. A long pole needs two paddles striking together, so
the machine aims at the midpoint between the two stations and fires both. Poles under 13 ft are
too short to reach both paddles, so bin 12 takes them alone. If you see bin 14 targeted alone,
something is wrong.

---

## Limits of the measurement

**2.5" and 2.6" material read the same.** They are about a tenth of an inch apart, against an
optical grid that resolves roughly 0.16". The threshold is set to err **inclusive**: material
from roughly 2.45" up is treated as sortable rather than no-sort. The alternative pushes good
2.6" to 2.75" logs into the no-sort bin. This is a deliberate choice and can be reversed by
changing one value.

**Oversize material goes to no-sort.** A small end of 12" or more matches no product and lands in
bin 9, and so does an 8-footer over 9".

---

## Faults

The controller tracks several fault conditions. None of them stops the machine — they latch as
indicators for whoever is diagnosing a problem.

| Fault | Means | What to do |
|---|---|---|
| `xBeamJamFault` | The beam has been blocked longer than 60 seconds | Something is sitting in the light grid. Clear it |
| `xQueueFullFault` | More logs in flight than the queue can track | Logs are being fed faster than the belt can carry them apart. Slow the feed |
| `xKickTooLateFault` | A log's centre was already past its station when it was classified | Usually a very long log routed to an early bin. Expected occasionally; a constant stream of them means the belt speed setting is wrong |
| `xAimClamped` | The aim point had to be pulled back to fit on the belt | Expected on poles over 21.5 ft. On anything else, the log is too long for the bin it was sent to |
| `xBinRangeFault` | The classifier produced a bin number outside 1–14 | Should never happen. Report it |

**A fault that latches once and stays on is not necessarily a live problem** — these do not clear
themselves. Note them, clear them at the start of a run, and see whether they come back.

---

## Simulation mode — leave it off

The controller has a built-in simulator that fabricates logs for testing. **On a live machine it
is dangerous**: it injects an invented log every 8 seconds and fires kickers at nothing.

Two settings control it, `SimMode` and `Sim_Auto`. Both are off by default and both should stay
off. `Sim_Auto` is the dangerous one — it works even when `SimMode` is off.

If the machine is firing paddles at an empty belt on a regular cycle, this is the first thing to
check.

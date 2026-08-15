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

| Bin | What goes there |
|---|---|
| **1** | Jacks — 3" to 6" diameter, roughly 6 ft |
| **2** | **No-sort / scrap** — anything under 2.6" diameter, or shorter than 5.75 ft |
| **3** | 8 footers, 4.5" to 6" |
| **4** | 8 footers, 2.6" to 4.5" |
| **5** | 8 footers, 6" to 7" |
| **6 + 8** | **Large poles** — over 6", 17 ft and up. *Both paddles fire together* |
| **7** | 8 footers, over 7" |
| **9** | 10 footers, 7.5" and up |
| **10** | **Catch-all** — good log that fits no product category |
| **11** | 10 footers, 2.6" to 7.5" |
| **12 + 14** | **21 ft poles** — 2.6" to 6", over 20 ft. *Both paddles fire together* |
| **13** | 7 footers, 2.6" to 13" |

Full detail, including exactly where the boundaries fall, is in
[reference/sorting-rules.md](reference/sorting-rules.md).

**Bin 2 is the default.** Anything the rules do not recognise ends up there, which is the safe
outcome — genuine scrap and anything unexpected both land in the same place rather than being
forced into a product bin.

**Bin 10 is not scrap.** It is good, usable log that does not match a product size — most often
12 to 17 ft material. It exists so off-cuts get re-cut rather than thrown away.

### Bins 8 and 14 never receive a log on their own

They only ever fire as the partner of bin 6 and bin 12. A long pole needs two paddles striking
together, so the machine aims at the midpoint between the two stations and fires both. If you
see bin 8 or bin 14 targeted alone, something is wrong.

---

## Two things that cannot be distinguished

**2.5" and 2.6" material read the same.** They are about a tenth of an inch apart, against an
optical grid that resolves roughly 0.16". The threshold is currently set to err **inclusive** —
material from roughly 2.45" up is treated as sortable rather than scrap. The alternative pushes
good 2.6" to 2.75" logs into the no-sort bin. This is a deliberate choice and can be reversed by
changing one value.

**There is no over-diameter rejection.** Bin 13's stated 13" upper limit is not enforced
anywhere. Oversize material will be sorted as though it were in range. The sensor does measure
it, so a rejection rule could be added if wanted.

---

## Faults

The controller tracks several fault conditions. None of them stops the machine — they latch as
indicators for whoever is diagnosing a problem.

| Fault | Means | What to do |
|---|---|---|
| `xBeamJamFault` | The beam has been blocked longer than 60 seconds | Something is sitting in the light grid. Clear it |
| `xQueueFullFault` | More logs in flight than the queue can track | Logs are being fed faster than the belt can carry them apart. Slow the feed |
| `xKickTooLateFault` | A log's centre was already past its station when it was classified | Usually a very long log routed to an early bin. Expected occasionally; a constant stream of them means the belt speed setting is wrong |
| `xAimClamped` | The aim point had to be pulled back to fit on the belt | The log is too long for the bin it was sent to |
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

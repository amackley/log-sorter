# Commissioning

First-time setup, joining the control system to the machine. **Do these in order** — several
later steps depend on earlier ones.

This assumes the control program is already loaded and the controller boots standalone. What
follows is everything physical.

---

## 1. Prerequisites

- [ ] Belt installed, running, and speed-stable
- [ ] Light grid mounted at the head end, emitter and receiver aligned
- [ ] 24 V control power to the rail
- [ ] Air supply to the kicker solenoids
- [ ] Laptop with the development environment, if code changes may be needed

## 2. Wire and verify the kicker outputs

**This is the biggest untested link in the system.** A swapped pair is completely invisible from
software: the logic is correct, the log lands in the wrong bin, and nothing reports a fault.

Wire per the table in [MAINTENANCE.md](MAINTENANCE.md), then verify **every one** using the
self-test, watching which paddle physically moves.

- [ ] Consecutive pairs (1&2, 3&4, …) move paddles at the **same** station
- [ ] Station distance climbs with the number — 1&2 nearest the sensor, 13&14 furthest
- [ ] **Kickers 6, 8, 12 and 14 fired twice each** — they are the paired-kick members
- [ ] Self-test returned to off, with no output left energised

## 3. Calibrate the light grid to the belt

**Required.** Any thresholds set before the grid was mounted are referenced to whatever surface
was used at the time, and will all be wrong once it sits over the belt.

Only the eight numbers change. Direction, bit mapping, polarity and the rest of the chain carry
over untouched.

Full procedure in [MAINTENANCE.md](MAINTENANCE.md).

- [ ] Grid re-calibrated against the belt surface
- [ ] Each reference block produces a contiguous bit pattern
- [ ] Material below 2.6" produces band 0
- [ ] Small material still breaks the beam — nothing passes through unseen
- [ ] **New threshold values written into the table in MAINTENANCE.md**

## 4. Verify belt speed

Length errors scale everything downstream. Get this right before tuning kick timing.

1. Run a log of **known length** through.
2. Read the measured length.
3. Correct the belt speed constant by any consistent ratio error.

- [ ] Measured length accurate against a known-length log

## 5. Tune the kick timing

Only after step 4, and **on a single kick** — never a paired one. Procedure in
[MAINTENANCE.md](MAINTENANCE.md).

- [ ] Single kicks land on the log's centre
- [ ] A paired kick (6+8 or 12+14) lands correctly on a long log

## 6. Standalone verification

The machine must run with no PC attached.

1. With the final code running, write the boot application.
2. Physical run/stop switch to **RUN**.
3. Shut down the development software and **unplug the ethernet cable** — otherwise you cannot be
   sure the PC is not propping something up.
4. Kill 24 V, wait 10 seconds, restore.
5. Confirm controller LEDs come up normal.
6. Run test logs and confirm correct sorting from the relay indicators alone.

- [ ] Boots and sorts with no PC attached

## 7. Sign-off

- [ ] All 14 kickers verified against the correct station
- [ ] Grid calibrated to the belt, thresholds **recorded in MAINTENANCE.md**
- [ ] Sub-2.6" material produces band 0; small material still breaks the beam
- [ ] Measured length accurate against a known log
- [ ] Single kicks land on centre
- [ ] A paired kick lands correctly on a long log
- [ ] Boots and sorts with no PC attached
- [ ] Fault indicators clear after a production run
- [ ] Simulator flags confirmed off
- [ ] Threshold table updated and committed

---

## Known limitations to raise with whoever owns the machine

These are properties of the design, not defects, and are better discussed before the machine is
in production than after.

- **2.5" and 2.6" material cannot be distinguished.** The threshold currently errs inclusive —
  material from roughly 2.45" up is treated as sortable. The alternative pushes good 2.6"–2.75"
  logs into no-sort. One value changes it. See [OPERATION.md](OPERATION.md)
- **There is no over-diameter rejection.** Oversize material is sorted as though in range. The
  sensor measures it, so a rule could be added
- **Bins 1 and 13 have very narrow length windows** and are verified only in simulation — their
  beam-break times are shorter than a person can reproduce by hand. Real production logs will
  exercise them properly
- **Length depends on belt speed being stable.** A belt that slows under load reads logs long
- **Nothing confirms a log landed in its bin.** A mechanical problem downstream shows up as
  mis-sorted product, not as an alarm

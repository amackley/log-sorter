# Maintenance

Procedures that need doing after a sensor swap, a mechanical change, or when sorting accuracy
drifts.

---

## Re-calibrating the light grid

**Required whenever:** the grid is moved or remounted, the sensor is replaced or factory reset,
the belt surface height changes, or someone changes the sensor's Beam Mode.

The eight height thresholds live **only in the sensor's own memory**. No file backs them up —
this has been verified. The table at the end of this section is the only record. Keep it current.

### Tool

**WAGO-I/O-CHECK.** This is licensed WAGO software (part number 759-302/000-923 or 759-920),
available from WAGO. Connect to the controller at `192.168.1.3`, then Show Device on the 750-1657
Port 1.

**The PLC must be stopped** — put the physical run/stop switch in STOP, so the control program
releases the IO-Link port.

### Before you start

**Untick `Read Process Data`.** The cyclic poll blocks parameter writes. Leaving it ticked
produces a `WriteRequest timeout, Index 104` error that looks like a permissions problem and is
not.

**Press `Upload` first.** Until you do, the Value column shows defaults from the description
file, not what is actually in the sensor. You know it worked when Product reads
`LGS8-800-IO/110/115b`.

Confirm these settings, all under Parameter:

| Parameter | Required value | Why |
|---|---|---|
| Object Identification Mode | **`LGS (Light Grid Mode)`** | Anything else kills beam detection while leaving the height channels working |
| Beam Mode | **`Three Beam Crossing`** | Halves the effective grid pitch to ~4.2 mm. **Changing this wipes every threshold** |
| Detection Reference | `Reference Opposite Cable Side` | |
| Small Object Suppression | `0` | |
| Switching Signal Polarity | `Not Inverted` | |
| Height Control 1–8 Polarity | `Not Inverted` | |
| Height Control 1–8 Mode | **`Active`** | A threshold does nothing while its channel is Inactive. Separate setting from the value |

### Measuring

**Do not calculate the thresholds from inches.** The sensor under-reports object height by a
non-constant amount — between roughly 7.6 and 14.2 mm depending on size. Every attempt to derive
these numbers arithmetically has failed. Measuring works first time.

For each reference block, resting on the belt surface:

1. Leave the block in the beam.
2. Untick `Read Process Data`, press `Upload`.
3. Read **Observation → Measurement Value → Highest Object Position**.

Use `Highest Object Position`, never `Object Height` — the latter jitters by several mm.

Then write that measured number as the threshold for that boundary's channel. The trigger
condition is `reported ≥ Position`, so the measured value is exactly right. Set `Position 1` and
`Position 2` to the same number.

`Download`, then `Upload` again to confirm the values stuck.

> **Do not use the Teach-In buttons.** The teach command times out on this setup, and the
> Position fields are directly editable anyway. Typing measured values is both more reliable and
> more precise.
>
> **Never press Teach-In → Object Identification.** It switches the sensor out of Light Grid
> Mode, which kills beam detection while leaving the height channels working perfectly — a
> failure that looks like a wiring or controller fault and is neither.

### Threshold record

Keep a set of reference blocks with the machine. **Update this table whenever the values change
— it is the only backup.**

| Channel | Boundary | Position | Measured on |
|---|---|---|---|
| HC1 | 2.6" | 54 | bench reference, not the belt |
| HC2 | 3.0" | 62 | bench reference, not the belt |
| HC3 | 4.5" | 104 | bench reference, not the belt |
| HC4 | 6.0" | 142 | bench reference, not the belt |
| HC5 | 7.0" | 167 | bench reference, not the belt |
| HC6 | 7.5" | 179 | bench reference, not the belt |
| HC7 | 9" | 221 | instrumentation only |
| HC8 | 12" | 296 | instrumentation only |

HC7 and HC8 are not used by any sorting rule — the classifier saturates at the 7.5"+ band. They
exist so that oversize material is visible in diagnostics.

### Verifying

Re-tick `Read Process Data` and confirm each block produces a contiguous run of bits:

| Block | Height bits | Band |
|---|---|---|
| below 2.6" | 0 | 0 |
| 2.6" | 1 | 1 |
| 3.0" | 3 | 2 |
| 4.5" | 7 | 3 |
| 6.0" | 15 | 4 |
| 7.0" | 31 | 5 |
| 7.5" | 63 | 6 |
| 9" | 127 | 6 (saturated) |
| 12" | 255 | 6 (saturated) |

**A block below 2.6" must produce 0.** If it does not, the array sits too low relative to the
belt and small material will be misclassified as sortable.

**If a small block produces no beam break at all, the array is too high** — small logs pass
through completely unseen, with no length, no sort, and no record. That is the worst failure mode
in the system. Check for it explicitly.

---

## Verifying the kickers

**Required whenever:** a solenoid, relay or its wiring is changed.

A swapped pair is **completely invisible from software** — the logic is correct, the log lands in
the wrong bin, and nothing indicates a fault. It can only be caught by watching paddles move.

| Kicker | Station | Distance | Bin |
|---|---|---|---|
| 1 | 1 | 11.500 ft | 1 |
| 2 | 1 | 11.500 ft | 2 |
| 3 | 2 | 23.833 ft | 3 |
| 4 | 2 | 23.833 ft | 4 |
| 5 | 3 | 35.500 ft | 5 |
| 6 | 3 | 35.500 ft | **6** |
| 7 | 4 | 47.500 ft | 7 |
| 8 | 4 | 47.500 ft | **8** |
| 9 | 5 | 59.500 ft | 9 |
| 10 | 5 | 59.500 ft | 10 |
| 11 | 6 | 71.833 ft | 11 |
| 12 | 6 | 71.833 ft | **12** |
| 13 | 7 | 83.667 ft | 13 |
| 14 | 7 | 83.667 ft | **14** |

The controller has a self-test mode that drives one output at a time. Step through 1 to 14 and
watch which paddle actually moves. Check:

- Consecutive pairs (1&2, 3&4, …) move paddles at the **same** station. If not, the odd/even
  convention is reversed and every bin is misplaced.
- Station distance climbs with the number — 1&2 nearest the sensor, 13&14 furthest.
- **Kickers 6, 8, 12 and 14 each fire twice** — once on their own number and once as a paired
  partner. A swap within either pair puts a 20 ft pole on the floor.

> **Never leave an output energised.** Self-test holds the relay on solid, while normal operation
> pulses it for 300 ms. Set the number, observe, set it back to zero. Solenoid coils are
> generally not rated for continuous duty.
>
> While self-test is on, the control program stops its normal cycle. Counters freeze and length
> stops updating. **This is normal, not a crash.**

---

## Belt speed

Everything downstream scales with this. A wrong belt speed makes every length wrong, which makes
every classification wrong.

1. Run a log of **known length** through.
2. Read the measured length from the controller.
3. If it reads consistently long or short by a ratio, correct the belt speed constant by that
   same ratio.

Current value: **3.8295 ft/s**, from timing 88.5 ft in 23.11 s.

**Re-check this if sorting accuracy drifts over a shift.** A belt that slows under load will
read logs long. Check belt speed before suspecting anything else.

---

## Kick timing

A separate constant compensates for actuation lag — the delay between the output switching on
and the paddle actually striking.

Tune it **after** belt speed is confirmed, and **on a single kick**, never a paired one:

1. Send logs to a single-kick bin (1–5, 7, 9–11, 13).
2. Watch where the paddle strikes relative to the log's centre.
3. If it strikes **behind** centre, increase the lead. It is measured in seconds:
   1 ft early is about `0.26` s.

**How to tell this apart from a belt speed error:** if the miss grows the further down the belt
you go, it is belt speed. If every station misses by the same amount, it is kick timing.

Only once single kicks land correctly should you test a paired kick (6+8 or 12+14).

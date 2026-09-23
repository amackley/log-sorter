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
available from WAGO.

Getting to the sensor's own parameters takes three windows and the route is not obvious:

1. **WAGO-I/O-CHECK** — connect to `192.168.1.3`.
2. Select **`Pos. 02: 750-1657, 4-Port IO-Link Master`** in the Navigation tree.
3. **`Settings ▾` → `For the selected module…`** — this is the step nobody guesses. It opens the
   **WAGO IO-Link Tool**.
4. In the IO-Link Tool: **Ports** page, select **Port 1**, press **Connect**.
5. Press **`Show Device`**, next to the IODD filename. That opens **`IO-Link Device on Port1`**,
   which holds the Parameter tree and the Upload / Download / Write Changes buttons.
6. There: **Connect**, untick **`Read Process Data`**, then **Upload**.

**All three windows stay open.** Neither tool does the whole job — I/O-CHECK owns the connection
to the controller and the IO-Link master, the IO-Link Tool owns the sensor's own parameters.

> **You are only really talking to the sensor once `Product` reads `LGS8-800-IO/110/115b`.**
> Before that it shows a dash or description-file defaults, and every value on screen is
> meaningless. Each window shows `Offline` or `Disconnected` bottom-left until connected.

**Three things compete for the same channel and only one can have it:**

| Contender | How to clear it |
|---|---|
| The control program | Physical **run/stop switch to STOP** |
| I/O-CHECK's process data view | Close it; leave `Control-Mode (Direct)` off |
| The IO-Link Tool's parameter channel | Untick **`Read Process Data`** before any write |

That is what the `Read Process Data` rule is really about. It is not arbitrary — it is one of
three claimants on one channel, and leaving any of them running during a write produces a
timeout that reads like a permissions error and is not.

`Control-Mode (Direct)` also lets I/O-CHECK drive the digital outputs directly, so leave it off
unless you are deliberately testing the kicker relays.

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
| Detection Reference | **`Reference at Cable Side`** | Matches the **cable-down** mounting. See below |
| Small Object Suppression | `0` | |
| Switching Signal Polarity | `Not Inverted` | |
| Height Control 1–8 Polarity | `Not Inverted` | |
| Height Control 1–8 Mode | **`Active`** | A threshold does nothing while its channel is Inactive. Separate setting from the value |

### Measuring

**Do not calculate the thresholds from inches.** The sensor under-reports object height by a
non-constant amount, currently 2 to 5 mm on this mounting. Every attempt to derive
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

Current values on the device, 2026-09-15:

| Channel | Boundary | Position |
|---|---|---|
| HC1 | 2.6" | **504** |
| HC2 | 3.0" | **521** |
| HC3 | 4.0" | **541** *(estimate, to be measured)* |
| HC4 | 5.0" | **567** *(estimate, to be measured)* |
| HC5 | 6.0" | **592** |
| HC6 | 7.0" | **617** |
| HC7 | 9" | **667** |
| HC8 | 12" | **742** |

All eight channels are in use, giving the nine diameter bands the sorting rules use. All but two
were measured against the real belt. HC3 and HC4 were interpolated from the neighbouring
measurements and should be measured with 4" and 5" blocks.

**Every threshold must sit inside the live window, 448–800.** A threshold inside a blanked region
can never fire, no matter what object is in the beam. If a channel refuses to trigger, check that
before suspecting anything else.

**The tightest gap is HC1→HC2, at 17 mm.** That is the 2.6"/3" boundary, so a mounting error
shows up there first.

**Outlier check.** Subtract each block's height from its reading to get an implied belt position.
All eight should agree within a few millimetres. A block turned the wrong way once gave a reading
20 mm adrift from every other — re-taken flat, it fell into line.

### Part of the array is blanked out

The grid is 800 mm but only about 14 in of it can see the belt. The rest looks at the belt surface
and the machine structure below it, which is **blanked** so the sensor ignores it.

| Field | Range | Covers |
|---|---|---|
| Blanking Field 1 | `10` – `448` | The belt surface and the structure below it |
| Blanking Field 2 | unused | Mode `Inactive` — a spare |

The field has a **Mode** setting, under Operation Mode Configuration → Blanking Field
Configuration, and it must be **`Active`**. Like the Height Control channels, the positions do
nothing while the Mode is Inactive.

**Without blanking the machine would not work at all.** The obstructed beams would hold the
switching signal on permanently and no log would ever be detected.

> **The blanked region must never reach the ends of the array.** The sensor synchronises optically
> on the **first and last beams**, and if **both** are blocked, synchronisation is lost and it
> stops measuring altogether. That is why Field 1 starts at 10 rather than 0, and why the array is
> mounted with both ends in clear air.
>
> The symptom is unmistakable once you have seen it: `Synchronization` **Inactive**, all three
> Measurement Values reading **0 mm**, and `Switching Signal` stuck **Active**. Active with zero
> positions is not "I see an object" — it is "I see no light at all", and **nothing responds
> anywhere, including inside the live zone.** No amount of blanking will fix it; the array has to
> move.

Two consequences worth knowing before they confuse you:

- **A Height Control threshold inside a blanked region can never fire.** It is not a fault in the
  channel. Check the value is above 448.
- **`Lowest Object Position` clamps at about 450** — the blanking edge — for every log, because
  logs rest at about 438, inside the blanked zone. **`Object Height` is therefore wrong for every
  log.** At 1 in steps that makes each block report about the size of the one below it, which
  looks exactly like a stale reading and is not. **Use `Highest Object Position` only.**

Assume changing **Beam Mode wipes the blanking field** as well as the thresholds. Set Beam Mode
first, then blanking, then thresholds.

### The array is mounted cable-down

The grid is inverted from the factory default so the cable exits at the bottom. The manufacturer
supports this explicitly: the device ships configured for cable-outlet-upwards, and for
cable-downwards installation the receiver is configured via IO-Link for the zero-point reference
on the cable side.

That is what `Detection Reference` = **`Reference at Cable Side`** does.

> **The parameter and the physical orientation are a matched pair.** Change one without the
> other and every height reads inverted — a small block reports a *large* position and lights
> nearly every Height Control bit. If you ever see that, check this parameter before suspecting
> the calibration.
>
> **The factory default is the opposite setting.** Anything that restores defaults — including
> pressing Download before Upload — will flip it and invert every reading.

Only the receiver carries the setting; the emitter has no configuration. **Both units must be
mounted the same way up, and at the same height** — with Beam Mode at `Three Beam Crossing`,
mirroring one against the other breaks the crossing geometry rather than merely offsetting it.

**Mounting reference: the printed arrows sit about 1 mm below the top edge of the window
opening.** That position is what keeps the top sync beam in clear air. It is checkable by eye —
if you cannot see the arrow through the opening, the array is too high.


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
- **Kickers 6, 8, 12 and 14 each fire twice**, once on their own number and once as a paired
  partner. A swap within either pair puts a long log on the floor.

> **Never leave an output energised.** Self-test holds the relay on solid, while normal operation
> pulses it for 400 ms. Set the number, observe, set it back to zero. Solenoid coils are
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

Current value: **3.625 ft/s**, the average of four tape-measured logs (2026-09-14). For each log,
divide its tape-measured length by how long it held the beam.

**Re-check this if sorting accuracy drifts over a shift.** A belt that slows under load will
read logs long. Check belt speed before suspecting anything else.

---

## Kick timing

A separate constant compensates for actuation lag — the delay between the output switching on
and the paddle actually striking.

Tune it **after** belt speed is confirmed, and **on a single kick**, never a paired one:

1. Send logs to a single-kick bin (1, 3, 5, 7, 11, 13, or 6 for a big log under 13 ft).
2. Watch where the paddle strikes relative to the log's centre.
3. If it strikes **behind** centre, increase the lead. It is measured in seconds:
   1 ft early is about `0.26` s.

**How to tell this apart from a belt speed error:** if the miss grows the further down the belt
you go, it is belt speed. If every station misses by the same amount, it is kick timing.

Only once single kicks land correctly should you test the paired kicks (6+8 and 12+14), on long
logs.

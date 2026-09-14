# Troubleshooting

**Every entry here has actually happened.** Most of them cost at least a day to diagnose the
first time, and several present as a completely different problem than they are.

---

## Start here

**If the controller is not scanning, nothing else means anything.** Check that first — if the
program's cycle counter is frozen, every other value you are looking at is stale and will send
you in the wrong direction.

---

## Sensor and detection

| Symptom | Cause |
|---|---|
| Beam detection dead, but the height readings are still correct | The sensor is in the wrong Object Identification Mode. Set it back to **LGS (Light Grid Mode)**. The beam signal and the height channels are evaluated independently, so one can die while the other works perfectly |
| Height thresholds typed in but a channel never triggers | That channel's **Mode is set to Inactive**. Each Height Control channel has its own enable, separate from its threshold value |
| All height thresholds suddenly zero | Someone changed **Beam Mode**. It wipes them. Set Beam Mode first, then re-enter the positions |
| `WriteRequest timeout, Index 104` when writing to the sensor | **`Read Process Data` is ticked.** Untick it before any write. This looks like a permissions error and is not |
| Sensor values look like defaults, not reality | You did not press **`Upload`** first. Until you do, you are looking at the description file, not the device |
| Small logs pass through completely unrecorded | The grid is mounted too high. Nothing sees them — no length, no sort, no record. **The worst failure mode in the system** |
| Thin card or paper reads as clear | Expected. The beams are infrared at maximum gain. Test with dense, opaque objects only |
| Red I/O LED on the controller | Has **twice** been an intermittent wire, not a software fault. Check the wiring before anything else |

---

## Sorting behaviour

| Symptom | Cause |
|---|---|
| Logs consistently land in the wrong bin, but the classification looks right | **Relay-to-solenoid wiring.** A swapped pair is invisible from software. Re-run the kicker verification in [MAINTENANCE.md](MAINTENANCE.md) |
| Everything classifies into the shortest length class | **Belt speed setting is too low**, so every length reads short |
| The miss grows the further down the belt | Belt speed |
| Every station misses by the same amount | Kick timing lead |
| Paddles firing at an empty belt on a regular cycle | **The simulator is on.** Check both `SimMode` and `Sim_Auto`. `Sim_Auto` injects fabricated logs even when `SimMode` is off |
| A long pole is dropped on the floor | Both members of the paired kick, 12+14, must fire. Check both relays, and check the pair is not swapped |
| Sorting accuracy drifts over a shift | Belt slowing under load. Logs read long. Check belt speed before suspecting the program |

---

## Controller

| Symptom | Cause |
|---|---|
| **Controller reverts to old code after a power cycle** | The boot application was never updated. **Loading code does not update it** — this is the single most common silent failure. The machine runs correctly right up until it reboots |
| Program does not start at power-up; log shows `denied to start setting` | The physical **run/stop switch is in STOP** |
| Cycle counter frozen, one relay stuck on | **Self-test mode is on.** It holds an output and stops the normal program cycle. Not a crash |
| Cycle counter frozen, self-test off | The controller genuinely is not scanning. Nothing else in the diagnostics means anything until this is fixed |
| A setting was changed but the value does not change | Changing a default requires a full load plus a cold reset. An online change does not re-initialise values — the setting changes while the running value persists, which looks exactly like the edit failed |
| IO-Link reports not ready after loading new code | Toggle the physical run/stop switch. Normal after a load; **does not** happen on a power cycle |
| Cannot connect to the controller | **Do not use Scan Network.** It has failed repeatedly here. Type `192.168.1.3` in directly |

---

## Two things that look like bugs and are not

**The IO-Link master's data is not reachable through normal I/O addressing.** The 750-1657 is a
complex module. Its process data is only available through the WAGO IO-Link library — the missing
I/O Mapping tab is by design, not a configuration error. The symptom worth recognising instantly:
a variable mapped to a direct input address shows one value in a watch window while the program
reading it gets a different one. This was investigated exhaustively and is a dead end.

**Acyclic parameter reads do not work on this controller.** The channel reports zero size in
every configuration tried. Height comes from the sensor's cyclic threshold bits instead. The
WAGO tooling *can* read sensor parameters — that is how calibration works — but the controller
itself cannot. Do not spend time trying to make the controller read sensor parameters directly.

---

## Testing a change without a machine

The control program has a simulator that sweeps a fixed set of logs through the classifier. It is
the regression test: after any change to sorting rules, it must reproduce a known bin
distribution exactly.

**It does not exercise the sensor path at all.** A change that broke the light grid connection
entirely would still pass the sweep cleanly. Always pair it with a physical check — block the
grid by hand and confirm the beam signal follows, and that a known block produces the expected
height bits.

**Set the simulator flags back off and cold-restart before leaving.** See
[OPERATION.md](OPERATION.md).

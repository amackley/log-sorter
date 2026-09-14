# Log Sorter — Machine Documentation

Reference documentation for an automated log sorting conveyor. This repository is the durable
copy: the printed manual at the machine points here, so that the documentation survives lost
binders, replaced laptops and staff turnover.

**This repository holds documentation only.** The control program itself is maintained
separately and privately.

---

## What the machine does

Logs travel down an 88.5 ft belt at roughly 3.83 ft/s. A light grid at the head end measures
each log's **diameter** and **length**. A controller classifies the log into one of 14 bins and
fires a pneumatic kicker at the moment the log's centre reaches that bin's station.

It runs standalone. Once the program is loaded, no PC is needed to operate the machine.

Measurement happens once, at the head of the belt, and everything after that is timing. There is
no sensor confirming a log actually landed where it was sent — so a mechanical problem
downstream shows up as logs in the wrong bin, not as an alarm.

### Hardware

| Item | Part | Notes |
|---|---|---|
| Controller | WAGO 750-8001 Basic Controller | IP `192.168.1.3`, CODESYS 3.5.21.30 |
| Digital input | 750-400 (2 DI) | Present but unused |
| IO-Link master | 750-1657 (4 port) | Light grid on **Port 1** |
| Digital output | 750-1500 (16 DO) | Drives kicker relays, `%QX0.0`–`%QX1.5` |
| End terminal | 750-600 | No I/O function |
| Sensor | Pepperl+Fuchs **LGS8-800-IO/110/115b** | 800 mm array, 96 beams, 8.33 mm pitch |
| Kicker relays | 857-304 | 14 outputs, 2 per station × 7 stations |

Rail order: `750-8001 → 750-400 → 750-1657 → 750-1500 → 750-600`

### Kicker stations

| Station | Distance from sensor | Serves bins |
|---|---|---|
| 1 | 11.500 ft | 1, 2 |
| 2 | 23.833 ft | 3, 4 |
| 3 | 35.500 ft | 5, 6 |
| 4 | 47.500 ft | 7, 8 |
| 5 | 59.500 ft | 9, 10 |
| 6 | 71.833 ft | 11, 12 |
| 7 | 83.667 ft | 13, 14 |

Station N serves bins 2N−1 and 2N — the two sides of one station, sharing one distance.

**Bins 12+14 fire as a pair** for poles of 13 ft and longer, aimed at the midpoint between their
two stations. A single paddle against a 20 ft pole would swing through empty air. Every other bin
fires alone.

---

## The documents

| Document | For | Contents |
|---|---|---|
| [OPERATION.md](OPERATION.md) | Operators | What the bins mean, what the machine does with a log, what the fault indicators mean |
| [MAINTENANCE.md](MAINTENANCE.md) | Maintenance | Re-calibrating the light grid, verifying kickers, belt speed, kick timing |
| [COMMISSIONING.md](COMMISSIONING.md) | Installers | Full first-time setup, in order, with sign-off |
| [TROUBLESHOOTING.md](TROUBLESHOOTING.md) | Everyone | Symptoms and their real causes. Every entry has actually happened |
| [reference/sorting-rules.md](reference/sorting-rules.md) | Everyone | The authoritative bin map and coverage grid |

**If something is behaving strangely, start with [TROUBLESHOOTING.md](TROUBLESHOOTING.md).**
Several of the machine's failure modes look exactly like a different problem than they are, and
each entry in that table cost someone at least a day to work out the first time.

---

## Two things worth knowing before you touch anything

**The sensor calibration has no digital backup.** The eight height thresholds live only in the
light grid's own memory. No configuration file contains them — this has been verified. The table
in [MAINTENANCE.md](MAINTENANCE.md) is the only record, and a single change to the sensor's
Beam Mode setting wipes all of them. If you change those values, update that table.

**Loading new code does not make it survive a reboot.** The controller runs downloaded code
immediately, but powers up into whatever was last written to its flash. The machine can run
perfectly for a week and then revert on the next power cut. See
[TROUBLESHOOTING.md](TROUBLESHOOTING.md).

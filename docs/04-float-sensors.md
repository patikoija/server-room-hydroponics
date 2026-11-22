# 04 — Float sensors (the "bobbers")

Eleven dry-contact float switches: **six in the fill-control bucket** forming a depth ladder, and
**five in the reservoir**. Each one is a reed switch actuated by a floating magnet — cheap,
dumb, and the right sensor for the job.

This page is longer than it looks like it should be, because almost nothing about these is
obvious and two full evenings were lost to things that are now one sentence each.

## Wiring: one wire to the input, one to KCOM

These are **dry contacts** and the board's inputs are optocoupler-isolated, so each float takes:

- **one wire** to its own input terminal, and
- **one wire** to **KCOM**.

The original install had each float wired **across two input terminals**, which completes no
circuit at all. Three floats read nothing through two complete watch windows before anyone
questioned the wiring rather than the sensors.

**KCOM pins 1 and 2 are commoned** — verified, not assumed: floats landed on pin 2 read
identically to floats on pin 1.

## Sense: measured per channel, never inferred

`inverted: true` was removed so that **WET reports as ON**. That is the only sane polarity for a
water sensor: with the inversion in place, every automation would have to read `to: 'off'` to
mean "water arrived", which is correct today and a bug waiting for the next person.
`device_class: moisture` then makes the dashboard show Wet/Dry.

### The floats are not all the same hardware

The two low floats and the hard limit came with the unit. The three depth floats were added
later and may be an entirely different kind of switch.

**So identical pin configuration proves nothing about identical sense.** A normally-open and a
normally-closed float on the same wiring read exactly opposite. Only measurement settles it, per
channel.

### Orientation does not flip the sense

One float is mounted upside down relative to its neighbours — these carry an orientation arrow
and hang down to trip at 90° — and it still reads the same way round. So mounting can follow the
geometry without per-channel inversion.

*(An earlier version of this note said it "still reads True dry / False wet", which described the
raw pin level from before the inversion was dropped. Sitting a few lines under "WET reports as
ON", it read as a flat contradiction and cost a session's worth of doubt about whether the
reported polarity was backwards. The lesson: when you change a polarity, re-read every nearby
sentence that mentions one.)*

## They do not actuate in air

**Buoyancy is the actuator.** Waving one by hand moves nothing. Two watch windows were spent
discovering this. Test them in water, and when a procedure says "lift the float to test the
failsafe", it means lift it *in water*.

## The depth ladder, and the soak that proved it

All six installed; five exercised in a single fill (the hard limit was deliberately not reached).
Every sensor fired in physical order with **zero chatter**:

| Elapsed | Float | Approx. depth | Gap |
|---:|---|---:|---:|
| 23.9 s | low guard | ~3" | — |
| 33.5 s | low second | ~3.4" | +9.6 s |
| 83.0 s | depth mature | ~6" | +49.5 s |
| 147.5 s | depth mid | ~7.5" | +64.5 s |
| 203.7 s | depth day 0 | ~9" | +56.2 s |
| — | **hard limit** | **~13"** | never reached |

### The fill-bucket rise rate is not the system rate

Those gaps average **~38 s/inch** across the 1.5"-spaced floats. The system calibration on file
is **26 s/inch**. Both are correct and they measure different things: this is the **fill bucket**
rising while water simultaneously gravity-feeds out to the plant buckets, which take 30–60 s to
equalise.

Anything that converts time into depth has to know which of the two it means — which is precisely
the argument for reading depth **from these floats** rather than inferring it from a stopwatch.

### Settling

A fill stopped at the ~9" float settles back to **~8.8"**. Small enough that trip-and-stop needs
no top-up loop.

## Validated in both directions

A float whose sense is correct rising and wrong falling is indistinguishable from a correct one
until you watch it fall. So the ladder was re-run after the rewire, in both directions:

| Direction | Observed |
|---|---|
| **Fill** | low guard → low second → depth mature → depth mid, bottom-up, in physical order |
| **Drain** | the same four cleared **top-down** |
| **Pump cutoff** | cut **the same second** the target float closed |

Four of six are now proven in both directions. An earlier soak had proven the *fill* direction
only, and just the two low floats had ever been seen clearing on a drain.

## The one that still is not proven, and why it is the one that matters

**`hard limit` (~13") has never been observed responding to water.** Its only state change in the
recorded history coincides with a restart, not a water event. It currently reads dry, which is
consistent — and consistent is not proven.

It is the float the **fill-overflow failsafe depends on**. Worse, a wrong-sense hard limit is
**invisible to the cross-check**: the "no wet float below a dry one" rule can only catch a
disagreement between floats, and the topmost float reading dry never violates it.

So it has to be proven deliberately — supervised, at the unit, hand on the plug, float lifted in
water.

## Reservoir floats

Five floats at ~6", 9", 12", 15" and 18" in the 60-gallon reservoir, giving a coarse level
readout for "is there enough to flood" and "is it time to top up". These are configured
`inverted: true` — opposite to the fill-bucket ladder — because they are different hardware,
which is the whole point of the per-channel rule above.

## What this buys

Depth is **read**, not timed. The flood cycle still computes a fill duration from the calibration
rate as its primary control, but the floats are what make the failsafes real: an overflow cutoff
that triggers on water arriving somewhere it should not, rather than on a timer expiring.

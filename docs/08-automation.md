# 08 — Automation

The flood cycle, the lights and the fans are run by Home Assistant as config-as-code. No manual
pump timers.

The controller ([03](03-controller.md)) reports water level and pH; Home Assistant owns the
decisions; a **Zooz ZEN20** five-outlet Z-Wave power strip does the actuating, reached through a
Zooz 800 Series USB stick and `zwave-js-ui`.

| Outlet | Load |
|---|---|
| 1 & 2 | grow lights |
| **3** | **fill pump** |
| **4** | **drain pump** |
| 5 | circulation fans |

**The hardware, its wiring, the firmware auto-off timer and the full timing budget are in
[09 — The control chain](09-control-chain.md).** That page is the one to read before plugging a
pump into anything.

## One switch runs everything

Tick **"Buckets Occupied"** and lights plus flooding start on the selected crop's schedule;
untick it at harvest and they stop. Schedules come from a **crop profile table**, so photoperiod
and flood frequency are never set by hand.

### Two axes, deliberately separate

```
input_boolean.hydro_occupied     — are there plants in the buckets at all?
input_select.hydro_growth_stage  — how mature are they? germination → seedling → mature
```

Under the old transplant workflow these collapsed into one flag, because plants only entered the
buckets at transplant and arrived already past seedling. **Direct sowing separates them:** a
bucket is occupied from day 0 but the plant is not mature for ~10 days.

Keeping them apart is also the smaller blast radius — only `hydro_occupied` gates the pumps.

### The vocabulary follows the plant, not the location

Nothing changes location any more; cubes are sown in the bucket they will finish in. So the
language is **germination / seedling / mature** rather than "transplanted" / "established". Worth
renaming deliberately: a schema that describes a workflow you no longer run quietly misleads
everyone who reads it next.

## The cycle

**fill → hold → drain**, repeated several times a day inside the lights-on window.

Fill and drain durations are **computed from depth**, not hard-coded:

```
fill seconds = target depth × (260 s ÷ 10")      # 26 s/inch, 6-bucket calibration
drain seconds = target depth × 28.5 s/inch
```

and the target depth itself steps down with crop age — 8" early, 7" mid, 6" mature. See
[01 — System](01-system.md#flood-depth-versus-the-media-surface).

> **The calibration figures are configuration, not constants.** Going from 6 buckets to 12
> roughly doubles the volume per inch, so a 12-bucket fill on 6-bucket numbers stops at about
> half depth. Re-measure and update them.

## Reboot safety: the floor-flood problem

This is the part worth copying.

Two facts combine badly:

1. **Z-Wave plugs hold their physical state across a restart.** A running fill pump keeps
   running.
2. **`delay:`-based scripts are killed on restart.** The drain step never fires.

So the *physical* phase is tracked in a `input_select` that **persists across reboot** (no
`initial:` value, so the last state is restored). On start-up, a recovery automation force-offs
**both** pumps and, if the last phase was anything but `empty`, **drains to a known-empty state**
before any flooding can resume. Belt and braces: the flood scheduler also refuses to start unless
the phase is `empty`.

The true hardware backstop for "the automation server is fully down while the fill pump is on" is
**the ZEN20's own auto-off timer**, set on the strip itself in firmware — independent of anything
running on a server. It is set to 6 minutes, and
[why it is 6 and not 7 or 8](09-control-chain.md#the-timing-budget-and-how-it-was-got-wrong-twice)
is the most useful safety lesson in this repository.

## Failsafes

Four, each independent of the cycle script:

| | |
|---|---|
| **Fill-overflow cutoff** | triggers on the topmost float going wet — rack-flood protection |
| **Drain-runaway cutoff** | bounds how long the drain pump may run |
| **Both-pumps interlock** | the two pumps may never be on together |
| **Master enable** | one toggle halts all automatic cycling |

A trip **latches a fault flag**, which stops automatic flooding until it is cleared by hand.
Lights and the manual button are deliberately unaffected — a water fault should not put the crop
in the dark.

Beyond the software: a [hard-wired relay interlock](02-bom.md#the-hard-wired-interlock-and-why-the-intuitive-wiring-is-wrong)
in the fill pump's mains feed, held closed by a float, wired so that every failure mode lands on
pump-off.

## It ships disabled

`hydro_occupied` defaults **off**, so deploying the configuration actuates nothing. The failsafes
are always armed regardless. Entity IDs for the real outlets started as obvious placeholders, so
that merging the configuration before commissioning could not fire a pump.

## Fans run continuously, not on the light schedule

Fan start is gated on **days since sow**, not on growth stage — the two came apart once the
workflow stopped having a transplant step, and gating on the wrong one means the fans wait for a
stage change that no longer happens on a fixed schedule.

Run them 24/7. There is no benefit to still air at night, and the first two batches — dead air
throughout — produced damping-off and tipburn respectively. See
[07 — Lighting](07-lighting.md#tipburn-it-is-a-transport-failure-not-a-shortage).

## Cross-checks on the sensors

- **"Floats disagree"** — flags any wet float sitting *below* a dry one, which is physically
  impossible and therefore evidence of a wiring or sense fault. Note its blind spot: the
  **topmost** float reading dry never violates the rule, so the overflow float is exactly the one
  this check cannot validate.
- **"Floats healthy"** — goes false when the controller stops reporting. It works; **nothing is
  wired to notify on it**, which is a known gap and the one I would close first.
- **"pH dosing permitted"** — ANDs reading sanity, calibration staleness and probe slope, failing
  closed. See [05 — pH](05-ph.md#trusting-the-reading-before-acting-on-it).

## A grow log, because the automation cannot remember why

Sow dates, crop, site and observations go in a phone-friendly journal alongside the parts
tracker. The automation knows the current stage; it has no idea that batch 2 tipburned or that
the fans were never wired. Those are the facts that actually change the next decision.

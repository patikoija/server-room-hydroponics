# 09 — The control chain: Z-Wave stick, zwave-js, and the ZEN20 strip

Nothing in this system is a "smart plug with an app". The actuation path is five hops, and each
one has a failure mode worth knowing before you build on it.

```
   Zooz 800 Series Z-Wave Stick (USB)
        │  /dev/serial/by-id/… → /dev/ttyACM0
        ▼
   zwave-js-ui  (container)          ← owns the mesh, holds the node database
        │  websocket, by container name
        ▼
   Home Assistant                    ← owns the decisions (see 08 — Automation)
        │  Z-Wave commands
        ▼
   Zooz ZEN20 power strip            ← 5 independently switched outlets
        │  mains
        ▼
   fill pump · drain pump · 2 × light bar · fans
```

---

## The radio: Zooz 800 Series Z-Wave Stick

A USB stick, not a hub. It is the radio only — the mesh logic lives in software.

### Bind it by `/dev/serial/by-id`, never `/dev/ttyACM0`

```yaml
devices:
  - /dev/serial/by-id/usb-Zooz_800_Z-Wave_Stick_<serial>-if00:/dev/ttyACM0
```

`ttyACM` numbering is whatever the kernel happened to enumerate, and it **moves** when another
CDC device appears or the stick is re-plugged. The `by-id` path is stable across reboots and
re-plugs.

Note the mapping target: the by-id path is presented **as `/dev/ttyACM0` inside the container**,
so zwave-js's stored `settings.json` needs no edit — the device path it has always used still
resolves. You get stability on the host side without touching application state.

### Exactly one machine can hold the stick

A USB device belongs to one guest. There is no configuration in which two copies of zwave-js
share one radio, which makes the stick the physical bottleneck in any migration: you stop the old
instance, **move the stick**, then start the new one. It cannot be rehearsed in parallel.

### The node database is the irreplaceable part

```yaml
volumes:
  - homeassistant_zwave:/usr/src/app/store
```

That volume is the Z-Wave node database — tens of megabytes of pairings, node IDs, routes and
per-device parameters. **Losing it means re-pairing every device by hand**, including anything
mounted behind furniture. It matters more than the container, which is one `docker pull` away.

Back it up like data, not like config.

### A second Home Assistant on an empty volume is not idle, it is harmful

Worth stating because the instinct for other services is "boot it empty to prove the declaration,
then seed it". For Home Assistant that is exactly backwards: a second instance on an empty config
volume connects to the shared MQTT broker, runs first-run setup, and starts **publishing
alongside the real one**.

---

## zwave-js-ui

Runs as its own container (`zwavejs/zwave-js-ui`), separate from Home Assistant, which is the
right split: HA restarts for configuration changes far more often than the Z-Wave mesh should.

Home Assistant reaches it over a websocket **by container name on the shared network**, so the
published ports are for humans and external tools rather than a dependency.

> **One failure mode worth recording.** Copying another host's port list verbatim caused a
> collision, and Docker's behaviour was worse than a clear error: the container came up
> **"running" with no published ports and no network at all**. Home Assistant's only symptom was
> `Z-Wave JS not ready: Failed to connect`, which points at the radio, the stick, the mesh —
> anywhere but a port conflict.

---

## The ZEN20 power strip

**Zooz ZEN20** — five individually switched outlets plus always-on USB. Each outlet appears in
Home Assistant as its own switch entity.

| Outlet | Load | Notes |
|---|---|---|
| 1 | grow light bar (row 1) | |
| 2 | grow light bar (row 2) | |
| **3** | **fill pump** | also fed through the [hard-wired relay interlock](02-bom.md#the-hard-wired-interlock-and-why-the-intuitive-wiring-is-wrong) |
| **4** | **drain pump** | |
| 5 | circulation fans | runs continuously; has its own opt-out toggle |

### Its firmware auto-off timer is the real backstop

This is the part that makes the strip more than a convenience, and the reason to choose one with
this feature.

Every software guard in [08 — Automation](08-automation.md) runs *on a server*. If that server is
fully down while the fill pump is energised, none of them exist. **The ZEN20's own auto-off timer
runs on the strip**, in firmware, and cuts the outlet regardless of what is or is not reachable.

Set to **6 minutes** on the fill pump outlet. The device only accepts whole minutes, which is why
this value is not derived from the second-based software bounds — the granularity would round away
any precision you pretended to have.

---

## The timing budget, and how it was got wrong twice

Four bounds sit on top of each other. They must be ordered, and the ordering is the whole design:

```
 computed fill       ~313–344 s     what a healthy fill actually takes
 software ceiling      340 s        blind-fill guard, when no float is trusted
 HARDWARE auto-off     360 s        the ZEN20 cuts its own outlet
 OVERFLOW LINE         389 s        water leaves the buckets
```

### First mistake: margin in the wrong direction

The auto-off started at **7 minutes**, on the reasoning that more margin above the software
ceiling is safer. Measured at the time:

| Auto-off | Pumping time | Reached depth | Past the 13" overflow |
|---|---:|---:|---|
| 6 min | 360 s | 13.8" | ~22 s, roughly 2–4 gallons |
| 7 min | 420 s | 16.2" | **~82 s, roughly 10–15 gallons** |

Seven bought interference-immunity that six already guaranteed, and paid for it in gallons on the
floor. Revised to six.

> **"Add margin" has a direction.** On a bound whose job is to fire *before* something spills,
> extra margin is not extra safety — it is extra spill.

### Second mistake: letting the software guard set the hardware bound

Later it was raised to **8 minutes** so that a software guard would always fire before the relay
cut itself. That optimised the wrong variable. Recomputed against the **measured** fill rate:

| Auto-off | Reached depth | vs the 13" overflow line |
|---|---:|---|
| **6 min** | 12.0" | safe, below the line |
| 7 min | 14.0" | 1.0" past |
| 8 min | 16.0" | **3.0" past** |

Eight minutes put the last backstop three inches beyond the point where the buckets spill.

**The hardware bound is governed by the overflow line, not by the software.** It is the last thing
between a stuck relay and water on the floor. If a software guard cannot fire before 360 s, then
**the guard's bound is what comes down** — and that ordering is now enforced by a test rather than
by a comment, so a future fill-time recalibration cannot push a safety bound outward again.

---

## Re-measuring the fill rate, and why the old figure caused trouble

Three consecutive cycles, pump-on to the 11" float going wet:

```
324 s · 328 s · 334 s        mean 329 s, spread 10 s
```

The original figure was "260 s fills to 10" ⇒ **26 s/inch**". Measured at 11" it is
**29.9 s/inch** — the expectation was ~13% optimistic, and the computed fill time read 286 s
against a real 329 s, **43 s short**.

A fill that routinely overruns its own expected duration makes every downstream judgement wrong:
the watchdog budget, the blind-fill timeout, and an operator reading "fill timeout" and going to
look for a fault in the floats that is not there.

> Both numbers appear in this repository: 26 s/inch is the original 10" calibration, 29.9 s/inch
> is the measured rate to 11". Where a safety bound is computed, **the measured rate is the one
> used** — and the slower real rate makes the 6-minute auto-off *safer* than the earlier estimate
> assumed, which is why there was never a reason to move it.

Drain, for completeness: 285 s to 10" ⇒ 28.5 s/inch calibration; measured 295 s.

---

## Commissioning order

1. **Stick in, bound by `by-id`.** Confirm the container sees it before touching anything else.
2. **Pair the ZEN20** in zwave-js-ui. Confirm all five outlets appear as separate entities in
   Home Assistant.
3. **Set the auto-off parameter on the fill-pump outlet** — before a pump is ever plugged in.
   This is the bound that does not depend on your software being right.
4. **Label the physical outlets.** Outlet 3 and outlet 4 doing each other's job is a drain pump
   trying to fill and a fill pump trying to drain, and it looks like a plumbing fault.
5. **Wire the float interlock** into outlet 3's feed. Test it by lifting the float **in water**.
6. **Then** put the real entity IDs into the automation and enable it — not before. Until this
   point the configuration ships with the cycle disabled and placeholder entity IDs precisely so
   that deploying it cannot actuate a pump.
7. **Prove the overflow float** responds to water, supervised, hand on the plug. It is the one
   float the automated cross-checks structurally cannot validate — see
   [04 — Float sensors](04-float-sensors.md#the-one-that-still-is-not-proven-and-why-it-is-the-one-that-matters).

## What is still missing from this chain

**Nothing alarms when the Z-Wave stick or the controller goes away.** The health sensors exist
and go false correctly; no automation triggers on them. A detector wired to no buzzer.

# 12 — pH calibration

Three-point calibration of the EZO-pH circuit. The circuit stores the calibration itself, so this
is a procedure you run against the hardware, not a setting in software.

## Order matters: MID → LOW → HIGH

> **MID 7.0 → LOW 4.0 → HIGH 10.0**

**MID clears any prior calibration and anchors the offset**, so it has to be first. Doing LOW
first and MID second silently discards the LOW point.

## For each point

1. **Rinse the probe** with distilled or filtered water, then submerge it in the matching buffer.
   Do not reuse a rinse that has been sitting in a previous buffer — buffer cross-contamination is
   the most common way to produce a calibration that looks fine and reads wrong.
2. **Wait for the reading to settle** — stable to within a few hundredths across two consecutive
   updates. The sensor publishes once every 60 s, so a repeated value *within* the same minute is
   not evidence of stability.
3. **Before calibration it will read noticeably off, and that is the whole point.** Typical:
   ~0.4–0.6 high in 7.0 and 4.0, ~0.3–0.4 low in 10.0.
4. **Trigger the matching calibration point.**
5. **Confirm the reading snaps to the buffer value** on the next update.

## A real run

| Point | Buffer | Settled reading before | After |
|---|---|---|---|
| MID | 7.0 | 7.58 | **7.00** |
| LOW | 4.0 | 4.39 | **3.99** |
| HIGH | 10.0 | 9.64 | **10.00** |

Each snap appeared on the first 60 s update after the press. Rinse and return the probe to the
reservoir afterwards.

If a point does **not** snap, do not press it again hoping — go back and check that the reading
had actually settled, and that the buffer is the one on the label.

## How often: about once a year, not monthly

This is the single most commonly over-done maintenance task on these probes.

A hydroponic reservoir is a **mild** environment — weak acids and bases — so drift is slow.
**Roughly once a year for the first two years, then every six months.**

Consumer-grade guidance of "every 30–60 days" is **about 12× too frequent** for a lab-grade probe,
and over-calibrating is not harmless: every calibration is a fresh opportunity to introduce
technique error — an unsettled reading, a contaminated buffer — which later shows up as a slope
figure nobody can explain.

**Recalibrate off-schedule only when** a reading looks wrong against a fresh buffer, or when the
slope reading says the probe is out of band.

## The trust ladder: three checks, not one

A drifted probe reports a plausible number forever. So the reading is qualified by three
independent checks, ANDed, all failing closed:

| Check | What it catches | What it misses |
|---|---|---|
| **Reading sanity** | 0 and 14, which per the datasheet are *electrical noise*, not readings | a plausible-but-wrong value |
| **Calibration staleness** | nobody has calibrated this in a very long time | a probe that is dying despite a fresh calibration |
| **Probe slope** | exactly that — the acid and base slope percentages the circuit reports | nothing else does this |

**Slope is the one that earns its keep.** A healthy probe reads ~95–105% on both acid and base
slopes. Drift shows up there long before the reading looks obviously wrong, and a
days-since-calibration counter is structurally blind to it.

The circuit reports both the calibration point count and the slope on request, so neither of these
needs a hand-maintained timestamp somewhere else — which is good, because a hand-maintained
timestamp is a thing that goes stale silently.

## When the probe really is tired

**The glass is not replaceable** — the body is a sealed assembly of membrane, internal wire and
electrolyte, so a cracked bulb means a new probe.

Short of that, **try reconditioning before buying.** Atlas sell a reconditioning kit (the
procedure opens with a ~15 s dip in 0.1N HCl). Two weeks in a Masterblend solution can deposit a
mineral film that reads exactly like a degraded probe and simply cleans off.

Replacement is **the probe alone, not another kit** — the circuit, the isolated carrier and the
buffers are all reusable. On a lab-grade double-junction probe that is ~$100 against a ~2.5 year
submerged life: about **$40/year**, and the industrial probe is twice that for a design aimed at a
different problem. See [05 — pH](05-ph.md#probe-care-and-the-real-running-cost).

## Buffer handling

- Buy more buffer than you think you need; it is the recurring consumable alongside the probe.
- Single-use sachets avoid the contamination problem entirely and are worth it at this cadence.
- Never pour used buffer back into the bottle.
- Storage solution is **not** a buffer and is not interchangeable with one.

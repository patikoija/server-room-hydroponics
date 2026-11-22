# 11 — Sowing, germination and the dry stage

The system direct-sows: the rockwool cube is seated in the bucket it will finish in, and the
seedling stage happens in place. No transplant.

That removes transplant shock, root disturbance, and a handling step that can introduce
contamination into a commissioned bucket. It also introduces one problem that has to be solved
before anything else works.

## The load-bearing problem: the flood does not reach a surface-seated cube

Flood depth is the bucket's engineered fill line, which **by design sits below the media
surface**, and hydroton has essentially **zero capillary rise**.

So a 1.5" cube resting on top of the media sits **3–5" above the highest water it will ever see**
(exact gap depends on the stage —
[see the depth ramp](01-system.md#flood-depth-versus-the-media-surface)) and dries out in about a
day. Worse: roots emerging from the cube bottom would have to cross that same 3–5" of **dry**
hydroton to reach the flood zone. That is a desiccation gap, not a delay.

### The resolution, and the two alternatives that were rejected

**Seat the cube in a well dug into the hydroton, base at or just above the flood peak.** The whole
direct-sow approach rests on this one decision.

Rejected:

- **Hand top-water daily for 10–14 days.** It works, and it defeats the point of having
  automation. On a system in a room you do not visit daily, it is also the step that gets missed.
- **Raise the flood depth during germination.** **No.** That fill line is the engineered overflow,
  and overflow next to a server rack is the one catastrophic failure mode in this build. Never
  trade the overflow margin for a convenience.

### How to find the right depth — a free experiment

Park a spare conditioned cube at two or three depths through several flood cycles on a batch
that is already running, and see which stays **moist but not sodden**.

A cube whose base sits *at* the flood line stays saturated between floods, and permanently soggy
rockwool is how you get damping-off. There is a window between "dries out" and "never dries", and
it is narrower than it sounds.

## The process

1. **Condition and break the sheet wet.** Soak at pH 5.5 first, then separate — sheets are
   pre-scored and part cleanly. **Dry rockwool dust is a respiratory irritant**; do not break it
   dry in an occupied room.
2. **One cube per bucket**, seated per the depth finding above, with hydroton packed up to the
   cube's shoulders. A mature head on a bare 1.5" cube flops over.
3. **Sow 2 seeds per cube, uncovered.** Lettuce is **positively photoblastic** — press into a
   shallow dibble, do not bury. Thin to one.
4. **Dome each cube.** An inverted clear deli cup or a cut bottle is a per-bucket dome. Vent
   daily; pull at first true leaf.
5. **Expect one hand top-up around day 5–6.** Do not count on the cube carrying the whole dry
   stage unassisted — **moisture, not temperature, is the binding constraint** here.
6. **Pumps off until roots show at the cube bottom** (~day 5–10). Trigger the first flood on the
   **observation**, not the calendar.
7. **Keep 3–4 backup cubes on a seedling shelf** purely for gap-fill. One cube per bucket is a
   lottery, and a failed cube otherwise means an empty bucket for a full ~55-day run.

> **The "thin to one" step has never actually been exercised on this build.** It is in the process
> and it keeps not happening, which is worth knowing if you are reading this as a worked example
> rather than a plan.

## Do not mix the reservoir on day 0

**Plain pH-5.8 water during the dry stage.** Ten days of warm, still, lit nutrient solution with
no plants drawing on it is a pythium and algae starter culture.

Dose to **seedling strength (~EC 0.6–0.8)** before the first flood, then ramp to EC 1.2 as roots
establish.

This is also the right moment for the *Bacillus* inoculant — during the dry stage rather than at
first mix, so it colonises the root zone before anything else does.

## Dry-stage lighting: three photoperiods, continuous, never pulsed

| Stage | Starts on | Window | Hours | DLI at ~250 µmol/m²/s |
|---|---|---|---:|---:|
| **germination** | sow (day 0) | 16:00–20:00 | 4 | ~3.6 — irrelevant, there is no leaf area |
| **seedling** | emergence (~day 3) | 10:00–20:00 | 10 | ~9.0 |
| **mature** | roots down / flooding starts | 06:00–20:00 | 14 | ~12.6 |

Two structural choices do the work:

### All three windows end at the same clock time

Only dawn walks earlier as the plant advances; **dusk never moves**. Dark-onset is the stronger
circadian anchor, so the plant is not re-entraining at every stage change. In the crop profile
table the rule is mechanical — `seedling_on = off − 10 h`, `germination_on = off − 4 h` — so it
holds for every crop rather than just for lettuce.

### The germination window is short to limit evaporation, not heat

During germination the optimisation target is **inverted**: DLI is irrelevant because there is no
leaf area, so the only things light does are add heat and drive evaporation.

Evaporation is the one that matters. The cube has **no water supply for ~10 days** and the room
runs 47–52% RH. Going from a 14 h photoperiod to 4 h is a large cut in evaporative load on an
unwatered cube. That, not temperature, is the real argument.

Placing those four hours in the room's coolest window is a secondary, nearly free bonus.

> An earlier version of this schedule had *two* photoperiods, put lettuce thermoinhibition at the
> wrong temperature, and claimed pulsed light "buys nothing thermodynamically". All three were
> wrong. The conclusion *not to pulse* survived — for a different reason — and the schedule gained
> a middle stage.

## Crop choice follows the room

The room's temperature is set by a server rack, so it is measured rather than chosen, and the crop
list follows it:

- **Heat-tolerant lettuce** ('Jericho' romaine, 'Muir' green leaf) — bred for hot climates and
  will not bolt in this range. Forgiving.
- **Bok choy / pak choi** — ~30 days, likes 70–80 °F. The quickest early win.
- **Genovese basil** — thrives at 75–85 °F and will outperform everything else here. Two sites is
  more basil than a household can use.

If the room ever holds genuinely cool, the cool-season menu opens up; if it rebounds warm, bias
further toward basil and the heat-tolerant lettuces. The point is that the measurement comes
first.

## Ambient is measured, not estimated

An hourly collector logs room temperature and humidity, which is what makes the crop decision and
the germination-window placement arguable at all.

One caution recorded the hard way: a **rack-intake proxy is not canopy truth**. The one live
cross-check available put the proxy ~1.6 °F warm against a sensor at canopy height. Useful for
trend and for hour-of-day shape; not a substitute for measuring where the plants actually are.

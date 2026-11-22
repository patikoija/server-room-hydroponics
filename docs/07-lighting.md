# 07 — Lighting

**Two Spider Farmer SF600 linear bars, one per row of three buckets.**

| Per bar | |
|---|---|
| Power | 74 W |
| PPF | 168 µmol/s |
| Efficacy | 2.3 µmol/J |
| Dimensions | 1000 × 150 mm |
| Beam | 120° |
| Vendor optimal coverage | 12" × 36" |
| Vendor recommended hang | 6–12" |

**Total photon budget: 336 µmol/s for the whole system.** Every intensity claim has to fit
inside that number.

## The fixture identity matters more than it sounds

This build was documented as having an **SF-2000** for a long time. It does not, and the
difference is roughly **3× in light output**. Every intensity conclusion computed from the wrong
fixture was wrong, and in the same direction: it made the rig look over-lit, which pointed the
tipburn diagnosis at the wrong cause for months.

If you are reading someone else's grow write-up, check the fixture model against a photo before
trusting a single PPFD number in it.

## Schedule and targets

| | |
|---|---|
| Photoperiod, mature lettuce | **06:00–20:00 = 14 h** |
| PPFD target at canopy | **200–300 µmol/m²/s** |
| Resulting DLI | **~15–17 mol/m²/day** |

Leafy greens never need flowering intensity. The DLI figure is the standard lettuce number.

## What 336 µmol/s can actually deliver

Two rows of three buckets is roughly **36" × 24" = 0.56 m²**. If every photon landed on canopy
and none escaped sideways, that is 336 / 0.56 ≈ **600 µmol/m²/s**. Real capture off a 120° bar at
close range is more like 45–70%, so the honest band is:

| Capture | PPFD at canopy | DLI at 14 h |
|---|---:|---:|
| Pessimistic (45%) | ~270 | **13.6** |
| Midpoint | ~330 | **16.6** |
| Optimistic (70%) | ~420 | **21.2** |

Mature lettuce wants 12–17 mol/m²/day and the tipburn threshold in the literature is **sustained
>17 over three consecutive days**.

**The band straddles the threshold.** It is neither comfortably safe nor obviously the culprit —
which is exactly why it has to be measured rather than argued. A phone-based meter (Photone, with
white-LED calibration) is enough to settle it.

## At 6" the problem is uniformity, not intensity

The SF600 is a **150 mm-wide bar** with vendor-stated optimal coverage of 12" × 36". The length
matches a row of three buckets almost exactly. **The width does not.** At 6" a 6"-wide bar lays
down a bright *stripe*, not a field: leaf area at the bucket edges sits in a much dimmer zone than
leaf area directly under the bar. Plants respond by reaching sideways — which fits one batch's
wide, sprawling heads with oversized outer leaves rather well.

**Standing guidance: hold a roughly constant ~10" bar-to-canopy gap by raising the bars as the
crop grows**, rather than setting a height once and forgetting it. That is mid-range for the
vendor spec, wide enough for the bar to actually cover its row, and the rig supports it — there is
32" of travel available and early batches used 15" of it, fixed. In practice: start near 11" above
the bucket tops and end near 19".

If a measurement comes back above 17 mol/m²/day, **trim the photoperiod rather than raising
further.** With a 74 W bar, height bought past ~12" costs more intensity than you want to give up.

## Tipburn: it is a transport failure, not a shortage

Brown, dried, crisped margins on the **youngest leaves at the centre of the head** — not the
outer leaves. *Which* leaves are affected is the whole diagnosis.

Tipburn is a **calcium-transport failure**, so adding cal-mag to the reservoir does nothing for
it. Calcium moves only in the xylem, carried by transpiration. The inner head leaves sit enclosed
in still air, barely transpire, and get shorted.

Ranked by how well each driver fits this system:

1. **No air movement around the head — the leading suspect.** The fans were mounted and never
   wired, so the first two batches both ran in dead air. **Batch 1 died of damping-off; batch 2
   showed tipburn** — the two classic still-air diseases. Two of two is not a coincidence worth
   explaining away. It is also the cheapest fix and the only one needing no measurement first.
2. **Growth rate outrunning calcium delivery** (DLI above ~17). Plausible but unresolved: the
   estimated band straddles the threshold and nobody has measured it. Not established.
3. **Root-zone oxygen problems.** Least supported; no observation points yet.

An earlier version of this analysis blamed excessive light *first* — a ranking that came straight
from mis-identifying the fixture. With the real 336 µmol/s budget, airflow moves to the front.

## Seedlings

Seedlings do not go under the main bars; at close range they are too hot. A pair of cheap 24"
full-spectrum strips over the propagation trays is the right tool, and if the main bars are ever
upgraded the SF600s become the seedling-station light.

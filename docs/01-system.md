# 01 — System and physical layout

## The kit

**GrowAce AirCube Multi-Flow, 12-site ebb-and-flow, 60-gallon reservoir.** Twelve 5-gallon
flex-buckets, one manifold, one reservoir, recirculating. A fill pump pushes nutrient solution
from the reservoir into a fill-control bucket at the head of the system; the buckets flood by
gravity through the shared manifold; a drain pump returns it.

**Six of the twelve sites are currently plumbed.** Floor space, not capability — the other six
are on hand and join the same manifold when there is room. Every timing figure in this repo was
measured in the 6-bucket state and **does not carry over to 12**: doubling the sites roughly
doubles the volume per inch of depth, so the fill rate goes from ~26 s/inch to ~52 s/inch. Left
unchanged, a 12-bucket fill would stop at roughly half depth.

## Every bucket must sit at the same elevation

Flood depth in an ebb-and-flow system is gravity-balanced across the manifold. Putting buckets on
shelves at different heights breaks the hydraulics — the low buckets flood deep and the high ones
barely wet. **Floor placement, single tier, no exceptions.**

## Why 3 ft of head above the buckets is not a flood hazard

This is worth stating plainly because reasoning about it from head pressure alone gives the wrong
answer, and did once.

The reservoir, the fill bucket and the plant buckets all stand on the **same floor**. Sixty
gallons puts the reservoir's water surface roughly **4 ft up** — about **3 ft above the top of a
12" bucket**. The pumps exist to fight that head. It does not create a siphon risk, because both
paths are deliberately broken by an air gap:

- **Supply:** an anti-siphon valve on the hose, *and* the hose stops short of the fill bucket's
  bottom so water **free-falls ~11"** into it. (That air gap is also why filling is so turbulent.)
- **Drain:** the hose terminates at the **top** of the reservoir and free-falls the rest of the
  way in. Same gap, same reason.

An air gap breaks the water column, so gravity cannot drive flow between reservoir and buckets
with the pumps off, whatever the head is. A passing check valve cannot put 60 gallons on the
floor.

> **If you ever replace a hose, preserve both air gaps.** They are load-bearing, not incidental,
> and they are the kind of detail a tidy re-plumb removes without noticing.

## Dimensions

| | |
|---|---|
| Bucket | 12" tall, tapered — 12" × 12" at the rim, ~11" × 11" at the base |
| Media surface below the rim (freeboard) | 1" → media sits at **11"** off the floor |
| Light bar bottom above the rim | 15" → **27"** off the floor, **16"** above the media |
| Arrangement | **2 rows × 3 buckets**, one light bar spanning each row |
| Head of row 1 | fill-control bucket |
| Head of row 2 | 60-gallon reservoir |
| Fans | one per row at the head end, blowing along the row axis |
| Heat source | server rack to one side, convecting across the rows |

Footprint options for the full twelve are a 3×4 array (~4 × 4 ft) or 2×6 (~2 × 6 ft). A flood
tray sits under the whole array for spill containment.

## Flood depth versus the media surface

The flood depth steps **down** with crop age — deep early to wet the propagation cube, shallow
later so the roots get oxygen. With the media surface at 11" off the floor:

| Stage | Flood depth | Gap from flood peak to media surface |
|---|---:|---:|
| Early (< 14 days from first flood) | 8" | **3"** |
| Mid (14–21 days) | 7" | **4"** |
| Mature (> 21 days) | 6" | **5"** |

### The taper makes depth non-linear in volume, and it does not matter

The buckets are tapered, so a fill is a frustum rather than a prism: an inch near the bottom
holds less than an inch near the top. The controller computes fill time as
`target depth × (260 s ÷ 10")` — a **linear** model.

Worked through: filling to 6" needs **58.2%** of the 10" calibration volume while the linear
model assumes 60%, so it runs **3% long — about 4.6 s, or 0.17" of extra depth** at the mature
target. Against a 5" gap to the media surface and an engineered overflow above that, this is
noise.

It is recorded because a 3% bias is exactly the sort of thing someone later rediscovers, mistakes
for a bug, and "fixes". It would start to matter if the target depth ever went much below 6",
where the taper's effect grows.

### The 10" calibration figure is not an operating depth

`260 s @ 10"` exists to derive 26 s/inch. **Nothing ever floods to 10".** Reading it as the flood
height makes the gap to the media look like 1" instead of 3–5", which is an easy and alarming
mistake to make from the variable names alone.

## Airflow

Fans sit at the head of each row and blow along the row axis, with the rack's convected heat
crossing the rows from one side. Batch 1 died of damping-off and batch 2 had tipburn — the two
classic still-air problems — so air movement is not optional decoration here. Run the fans
continuously rather than on the light schedule; there is no benefit to still air at night and
the two failures above both happened under it.

## Humidity

Sixty gallons of open water plus transpiration will nudge room humidity up a few percent. In a
server room that is worth knowing before you add it, not after.

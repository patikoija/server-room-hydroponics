# 02 — Bill of materials

Thirty-five line items, tracked in tiers: Tier 1 is what the system needs to run, Tier 2 is
deferred until a measurement says it is needed. Prices are what was paid or quoted at the time
and will have moved.

## System

| Item | Price | Status |
|---|---|---|
| GrowAce AirCube Multi-Flow 12-site ebb-and-flow, 60-gal reservoir | — | **owned** |

Twelve 5-gallon flex-bucket sites plus a 60-gallon control bucket / reservoir, recirculating.

## Tier 1 — consumables

| Item | Price | Notes |
|---|---:|---|
| Hydroton expanded clay pebbles, **original** size (8–16 mm), 50 L × 2 | $35 | **Original, not giant.** Original packs snugly around a 1.5" cube with no voids; giant leaves gaps and the cubes tilt or sink during flood cycles. Rinse thoroughly before first use — the dust is relentless. Not vermiculite (washes away), not soil (clogs the manifold). Reusable across grows. |
| Grodan 1.5" rockwool starter cubes (7×7 sheets of 49) | $22 | Sold as sheets, not 50-packs. Sow 2 seeds per cube, thin to one. When roots poke out the bottom, the whole cube goes into the hydroton. |
| Masterblend combo kit — 4-18-38 + calcium nitrate 15.5-0-0 + Epsom salt, 1 lb each | $38 | The cheap-and-effective standard for leafy greens. 1 lb of each lasts roughly a year at this volume. See [06 — Nutrients](06-nutrients.md). |

## Tier 1 — instruments

| Item | Price | Notes |
|---|---:|---|
| Apera PH20 waterproof pH pocket tester | $50 | The single most important measurement. A $15 pen drifts weekly; the PH20 is the cheapest tier that actually holds calibration. |
| Apera PC60 (AI316) 5-in-1 — pH / EC / TDS / salinity / temp | ~$20 | **The meter actually on hand**, and it supersedes the TDS-only pen originally listed. Reads **EC directly**, which removes the ppm-scale ambiguity entirely. Calibrate EC against 1413 µS/cm and pH against 4.00/7.00 buffers before trusting anything. |
| General Hydroponics pH Control Kit (Up + Down + indicator) | $25 | Tap water starts around 7.5, so it is mostly pH Down. |

## Tier 1 — light

| Item | Price | Status |
|---|---:|---|
| Spider Farmer SF600 LED bar × 2 | — | **owned** |
| 7-day programmable outlet timer | $15 | Superseded in practice — the lights run on an automation schedule instead. |

See [07 — Lighting](07-lighting.md) for why the fixture identity matters: this build was
documented as having an SF-2000 for a long time, which is a 3× error in light output.

## Tier 1 — safety

| Item | Price | Notes |
|---|---:|---|
| Flood tray sized to the bucket footprint | $80 | **Non-negotiable** under 60 gallons in a server room. One stuck fitting is otherwise catastrophic. |
| Power relay, SPDT, **12 VDC coil**, ≥10 A / 250 VAC contacts | $17 | See the hard-wired interlock below. |
| 12 V DC wall adapter, ≥500 mA | $8 | Powers the relay coil only. |
| Sealed plastic junction box + strain-relief glands | $15 | Mains and water in one room. Mount **above** any possible water line. |
| Small parts: flyback diodes, terminal blocks, 22 AWG 2-conductor cable, short grounded extension cord | $20 | The extension cord is sacrificial — cut it and land both ends in the box. |

### The hard-wired interlock, and why the intuitive wiring is wrong

A software failsafe runs in software. The independent one is a relay in the fill pump's mains
feed, held closed by a float.

- **The coil must be 12 VDC, never 120 VAC.** A mains coil puts line voltage on float wiring that
  sits in nutrient solution.
- **Wire the float normally-closed** (opens as water rises) **to the relay's NO contact.** Then
  coil-energised is the *normal* state, and every failure mode — lost coil power, broken float
  wire, pulled plug — lands on **pump off**. Wiring it the obvious way round fails the wrong way.
- **The flyback diode is not optional.** Inductive kickback welds reed contacts within a few
  hundred cycles, and a welded reed fails **closed** — precisely the direction that defeats the
  interlock.
- Run the 22 AWG float cable separately from the mains conductors.

## Tier 1 — seeds and propagation

| Item | Price | Notes |
|---|---:|---|
| Lettuce 'Jericho' — heat-tolerant romaine | $5 | Bred for hot climates; will not bolt at server-room temperatures. Forgiving first variety. |
| Lettuce 'Muir' — heat-tolerant green leaf | $5 | Pairs with Jericho for variety in one harvest. |
| Bok choy / pak choi | $5 | ~30 days, likes 70–80 °F. Quick early win. |
| Genovese basil | $5 | Thrives at 75–85 °F — outperforms everything else in this temperature range. Two sites is more basil than a household can use. |
| 1020 propagation trays + humidity domes, 5 sets | $30 | Staggered germination on existing shelving. |
| 24" T5/LED seedling strips × 2 | $60 | Seedlings do not need the main bars, which are too hot at close range. |

## Tier 2 — reservoir monitoring

| Item | Price | Notes |
|---|---:|---|
| **Atlas pH Kit** — EZO-pH circuit + Gen 3 lab probe + **isolated** carrier + buffers + storage solution | **$174.99** | **Bought as the kit, and the kit is the right buy.** See below. |
| Atlas lab-grade pH probe (double junction) | ~$170 | Listed separately because **the probe is the recurring item** — budget it as a consumable, not a one-off. |
| Atlas Conductivity K 1.0 Kit — EZO-EC + K1.0 probe + isolator + calibration solution | $229.99 | Buy the kit ($229.99) not the pieces ($207.98 without isolator or solution). **It reads EC only** — the bundled lab probe has no internal temperature sensor, and EC shifts ~2%/°C, so temperature compensation has to come from somewhere else. |
| Atlas SMA PT-1000 temperature kit — EZO-RTD + carrier + probe | $61.99 | The "somewhere else". Originally listed as the circuit alone, on the mistaken assumption the EC probe contained a PT-1000. It does not. |
| pH calibration buffers 4.00 / 7.00 / 10.00 | $20 | Recurring. Buy more than you think. |

### Buy the kit, not the pieces

This is the most expensive lesson in the file, and it had to be learned twice.

The pH line originally specified the circuit ($45.99) and the probe (~$170) as separate items:
**$215.99, with no isolator, no calibration buffers and no storage solution.** The kit is
**$174.99 — $41 cheaper and complete.** The list already carried "buy the kit" advice for the EC
side and simply failed to apply it to pH.

The isolator in particular is not an accessory. A pH circuit reads microvolts, and pumps and
solenoids in the same water bleed enough noise to pin readings at 0 or 14. The isolated carrier
is the fix, and it is already in the box.

## Tier 2 — water temperature and chilling

| Item | Price | Notes |
|---|---:|---|
| ESP32 DevKit (WROOM-32) | $8 | Host for the water probe and the chiller loop. |
| PT-1000 + MAX31865 amplifier | $15 | |
| DS18B20 (alternative) | $10 | |
| Resistor assortment | $8 | |
| Chiller contactor | $30 | **A compressor must not be switched by a smart plug.** |
| Snubber | $10 | |
| Active Aqua 1/10 HP chiller | $240 | **Deferred behind a measurement.** |
| In-reservoir digital thermometer | $12 | **Superseded — do not buy.** |

### Two deferrals worth copying

**The thermometer.** A local display answers "what is it right now", and only if somebody is
standing there. The actual question is "what has it been doing for three weeks", which needs
logging. The logged version costs about $25 in parts and lands in the dashboard.

**The chiller.** Measure reservoir temperature for two to three weeks *first* — until that data
exists the decision cannot be evaluated at all. Above 75 °F water is a pythium risk; under 72 °F
and you skip a $240 purchase and ~264 W of continuous draw on a circuit that already carries a
server rack. Root-zone temperature matters more than air temperature.

**Put the chiller's control loop on the ESP32, not in the home-automation server.** ESPHome's
`thermostat` component makes `min_idle_time`, `min_cooling_off_time` and `min_cooling_run_time`
*required* — it refuses to compile without compressor protection. A home-automation outage or
upgrade then cannot leave a compressor running unbounded.

## Tier 2 — biologicals

| Item | Price | Notes |
|---|---:|---|
| Botanicare Hydroguard (Bacillus inoculant) | $25 | Cheap insurance against pythium at warm root temperatures. Bought with the kit and sat unused for months; first dosed at 40 ml into a ~20-gallon working level. Start it during the dry stage rather than at first mix. |

# Server-Room Hydroponics

A 12-site recirculating ebb-and-flow system for leafy greens, built on the floor of a home
server room, with water level, pH and the flood cycle under full automation.

The room is the grow space, which is the constraint that shapes everything: ambient temperature
is set by a server rack rather than chosen, so plant selection follows the measured room rather
than the other way round.

**New here?** [10 — First run](docs/10-first-run.md) is the build order; everything else is
reference.

**Status:** running. Three batches sown, two harvested. Water-level float ladder validated in
both directions, reservoir pH reading continuously, flood cycle automated with four independent
failsafes. Six of the twelve sites are plumbed; the rest are on hand and waiting on floor space.

---

## What is here

| | |
|---|---|
| [01 — System and physical layout](docs/01-system.md) | Topology, dimensions, why every bucket must sit at one elevation, and the two air gaps that make 3 ft of head safe |
| [02 — Bill of materials](docs/02-bom.md) | 35 line items, what was bought, what was deferred, and the mistakes worth not repeating |
| [03 — The controller (KC868-AI)](docs/03-controller.md) | ESP32 + Ethernet + 48 isolated digital inputs, six I²C expanders, and why it has no analog inputs |
| [04 — Float sensors](docs/04-float-sensors.md) | The depth ladder: wiring, how the sense of each float was *measured* rather than assumed, trip depths, and the two evenings lost to floats that do not actuate in air |
| [05 — Reservoir pH (Atlas EZO-pH)](docs/05-ph.md) | Stamp orientation, the UART→I²C switch that makes the difference between "reading" and "apparently dead", commissioning tells, probe care |
| [06 — Nutrients](docs/06-nutrients.md) | Masterblend 4-18-38 three-part, dose order, EC and pH targets, and why pH goes last |
| [07 — Lighting](docs/07-lighting.md) | Two Spider Farmer SF600 bars, the whole-system photon budget, PPFD/DLI targets, hang height |
| [08 — Automation](docs/08-automation.md) | The ebb-and-flow cycle, depth-driven timing, and the four failsafes |
| [09 — The control chain](docs/09-control-chain.md) | Z-Wave stick → zwave-js → Home Assistant → ZEN20 strip, the firmware auto-off that is the real backstop, and the timing budget it belongs to |
| [10 — First run](docs/10-first-run.md) | **Start here if you are building one.** Boxes to first flood, in the order that works, with what this build did out of order |
| [11 — Sowing and the dry stage](docs/11-sowing.md) | Direct-sow, the desiccation gap that decides the whole method, three staged photoperiods, crop choice |
| [12 — pH calibration](docs/12-ph-calibration.md) | Three-point procedure, a real run's numbers, the trust ladder, and why monthly calibration is 12× too often |
| [`config/`](config/) | The ESPHome configuration that runs the controller, lightly sanitised |

---

## The short version

- **Kit:** GrowAce AirCube Multi-Flow, 12 sites, 60-gallon reservoir. Recirculating — all buckets
  share one manifold and one reservoir.
- **Controller:** KinCony KC868-AI (ESP32, wired Ethernet, 48 optocoupler-isolated digital inputs
  across six PCF8574 expanders) running ESPHome.
- **Water level:** eleven dry-contact float switches — six in the fill-control bucket forming a
  depth ladder from ~3" to ~13", five in the reservoir.
- **pH:** Atlas Scientific EZO-pH circuit on an electrically isolated carrier, lab-grade
  double-junction probe, reading over I²C on the same bus as the input expanders.
- **Actuation:** a **Zooz 800 Series Z-Wave Stick** on USB → `zwave-js-ui` → Home Assistant →
  a **Zooz ZEN20** five-outlet power strip. Pumps, both light bars and the fans are each their own
  switched outlet. The strip's **own firmware auto-off timer** is the backstop that still works
  when the server does not.
- **Flood cycle:** fill → hold → drain, several times a day inside the lights-on window, with the
  flood depth stepping down as the crop ages (8" → 7" → 6").
- **Nutrients:** Masterblend 4-18-38 + calcium nitrate + Epsom, mixed to EC ~1.2 mS/cm at
  pH 5.8–6.0.
- **Light:** 2 × Spider Farmer SF600 — 336 µmol/s total, aiming at 200–300 µmol/m²/s at canopy.

---

## Why the write-up reads the way it does

Most of these pages spend as much space on **what turned out to be wrong** as on the final
answer. That is deliberate. Nearly every hour lost on this build went to something that looked
settled and wasn't:

- Float switches wired across two input terminals, completing no circuit — three sensors read
  nothing through two full watch windows before anyone questioned the wiring rather than the
  sensors.
- Floats that cannot be tested by waving them in the air, because buoyancy is the actuator.
- A $175 pH kit that is invisible to an I²C scan out of the box, because it ships in UART mode —
  indistinguishable from dead on arrival.
- A fill-rate figure measured on the fill bucket being used as the whole-system rate, which are
  different numbers for a good physical reason.
- A hardware safety timer *raised* twice "for margin", which both times moved the last backstop
  past the point where the buckets overflow.
- A container that came up "running" with no network at all after a port collision, reporting
  only `Z-Wave JS not ready` — which points at the radio, not at the port.
- "Recalibrate the pH probe every 30–60 days" — consumer-grade advice applied to a lab-grade
  probe, about 12× too often, and over-calibration is not harmless.

The corrections are kept in place rather than quietly edited out, because the wrong version is
usually the intuitive one, and anyone building this will reach for it too.

## Measurements, not estimates

Where a number appears here it was measured on this rig unless it says otherwise. A few that
matter:

- **26 s/inch** fill rate for the 6-bucket configuration (260 s to the 10" calibration mark);
  drain 28.5 s/inch. Doubling to 12 buckets roughly doubles the volume per inch — the figures do
  not carry over.
- **~38 s/inch** apparent rise in the fill bucket during a flood. Not a contradiction: the fill
  bucket rises while water simultaneously gravity-feeds out to the plant buckets, which take
  30–60 s to equalise.
- A fill stopped at the ~9" float **settles back to ~8.8"**, which is small enough that
  trip-and-stop needs no top-up loop.
- Reservoir pH drifts **up ~0.5 over three days** on a 60-gallon reservoir with a crop on it.
- **29.9 s/inch** measured fill rate to the 11" float (three cycles: 324 / 328 / 334 s, mean 329).
  The original 26 s/inch calibration was ~13% optimistic, and every safety bound computed from it
  was 43 s short.

## Licence

[MIT](LICENSE) for the configuration and code. The prose is a build log; take what is useful.

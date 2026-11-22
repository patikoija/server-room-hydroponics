# Configuration

ESPHome configuration for the KinCony KC868-AI that reads the water-level floats and the
reservoir pH probe.

| File | What |
|---|---|
| `kc868-ai.yaml` | The device config: Ethernet, I²C, six PCF8574 input expanders, every float |
| `hydro-ph.yaml` | The Atlas EZO-pH sensor, included as a package by the file above |

**Lightly sanitised for publication:** the board's address and MAC, the switch port it lands on,
and references to internal tooling are removed. `!secret` references are kept so the structure is
clear — the values come from a secret store at deploy time and were never in version control.

The extensive comments are the point. Float *sense* in particular is per-channel measured rather
than inferred, and those comments are the record of what was observed and when — see
[04 — Float sensors](../docs/04-float-sensors.md).

# Home Assistant configuration

The real package that runs this system, sanitised. Not a cut-down example — ~8,200 lines
including the comments, which are most of the value: nearly every guard in here exists because
something specific went wrong.

| File | What |
|---|---|
| `hydroponics.yaml` | The whole thing as a self-contained HA **package**: helpers, templates, scripts, automations, failsafes |
| `dashboard.yaml` | A Lovelace view — gauges, float ladder, phase, pH trend, manual controls |

## Install

Drop the package in and reference it from `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

with `hydroponics.yaml` inside `config/packages/`. A **package** rather than loose entries so it
cannot collide with keys you already have — everything it defines is namespaced under `hydro_`.

The dashboard is a view; paste it into a dashboard in YAML mode.

**It ships inert.** `input_boolean.hydro_occupied` defaults off, so loading it actuates nothing.
The failsafes arm regardless.

---

## The eight entity IDs you must make match

Everything else is self-contained. These eight come from *your* hardware, and the package will
simply report unavailable until they resolve.

### 1–5. The power strip outlets

```
switch.z_wave_plus_power_strip_1    grow light bar, row 1
switch.z_wave_plus_power_strip_2    grow light bar, row 2
switch.z_wave_plus_power_strip_3    FILL pump
switch.z_wave_plus_power_strip_4    DRAIN pump
switch.z_wave_plus_power_strip_5    circulation fans
```

That is the entity ID Home Assistant generates for a ZEN20 paired through Z-Wave JS, where the
trailing number **is the physical outlet**. If yours differ, find and replace — outlets 3 and 4
are referenced ~95 times between them, so do it with a tool, not by eye.

> **Check which physical socket is which before you trust the number.** Outlets 3 and 4 swapped
> means a drain pump trying to fill, and it presents as a plumbing fault.

### 6–7. The ambient sensor

```
sensor.grow_room_air_temperature
sensor.grow_room_humidity
```

Any temperature/humidity sensor in the grow space. Ours is a Z-Wave multisensor; anything that
produces a numeric state works. Several templates gate on the temperature being
`is_number()`-valid, so an unavailable sensor degrades rather than breaks.

(`sensor.grow_room_sensor_battery` appears on the dashboard only.)

### 8. The notification target

```
notify.hydro_alerts
```

Referenced ~33 times — every failsafe trip, health alert and maintenance reminder. Point it at
whatever you actually read: a phone app, a push service, email. **Do not point it at something
you will mute**, which is the only real failure mode for an alert channel.

---

## The ESPHome entity prefix, which is the one that catches people

Eighteen entities come from the ESPHome node in [`../config/`](../config/) and are referenced as:

```
binary_sensor.hydroponics_inputs_hydro_float_6_inch
sensor.hydroponics_inputs_hydro_reservoir_ph
sensor.hydroponics_inputs_ph_probe_slope_acid
button.hydroponics_inputs_ph_cal_mid_7_0_do_first    # from name: "pH cal MID 7.0 (do first)"
...
```

**That prefix is not arbitrary and it is not in the ESPHome YAML either.** Home Assistant builds
an entity ID from the *device* name plus the *entity* name:

```
esphome:
  name: kc868-ai
  friendly_name: Hydroponics Inputs     ← becomes the "hydroponics_inputs_" prefix

binary_sensor:
  - name: "Hydro float 6 inch"          ← becomes "hydro_float_6_inch"
```

So the prefix changes if you change `friendly_name`, and it also picks up an **area or device
prefix** depending on how your HA names things. In this build the live prefix is longer than what
is published here, because the device sits in a named area — that part has been normalised out.

**Two ways to make them line up, pick one:**

1. Set `friendly_name: Hydroponics Inputs` on the ESPHome node and let HA generate these IDs. Then
   check one entity in Developer Tools before assuming the other seventeen match.
2. Keep your own names and find-and-replace `hydroponics_inputs_` throughout `hydroponics.yaml`.

Either way, **verify against Developer Tools → States rather than reasoning about it.** A
mismatched prefix makes every float read unavailable, and the package's defensive templates mean
that looks like "no water" rather than like a configuration error.

---

## What the package actually contains

| Group | Notable members |
|---|---|
| **State machine** | `input_select.hydro_phase` persists across restart — the floor-flood fix ([08](../docs/08-automation.md#reboot-safety-the-floor-flood-problem)) |
| **Crop profiles** | photoperiod and flood frequency per crop, so times are never set by hand |
| **Calibration helpers** | `hydro_fill_seconds`, `hydro_drain_seconds`, depth targets, every bound tunable from the UI |
| **Failsafes** | fill-overflow, drain-runaway, both-pumps interlock, master enable, fault latch |
| **Cross-checks** | `hydro_floats_disagree`, `hydro_floats_healthy`, `hydro_phase_contradicts_floats`, `hydro_state_drift` |
| **pH trust ladder** | `hydro_ph_reading_sane`, `hydro_ph_calibration_stale`, `hydro_ph_probe_degraded` → `hydro_ph_dosing_permitted` ([12](../docs/12-ph-calibration.md#the-trust-ladder-three-checks-not-one)) |
| **Tracker bridge** | a webhook that accepts sow/harvest events from an external grow log |

### Numbers in here that are specific to this rig

Change these before running it on different plumbing — they are the ones measured on six buckets:

```
hydro_fill_seconds          313    derived so the computed fill carries a 15 s margin over
                                   the measured 329 s to the 11" float
hydro_drain_seconds         285    10" calibration; measured 295 s
hydro_pump_autooff_minutes    6    MUST stay below the overflow line — see below
hydro_cal_depth_in           11    the float the calibration was timed to
```

> **`hydro_pump_autooff_minutes` is not just a helper.** It mirrors the timer set *on the power
> strip itself*, and that bound is governed by where your buckets overflow, not by your software.
> Raising it "for margin" puts the last backstop past the spill point — it was raised twice here
> and was wrong both times.
> [The arithmetic](../docs/09-control-chain.md#the-timing-budget-and-how-it-was-got-wrong-twice).

---

## What was changed for publication

- The ESPHome entity prefix normalised to `hydroponics_inputs_` (the live one carries an area
  name).
- The ambient sensor entities renamed to `sensor.grow_room_*`.
- Notification targets collapsed to one `notify.hydro_alerts`; ours are several topics on a
  self-hosted push service.
- Internal tracker IDs, merge-request numbers, host addresses and internal URLs removed. Where a
  comment's reasoning referenced one, the reasoning is kept and the identifier is not.
- One `!secret` reference remains, for the tracker-bridge webhook ID. Name only; put your own in
  `secrets.yaml`.

Outlet numbers, I²C addresses, PCF8574 banks and measured timings are all left exactly as they
are — they are the useful part.

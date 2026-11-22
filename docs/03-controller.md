# 03 — The controller: KinCony KC868-AI

**ESP32 + LAN8720 wired Ethernet + 48 optocoupler-isolated digital inputs across six PCF8574
expanders.** Running ESPHome, reporting to Home Assistant over the native API.

## Why this board

Dry-contact float switches are exactly what it is for. Forty-eight isolated channels means the
depth ladder can be as fine as the plumbing allows, with the optocouplers keeping the wet side
away from the logic side.

## It has no analog inputs — none

Every channel is on/off. **Anything with a 0–10 V or 4–20 mA output cannot be read by this board
at all.** That is worth knowing before buying a sensor, and it is the single most important fact
about the KC868-AI when planning what to attach to it.

It is also why the pH probe does not connect "to the board" in the way people expect: a bare pH
electrode puts out ±414 mV at roughly 100 MΩ impedance, which no general-purpose ADC can read
anyway. The EZO circuit is the thing that bridges them — see [05 — pH](05-ph.md).

## Ethernet, not wifi

The config deliberately has no wifi block. The board is wired, and the LAN8720 pinout below is
KinCony's hardware design, not a choice:

```yaml
ethernet:
  type: LAN8720
  mdc_pin: GPIO23
  mdio_pin: GPIO18
  phy_addr: 0
  clk:
    mode: CLK_OUT
    pin: 17
```

## I²C

```yaml
i2c:
  sda: GPIO4
  scl: GPIO5
  scan: true
```

**GPIO5 is a strapping pin.** That is KinCony's choice for SCL, so the build-time warning about
it is expected and not a misconfiguration.

`scan: true` makes the boot log list which expanders actually answered, which is how you tell a
dead expander from a wiring fault in ten seconds instead of an evening.

## The expander addresses are not in numeric order

```yaml
pcf8574:
  - { id: hub_in_1, address: 0x24 }   # inputs 1-8
  - { id: hub_in_2, address: 0x25 }   # inputs 9-16
  - { id: hub_in_3, address: 0x21 }   # inputs 17-24
  - { id: hub_in_4, address: 0x22 }   # inputs 25-32
  - { id: hub_in_5, address: 0x26 }   # inputs 33-40
  - { id: hub_in_6, address: 0x23 }   # inputs 41-48
```

`0x24, 0x25, 0x21, 0x22, 0x26, 0x23` maps to input banks 1–8, 9–16, 17–24, 25–32, 33–40, 41–48
**in that order**. Sorting them tidily into ascending address order silently remaps every input
on the board — the config still compiles, the board still boots, and every float is suddenly
somewhere else.

## Address map, all together

| Device | Address | What |
|---|---|---|
| PCF8574 ×6 | 0x21–0x26 | 48 digital inputs |
| Atlas EZO-pH | **0x63** | reservoir pH over I²C |

No collision, which is why the pH circuit can share the bus the input expanders already run.

## Debounce

Every float carries:

```yaml
filters: [delayed_on: 2s, delayed_off: 2s]
```

Two seconds in each direction. Water arriving at a float is turbulent — the supply free-falls
~11" into the fill bucket — and without debounce a single fill produces chatter on every channel
it passes. With it, a full six-float soak produced **zero** chatter.

The debounce has to be shorter than the time water spends between adjacent floats, which here is
a minimum of ~9.6 s (see [04 — Float sensors](04-float-sensors.md)). Two seconds is comfortably
inside that, and that relationship is worth re-checking if the float spacing ever gets tighter.

## Deploy it from the repository, not from a laptop

The configuration is deployed by automation from version control. Editing a copy on a workstation
and flashing it by hand is how the config on the device stops matching the config in git — and
there is no way to tell from the outside that it has happened.

Secrets (the API encryption key and the OTA password) are rendered at deploy time from a secret
store into `secrets.yaml` and are never committed. The `config/` directory here keeps the
`!secret` references so the structure is clear.

## One known gap

**Nothing alarms when the controller disappears.** A health sensor correctly went unavailable when
the unit was powered down for eight hours, and nobody was notified: every automation referencing
the floats guards defensively against `unavailable` rather than *triggering* on it. A working
detector wired to no buzzer — and one of those floats is the overflow failsafe.

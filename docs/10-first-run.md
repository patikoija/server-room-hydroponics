# 10 — First run: boxes to first flood

The order matters, and some of it is not reversible. This is the sequence as it should have been
done, which is not quite the sequence it was done in.

## 1 — Site it before you plumb it

- **Single tier, on the floor.** Every bucket at the same elevation or the flood will not balance
  ([01](01-system.md#every-bucket-must-sit-at-the-same-elevation)).
- **Flood tray under the whole array, first.** Not after the first leak.
- Opposite wall from any server intakes; reservoir and fill bucket at the heads of the two rows.
- Decide 6 sites or 12 **now**, because every timing figure below is specific to that choice.

## 2 — Prepare the media

Rinse the hydroton. Then rinse it again. The dust is relentless, it clouds the reservoir, and it
is the one job that is far worse to do after the system is assembled.

Use **original**-size pebbles, not giant — original packs snugly around a 1.5" cube with no
voids, giant leaves gaps and the cube tilts or sinks during flood cycles.

## 3 — Preserve the two air gaps

When you assemble the hoses:

- the supply hose stops **short** of the fill bucket's bottom, so water free-falls into it;
- the drain hose terminates at the **top** of the reservoir and free-falls in.

Those gaps are what make 3 ft of head above the buckets safe. See
[01](01-system.md#why-3-ft-of-head-above-the-buckets-is-not-a-flood-hazard). If a hose is ever
replaced, they have to be preserved.

## 4 — Wire the control chain before the pumps

Full detail in [09 — The control chain](09-control-chain.md). The short order:

1. Z-Wave stick in, bound by `/dev/serial/by-id`.
2. Pair the ZEN20; confirm five separate switch entities.
3. **Set the strip's firmware auto-off on the fill-pump outlet — before a pump is plugged in.**
4. Label the physical outlets. Outlets 3 and 4 swapped is a drain pump trying to fill.

## 5 — Install the floats, then measure them

Mount the ladder, wire each float **one wire to its input terminal and one to KCOM** — not across
two input terminals, which completes nothing.

Then **measure the sense of every channel in water**, one at a time, bottom to top, while
watching the log. Do not infer it from the pin configuration: the floats are not all the same
hardware, and a normally-open and a normally-closed switch on identical wiring read exactly
opposite. They also do not actuate in air, so this has to be a water test.

Record which terminal corresponds to which depth. On this build the port-to-depth mapping is
**not** in numeric order, and the only reason that is known is that someone actuated them one at
a time with timestamps running.

## 6 — Calibrate the fill and drain

With plain water in the reservoir, run the fill pump from empty and time it to a known float.
Then do it again, twice.

Three consecutive cycles on this rig gave 324 / 328 / 334 s to the 11" float — mean **329 s**,
spread 10 s. That is the number to compute from. A single cycle is not a measurement; the first
calibration here came from one run to a different float and was 13% optimistic, which made every
safety bound derived from it 43 s short.

Then set, in this order and checking each against the next:

```
computed fill  <  software ceiling  <  hardware auto-off  <  overflow line
```

## 7 — Commission the pH probe

1. Switch the EZO circuit from UART to I²C **before** wiring it in, on the unseated stamp
   ([05](05-ph.md#it-ships-in-uart-mode-and-that-looks-exactly-like-a-dead-kit)). Until you do, it
   is invisible to an I²C scan and looks like a dead kit.
2. Seat the stamp with the **probe pins facing the probe**. It is not keyed.
3. **VCC to 3.3 V, not 5 V.**
4. Confirm 0x63 answers on the bus scan.
5. Test the probe in a cup of water on its own first. If it pins to 0 or 14 in the reservoir but
   settles in a cup, that is electrical noise from the pumps — which is what the isolated carrier
   is for.
6. **Three-point calibrate** before trusting a reading — see
   [12 — pH calibration](12-ph-calibration.md).
7. Return the probe to **storage solution** whenever it is out of the reservoir. A dried bulb is
   an unrecoverable failure.

## 8 — Prove the failsafes, deliberately

The point of a failsafe is that it fires when you have stopped watching, so the only time you
will ever see it work is now.

- **Overflow float:** lift it in water, supervised, hand on the plug. The automated cross-check
  structurally cannot validate the topmost float
  ([04](04-float-sensors.md#the-one-that-still-is-not-proven-and-why-it-is-the-one-that-matters)).
- **Hard-wired interlock:** unplug the relay's 12 V supply mid-fill. The pump must stop. If it
  keeps running, the float is wired to the wrong contact.
- **Both-pumps interlock:** try to turn both on.
- **Reboot recovery:** start a fill, then restart the automation server. The pump should be forced
  off and the system drained to a known-empty state before anything floods again.

## 9 — Mix nutrients, in the right order

Nutrients first, pH last; calcium nitrate → Masterblend → Epsom, each fully dissolved before the
next. Target EC ~1.2 mS/cm, then pH to 5.8–6.0. Full detail in
[06 — Nutrients](06-nutrients.md).

**Do not mix the reservoir until the crop needs it** — see
[11 — Sowing](11-sowing.md#do-not-mix-the-reservoir-on-day-0). Ten days of warm, still, lit
nutrient solution with no plants drawing on it is a pythium and algae starter culture.

## 10 — Only now enable the cycle

Put the real entity IDs into the automation and turn it on. Until this step the configuration
ships with the cycle disabled and placeholder entity IDs specifically so that deploying it cannot
fire a pump.

---

## Things this build did out of order, for the record

- **The fans were mounted and never wired**, through two complete crops. Batch 1 died of
  damping-off, batch 2 tipburned — the two classic still-air failures. Wire them at step 1; they
  are the cheapest part of the whole system and they were the leading cause of both losses.
- **The pH probe sat unread for ten days** because the circuit was still in UART mode and that
  looks identical to broken hardware.
- **Three floats read nothing for two watch windows** because of the two-terminal wiring, and the
  sensors were suspected long before the wiring was.
- **The first float soak never reached the overflow float**, so the one float that the overflow
  failsafe depends on remains unproven.

# Ballistics & Zeroing

How projectiles fly in Reforger and how to make sights actually hit where they aim.
Everything here was verified in game with measured impacts.

## Sights ranges

`SightsComponent → Sights Ranges` is an array of `SightRangeInfo`:

```
SightRangeInfo {
  Range 0.0 100        // X = animation source position 0..1, Y = meters
  WeaponPosition PointInfo { Angles <pitch> 0 0 }   // barrel elevation for this range
}
```

- With n entries, the X interval is `1/(n-1)` (evenly spaced positions of the zeroing drum).
- `SightsRangesDefaultIndex` is an **array index**, not a distance in meters. Setting "150"
  when you meant "the 150 m entry" points past the array.
- The elevation angle per range can be computed ballistically — first approximation without drag:
  `θ ≈ atan(g·R / (2·v₀²))`. With drag, integrate numerically or calibrate in game (below).

**Generator route:** set Sights Point Front/Rear onto your front/rear sight points (they must
form a line!), assign `BaseZeroingGenerator`, then RMB → *Process zeroing data*.
⚠️ This does **not** work in the Prefab Edit Mode — place the weapon in a world, process there,
then *Apply to prefab*.

## ⚠️ Trap #1: the missing ZeroingWeaponAimModifier

`Rifle_Base` does **not** include a `ZeroingWeaponAimModifier` in the MuzzleComponent's Weapon Aim
Modifiers. Without it, your SightRangeInfo angles are **never applied** — the weapon always
shoots flat.

Symptom: a 320 m/s projectile lands at ~150–190 m no matter the zeroing (that's just ballistic
drop from shoulder height). Fix: add `ZeroingWeaponAimModifier` to the modifiers list — and when
overriding the list in a child prefab, re-list the inherited entries (sway) with their instance
GUIDs.

## ⚠️ Trap #2: optics override the weapon's zeroing

With an optic attached, `GetCurrentSightsZeroing()` (and the fire solution) uses **the optic's
SightsComponent**, not the ironsights. Any custom zeroing table must be present on **both** the
weapon and every optic that mounts on it.

## ⚠️ Trap #3: PIP scopes need a zeroing type

2D/PIP optics have a zeroing-type option (none / reticle offset / camera turn). With the default
"none" the PIP camera tilts *with* the barrel — impacts appear low through the scope even though
the ballistics are right. Use **camera-turn** style zeroing (camera counter-rotates by the current
range) for FCS-style optics, or reticle-offset with calibrated holdover fractions like the vanilla
PSO-1.

## Air drag: calibrate in game

The drag model behaves roughly quadratically (`decel ≈ k·v²`), **but the effective k in game is
~3.2× the naive value** you'd compute from `v(x) = v₀·e^(−kx)` fits. Practical recipe:

1. Pick your `v₀` and an AirDrag value; compute a zeroing table.
2. In game, shoot at a measured long distance (max zeroing).
3. Measured impact short of the aim point → fit the effective k from that one data point
   (`k_eff ≈ 3.2 × configured` was the measured factor on 1.7) and recompute all angles.

For realistic drag values, fit against published ballistic coefficient data (G1/G7) — the official
ballistics spreadsheet in Bohemia's sample material and any BC calculator work; iterate AirDrag
until computed velocity-at-distance matches the table.

## Penetration

`ShellMoveComponent` penetration = Depth (mm) + Density + Speed at reference. Work from RHA
equivalence (e.g. a 5.56 M855A1-class round: ~9.5 mm mild steel at 350 m ≈ 5.7 mm RHA;
density 7860; speed = remaining velocity at that distance). Steel gamemats apply a KE coefficient,
so the effective numbers differ from paper values — verify with *Show penetration config* and
in-game tests.

## Airburst / programmable ammo (advanced, proven pattern)

A scripted `ScriptComponent` on the projectile can implement airburst fuzes with **zero new UI**:

- Read the shooter's dialed range at launch:
  `BaseTriggerComponent.GetInstigator() → instigator entity → BaseWeaponManagerComponent.
  GetCurrentWeapon() → GetCurrentSightsZeroing()` → that *is* the detonation distance.
- Server-side only (`Replication.IsServer`), arm after a few frames, per-frame distance check,
  sub-frame precision via `GetCallqueue().CallLater(remaining/speed*1000)`.
- Detonate through the trigger component (`OnUserTrigger(owner)`, guarded by `WasTriggered()`);
  keep impact detonation as fallback; clean up CallLater in `OnDelete`.
- Optional proximity fail-safe: per-frame `TraceMove` along the flight path, detonate a few
  meters before the surface.

The same "zeroing as an input channel" idea generalizes to rangefinders and fire-control systems.

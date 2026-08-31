# Weapon Prefab Configuration

The Workbench half of weapon building. Assumes an imported `.xob`
([Weapon pipeline](weapon-pipeline.md)).

## Base inheritance

Two viable parents:

1. **`Prefabs/Weapons/Core/Rifle_Base.et`** — the generic skeleton. You must add more yourself
   (see below).
2. **A concrete vanilla rifle base** (e.g. the M16A2 base) — inherits a working animation graph,
   IK hand pose, recoil curves, RigidBody, physics attributes, attachment slots and action
   contexts. Recommended for first weapons; override the mesh and the deltas.

⚠️ When inheriting a concrete weapon, remember **slot merge semantics**: slot lists merge **by
name** across the inheritance chain. Inherited slots keep spawning their vanilla contents; to
neutralize one, define a slot with the same name and empty `Prefab ""`. Attach your parts to the
*inherited* slot names instead of inventing new ones (else you get doubled parts).

## Components you must touch

**MeshObject** — your `.xob`. ⚠️ Override the **inherited component instance** (its instance GUID),
never add a second MeshObject (→ *"cannot be combined"*).

**RigidBody** — enable *Model Geometry* (without it the weapon can't be picked up).

**SCR_MeleeWeaponProperties** — rifle melee damage ~45.

**WeaponComponent** — UI info (display name), obstruction (3 phases: torso back → weapon back →
ADS blocked; usually the default character offset test suffices).

**MuzzleComponent**
- `barrel_chamber`/`barrel_muzzle` drive projectile spawn/direction.
- **Fire modes:** `MaxBurst 1` = single, `-1` = full auto, `>1` = burst
  (interruptible variants selectable). Vanilla ships FireMode configs you can reference; RPM is
  per-weapon (`RoundsPerMinute`).
- **MagazineWell** class (see below) + default `MagazineTemplate`.
- Dispersion: diameter/range in meters — `MOA × 0.029 = diameter @ 100 m`.
- Shorter barrel variants: `Muzzle Init Speed Coef` here, or reduced init speed on the bullet.

**Weapon Aim Modifiers** (inside MuzzleComponent)
- `RecoilWeaponAimModifier` — linear/angular/turn-offset curves (start from a similar vanilla
  weapon's values).
- ⚠️ **`ZeroingWeaponAimModifier` is NOT inherited from Rifle_Base** — without it your sights
  ranges are never applied and every shot flies flat
  ([Ballistics & zeroing](ballistics-and-zeroing.md)).
- When overriding the modifier list in a child, re-list inherited entries (e.g. sway) **with their
  instance GUIDs**, or you lose them.

**SightsComponent** — `eye` point = ADS camera; front/rear sight points must form a line;
sights ranges array (see [Ballistics & zeroing](ballistics-and-zeroing.md)).

**SCR_WeaponAttachmentsStorageComponent** (inventory)
- Item display name, weight (kg), volume (cm³ — assault rifle ≈ 1500, MG ≈ 3000).
- `ItemOneHandAnimAttributes` in custom attributes; `PreviewRenderAttributes` → camera distance
  for the inventory 3D preview.
- `ItemAnimationAttributes → Animation IK Pose` — **this `p_*_ik.anm` drives the first-person
  hand pose**, not the deployment hand targets. Wrong IK pose = hands floating beside the weapon.

**ActionsManagerComponent** — action contexts (pivot + radius; keep radius small, ~0.1). Context
names must be **unique** — empty/duplicate names collide ("already exists"). A muzzle-slot
inherited from a base auto-registers its own context; don't add a duplicate.

**WeaponSoundComponent** — see [Custom sounds](custom-sounds.md).

**Effects** — `SCR_MuzzleEffectComponent` (muzzle flash particle), CaseEjecting/casing particles
(pivot `barrel_muzzle` / ejection port bone).

## Magazine well (caliber compatibility)

MagazineWell types are tiny script classes:

```c
// Scripts/Game/Weapon/MyMod_MagazineWell.c
class MagazineWellMyRifle : BaseMagazineWell {}
```

Compatibility = the weapon's MuzzleComponent `MagazineWell` class **matches** the magazine's
MagazineComponent `MagazineWell` class. Recompile scripts, then assign on both sides. When
rebasing a weapon on a vanilla rifle, override the *inherited* magwell instance on the weapon side.

## Magazine prefab

Inherit `Magazine_Base.et`: MeshObject + MagazineComponent (`MaxAmmo` — don't forget it! —
ammo config, magwell class), inventory attributes (weight ≈ 0.12–0.34 kg by size,
`WeightPerAmmo`), `MagazineUIInfo` with caliber/type strings and an `m_MagIndicator` config
(that's what feeds the ammo-check UI when holding R).

## Ammunition

Inherit `Ammo_Bullet_Base.et` for **magazine-fed** projectiles.
⚠️ `Ammo_GrenadeLauncher_Base` is only for single-load grenade prefabs — using it for
magazine-fed ammo produces replication errors.

`ShellMoveComponent` (override the inherited instance):
- Init Speed, Mass, **Air Drag** (see [Ballistics & zeroing](ballistics-and-zeroing.md)),
  Init Speed Variation (~±10 m/s).
- Penetration Depth/Density/Speed (RHA-referenced). Damage is derived automatically
  (RMB → *Show penetration config*).
- Tracers: inherit `Ammo_BulletTracer_Base.et` (burn start/…, tracer models).

**AI:** `AICombatPropertiesComponent` goes on the **magazine/grenade/rocket, not the weapon**
(UsedAgainst, Priority). AI ballistic tables: a `BallisticTableArray` config wired into
ShellMoveComponent → RMB *Generate ballistic tables*.

## Deployment (bipods / resting)

- `AimingModifierAttributes` (custom attributes) → Deployment Points. System identifier 0 =
  no/folded bipod; 2 = bipod deployed (needs bipod bones + stabilization points on the leg bones).
- MuzzleComponent → Deployment IK Targets: LEFT HAND = `slot_magazine`, RIGHT HAND =
  `snap_hand_right`, BUTTSTOCK = offset at the stock end.
- Vanilla controls: **C** = rest/deploy, **hold V** = toggle bipod. Even without an animated
  fold, stabilization works.

## Verification loop

Spawn in a test world → fresh session → grep the newest log for your prefab name and `(E)`.
Common traps: duplicate component instances, PivotID typos (silent), missing `+` when overriding
action lists (`additionalActions +` — without the plus you *replace* the inherited list and lose
equip/pickup).

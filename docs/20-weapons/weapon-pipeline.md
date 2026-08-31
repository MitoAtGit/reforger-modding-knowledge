# Weapon Pipeline — Overview

The full path from a Blender model to a working rifle in game. Official basis:
BIKI *Weapon Creation* (Asset Preparation + Prefab Configuration) — this page adds ordering and
the traps.

```
Blender model ──► rig + memory points + colliders ──► FBX ──► Workbench import (.xob)
      ──► prefab (inherit Rifle_Base) ──► components/ballistics/sounds ──► arsenal ──► publish
```

## Stage 1 — Blender asset

**Orientation:** barrel along Y+ ([FBX import rules](../10-assets/fbx-import-rules.md)).

**Split the weapon into parts.** Grip, ironsights, magazine, and every attachment are their own
FBX — variants are built via slots in the prefab, not by hiding meshes.

**Skeleton:** armature named exactly `Armature`, at origin, scale 1.0. Root bone `w_root`
pointing Y+; moving parts as child bones using the **vanilla bone names** (animation export
profiles expect them):

```
w_root
├── w_trigger          (~25° pull)
├── w_fire_mode        (selector; one key per mode)
├── w_charging_handle
├── w_bolt             (3 states: closed / open / closed-on-last)
├── w_bolt_release, w_mag_release, w_safety … (as the weapon needs)
```

Everything fully skinned; leftover geometry → `w_root`.

**Memory points** (plain-axes empties, parented to the armature; rotation = axis):

| Point | Used by |
|---|---|
| `barrel_chamber`, `barrel_muzzle` | MuzzleComponent — projectile spawn + direction |
| `eye` | SightsComponent — ADS camera position |
| `slot_ironsight_front` / `slot_ironsight_rear` | sights |
| `slot_magazine`, `slot_optics`, `slot_underbarrel`, `slot_barrel_muzzle` | attachment slots |
| `snap_hand_right`, `snap_hand_left` | hand-IK targets (deployment) |

⚠️ Slot names are matched **exactly** by the prefab's PivotIDs — a Blender `.001` suffix breaks
them silently (sights/mag/muzzle just stop working). Semantics matter too: put `slot_barrel_muzzle`
at the *thread plane* (where a suppressor mounts), `barrel_muzzle` at the actual muzzle exit;
derive `slot_magazine` from the magwell, not from the magazine mesh.

**Colliders:** at minimum one collider with a Weapon layer preset — without it there are **no
equip/pickup actions in game**. Simple: one `UCX`/`UBX` with `usage="WeaponFire"`. Richer: `UCX_`
(`Weapon`) + small `UTM_` (`FireGeo`). Real gamemat names (`weapon_metal_<GUID16>`).
Budget sanity: official sample weapon FireGeo is **29 faces**.

**Textures:** PBR MetalRough → BCR/NMO packing
([Materials & textures](../10-assets/materials-and-textures.md)).

## Stage 2 — Import

Right-click FBX → *Register and Import (as Model)*. Check Import Settings: **Export Skinning** ON
for the weapon body (rigid attachments: **OFF**, Scene Hierarchy ON). Watch the log for skinning
and collider warnings.

## Stage 3 — Prefab

Inherit `Prefabs/Weapons/Core/Rifle_Base.et` (or the closest vanilla rifle base — inheriting a
concrete rifle base like the M16A2's brings its animgraph, IK pose, recoil curves, attachment
slots and action contexts for free, which is usually a massive head start).

Details: [Weapon prefab configuration](weapon-prefab-configuration.md).

## Stage 4 — Ballistics, sounds, animation

- [Ballistics & zeroing](ballistics-and-zeroing.md) — sights ranges, air drag, the
  ZeroingWeaponAimModifier trap.
- [Custom sounds](custom-sounds.md).
- [Weapon animation](weapon-animation.md) — or inherit vanilla animation via the base prefab.

## Stage 5 — Into the game

- [Arsenal & catalogs](../60-publishing/arsenal-and-catalogs.md) to make it obtainable.
- [Workshop publishing](../60-publishing/workshop-publishing.md).

## Sanity checklist before calling it done

- [ ] Weapon spawns with 0 errors in a **fresh** session log
- [ ] Pickup/equip actions appear (collider + RigidBody *Model Geometry* enabled)
- [ ] ADS aligns (eye point), sights zero correctly at 2+ ranges
- [ ] Fire modes cycle; correct RPM; casings eject
- [ ] Magazine inserts/removes; correct caliber & capacity; shows in inspection (R)
- [ ] Sounds fire (first-person and distance)
- [ ] Inventory: name, weight, volume, 3D preview framing
- [ ] Attachments mount at correct positions
- [ ] No leftover dev/debug components; passes a fresh-session log scan

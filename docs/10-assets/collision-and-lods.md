# Collision & LODs — Deep Dive

The collider rules from [FBX import rules](fbx-import-rules.md), expanded with budgets, layer
presets, and the failure modes seen in real imports.

## Collider roles

An entity typically carries two kinds of collision:

| Role | Purpose | Shape guidance |
|---|---|---|
| **Physics collision** | movement, ragdolls, vehicle contact, item pickup | as few and as simple as possible — primitives (`UBX/USP/UCS/UCL`) or one convex `UCX` |
| **Fire geometry (FireGeo)** | projectile hits, penetration, *and inventory interaction raycasts* | may be detailed, may be concave — `UTM_` trimesh is the norm |
| **View geometry** | AI/visibility occlusion | often combined with FireGeo via a combined layer preset |

Notes:

- A weapon needs at least one collider with a Weapon-type layer preset — **without it, the
  equip/pickup actions don't appear in game at all.** Simplest setup: one collider with a
  `WeaponFire` preset covering both roles; richer setup: `UCX_` (Weapon) + `UTM_` (FireGeo).
- On vehicles, physics collision should be *very* simple (few large convex/box/cylinder shapes) —
  detailed physics shapes cause contact jitter. FireGeo carries the detail.
- View/Fire can be a **layer-preset combination on one collider** (e.g. a box with a
  `PropFireView` preset) — you don't necessarily need separate meshes (verified in official
  samples).
- Buildings: FireGeo trimesh with a building-type preset (e.g. `BuildingFireView`), concrete-type
  gamemat. Remember the triangle budget below — split big buildings into several `UTM_` parts.

## Budgets (from official sample assets)

Bohemia's own sample weapon is instructive: total visual ~4.7k verts, physics `UCX` **14 faces**,
FireGeo `UTM` **29 faces**. If your FireGeo has 40,000+ faces (a decimated copy of the visual
mesh), it is orders of magnitude too heavy — and may exceed the 65,535-**triangle** limit and get
silently dropped.

Rule of thumb: FireGeo of a rifle ≈ a few hundred faces max; of a building ≈ low thousands,
split into parts.

## Layer presets you'll actually use

Set as the `usage` string custom property (comma-list allowed):

- Weapons: `Weapon`, `FireGeo`, `WeaponFire`
- Vehicles: `VehicleSimple`, `VehicleComplex`, `FireGeo`, `FireView`, `MineTrigger`
- Props/buildings: `PhyCol`, `PropFireView`, `BuildingFireView`
- `CenterOfMass` for COM helpers (when not using the `COM_` prefix convention)

The authoritative list lives on the BIKI (*Collision Layers* page). If actions/hits behave oddly,
the layer preset is the first thing to check.

## Import failure modes (symptom → cause)

| Symptom | Cause |
|---|---|
| *"Invalid collider … Export skipped"* / *"Face is degenerated"* | broken hull — rebuild simple; an 8-vert box is a legitimate fix |
| *"Convex is not closed"* | non-manifold `UCX` — must be a closed convex volume |
| Collider silently missing in game, no error | triangle count over 65,535 after triangulation → split the mesh |
| *"Wrong GUID/name for resource … in property 'UBX_…'"* per collider | fantasy material names on colliders → assign real `name_GUID16` gamemats or fix meta `GeometryParams` before reimport |
| Equip/pickup actions missing on a weapon | no collider with Weapon layer preset, or RigidBody's *Model Geometry* not enabled on the prefab |
| Attachment mesh explodes when weapon animates | rigid attachment imported with `ExportSkinning` → remove it, keep `ExportSceneHierarchy` |
| Collider works, but penetration feels wrong | wrong gamemat (e.g. generic 7mm steel on a fuel tank) — vanilla ships part-specific gamemats (engine, fuel_tank, drive_shaft…) worth using |

## The meta file (`.xob.meta`)

The import settings and `GeometryParams` (per-collider layer preset + gamemat reference) live in
the model's `.meta`. Two practical facts:

- Meta fixes **survive reimports** — fixing the meta and re-importing the FBX is a legitimate
  repair path.
- Gamemat references are compiled *into* the `.xob` — a meta-only fix takes effect **after** the
  next FBX reimport, not immediately.

## LOD generation

- Decimate-based auto-LODs (bk-tools `create_lods`, ratios like 0.3/0.15/0.08/0.06) are fine for
  most props/vehicles; hand-tuned LOD1 pays off on hero assets (see optics note in
  [Attachments](../20-weapons/attachments.md)).
- Keep the LOD0 collider a child of LOD0 where the tool expects it; verify naming after any batch
  operation.

# FBX Import Rules

Distilled from the official [FBX Import](https://community.bistudio.com/wiki/Arma_Reforger:FBX_Import)
documentation plus real import debugging. Get these right and the importer is your friend; get them
wrong and it fails *silently*.

## Orientation

Everything points along **Y+** in Blender/3ds Max (becomes Z+ in Enfusion):

| Asset | Y+ is… |
|---|---|
| Weapon | barrel direction |
| Vehicle | nose |
| Building | main entrance |
| Wall segment | runs from origin along **X+** (NOT centered!) |

Apply **Rotation & Scale** before export (not Location).

## Naming

- Case-insensitive; no spaces/special characters (`/\<>:"` → `_`).
- Prefixes/suffixes UPPERCASE, core name lowercase, unique object names.
- ⚠️ A dot in a material name is **stripped on import** — Blender's `.001` duplicates collide with
  the original name and corrupt FBX↔emat matching. Clean all `.001` suffixes (objects, materials,
  empties) before export.

## LODs

- Suffix `_LODx` (`_LOD0`, `_LOD1`, …; vehicles support up to 8).
- Multiple meshes may share one LOD level (e.g. `Car_LOD0` + `Wheel_LOD0`).
- An object with no known prefix/suffix lands in LOD0.
- Gaps shift up (only LOD0/2/4 present → 2 is treated as 1). Official sample assets use aggressive
  reference chains (e.g. 1673→802→157→81→28→25 verts across 6 levels for a small prop part).

## Colliders

Prefixes: `UBX_` box · `UCX_` convex · `USP_` sphere · `UCS_` capsule · `UCL_` cylinder ·
`UTM_` trimesh.

Rules that actually bite:

- As simple and as few as possible; avoid overlaps (>2 overlapping is unacceptable).
- `UCL/UCS/USP`: origin must sit at the object's center (*Set Origin → Origin to Geometry*); only
  capsule/cylinder can be scaled along their vertical axis. Apply Rotation & Scale.
- **Limit: 65,535 vertices/faces per collider — and the importer counts TRIANGLES** after
  triangulating trimeshes. A 48k-polygon trimesh can be 69k triangles → collider is **silently
  discarded** (no error, entity simply has no collider). Count triangles, not polygons; split big
  trimeshes into several `UTM_` parts (same layer preset is fine).
- Invalid colliders are skipped with *"Invalid collider … Export skipped"* — degenerate faces,
  non-manifold "convex" shapes (*"Convex is not closed"*). Fix: rebuild as a clean closed convex
  shape; for stubborn cases an 8-vertex bounding box beats hours of hull repair.
- Dynamic objects (weapons, vehicles, magazines): **never trimesh for physics collision** — one
  primitive, else convex. Trimesh is for Fire/View geometry. Static objects: prefer one uniform
  trimesh over many primitives.
- Hierarchy: **static colliders have NO parent**; only colliders for animated parts are parented
  to their bones. Only trimesh colliders may be skinned; everything else 100% weight / object
  relation to a bone.
- **`usage` custom property (layer preset):** a string custom property on the collider object, e.g.
  `Weapon`, `FireGeo`, `WeaponFire`, `VehicleSimple`, `BuildingFireView`. Missing → the collider
  collides in ALL layers + console warning. It may be a comma list (`collision,fire`).
  Weapon convention from official samples: physics `UCX_` = `Weapon`, `UTM_` fire geometry =
  `FireGeo` (or a single collider with `WeaponFire` covering both).

## Game materials (physical surfaces)

Assign a **gamemat** via the material *name*: `MaterialName_MaterialGUID`
(e.g. `weapon_metal_<GUID16>`; get the GUID via right-click on the `.gamemat` → *Get Resource
GUID(s)*). Notes:

- Trimeshes may carry a gamemat per face (enable *Merge Tri Meshes*; only same layer presets merge).
- Primitives/convex: exactly one physical material per collider.
- ⚠️ **Fantasy material names on colliders write broken references** into the model's meta
  (`Wrong GUID/name for resource … in property "UBX_..."` for every collider). Fix: remove
  made-up materials from colliders in Blender, or fix the `GeometryParams` in the `.xob.meta`
  before reimport (seed the correct gamemat first, then reimport — otherwise the importer writes
  broken name-based refs again).
- After changing gamemats: reset Geometry Params in Import Settings, then reimport.

## Sockets (empties)

- Plain empties; the empty's **rotation defines the axis**. Hierarchically parentable.
- `COM_` = center of mass (missing → CoM at 0,0,0). Official weapon samples keep `COM_Weapon` as
  an empty and hand-IK snap points **unparented**.
- `LC_` = land contact (min 2 points; snaps object to ground slope).
- `OCC_` = occluder (not triangulated, axis-aligned, max 3–4 per building, min ~4–5 m).
- Sockets are otherwise a prefab-side convention — they import as scene nodes ("memory points")
  that prefabs reference by exact name (PivotIDs). **Exact** name: `eye.001` will never match `eye`.

## Skinning

- Skinned objects must be **fully skinned** — every face weighted to a bone; leftovers go to the
  root bone (`w_root` / `v_root`). Otherwise: *"Missing some mesh skinning weights … Weighting
  them to root"*.
- Max **4 bones per vertex**.
- Name the armature exactly `Armature` to avoid an artificial extra bone on import; origin at
  0,0,0, scale 1.0.

## Blender FBX export — the 3 critical settings

1. Object Types: **Empty + Armature + Mesh** enabled.
2. **Custom Properties ON** (otherwise all `usage`/layer presets are lost!).
3. **Add Leaf Bones OFF.**

Also: transforms applied, export smoothing = Face, and for model-only exports set `bake_anim`
off (stray actions bloat the file and can corrupt object scale). Axis settings: the Workbench
importer accepts both the `-Z forward / Y up` default and `Y forward / Z up` — the three settings
above are the ones that break things.

## Import into Workbench

- Right-click the FBX → **Register and Import (as Model)** → produces `.xob` (+meta, + emats from
  FBX material names).
- Import Settings: **Export Skinning** for skinned meshes; **Export Scene Hierarchy** for pure
  socket objects (e.g. a magazine that only needs its snap points); *Merge Meshes* is a render
  optimization (meshes with different UV-set counts won't merge).
- Changes take effect on **Reimport** (the pipeline keys on the FBX being newer than the compiled
  output).
- ⚠️ **`ExportSkinning` on rigid attachments is wrong** — a rigid attachment skinned to the weapon
  skeleton explodes when the weapon animates. Attachments = rigid (`Export Scene Hierarchy` only);
  only the weapon body is skinned.

## Related

- [Collision & LODs](collision-and-lods.md) — deeper on colliders and failure modes.
- [Blender pipeline](blender-pipeline.md) — tooling that automates most of this.

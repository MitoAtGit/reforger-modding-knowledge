# Troubleshooting — Symptom → Cause → Fix

Collected from real debugging sessions. Ordered by where you'll see the symptom.
General method: **newest log directory first**, first `(E)` from your own files is the real error,
everything after is cascade.

## Import / assets

| Symptom | Cause → Fix |
|---|---|
| *"Invalid collider … Export skipped"*, *"Face is degenerated"*, *"Convex is not closed"* | broken collider hull → rebuild simple (an 8-vert box is legitimate); see [Collision & LODs](../10-assets/collision-and-lods.md) |
| Collider silently missing (no error) | >65,535 **triangles** after triangulation → split the trimesh |
| `Wrong GUID/name for resource … in property "UBX_…"` per collider | fantasy material names on colliders → real `name_GUID16` gamemats, or fix meta `GeometryParams` **before** reimport |
| `Missing some mesh skinning weights … Weighting them to root` | incomplete skinning → weight leftovers to the root bone |
| Model invisible in game | broken material (`.emat` parse failure — check for **UTF-8 BOM**!) |
| Red/white checkerboard | missing texture (different from invisible!) |
| Sights/mag/muzzle silently non-functional | memory-point names with Blender `.001` suffixes — engine PivotID lookups are exact → clean names + reexport (or repoint every PivotID) |
| Attachment mesh explodes when weapon animates | rigid attachment imported with `ExportSkinning` → remove it, keep scene hierarchy |
| Import changes don't appear | reimport needed; FBX must be newer than compiled output; play mode doesn't reimport scripts/layouts — restart the session |
| Non-ASCII asset/material names | registration silently broken → rename to ASCII, keep GUIDs via metas |

## Prefabs

| Symptom | Cause → Fix |
|---|---|
| `component X cannot be combined with component X` | you added a new component instead of overriding the **inherited instance** (by instance GUID) |
| `Broken resource GUID=0` | prefab reference written with path but no GUID → use GUID-qualified references |
| Inherited slots spawn unwanted parts (rotors, glass…) | slot lists **merge by name** → override same-named slots with `Prefab ""` |
| `starting node X doesn't exist` (animation) | assigned graph lacks expected node names → use the inherited vanilla graph, or author the nodes |
| Warning *"should have a HierarchyComponent"* on slot-spawned prefab | the component class is called **`Hierarchy`** — the warning text's name doesn't exist |
| Item has no pickup/equip actions | no collider with Weapon layer preset, or RigidBody *Model Geometry* off, or actions list overridden without `+` |
| Weapon always shoots flat regardless of zeroing | missing `ZeroingWeaponAimModifier` ([Ballistics & zeroing](../20-weapons/ballistics-and-zeroing.md)) |
| Zeroing wrong only with optic mounted | optic's SightsComponent wins → zero tables on weapon **and** optic |
| One-time `Wrong GUID {0000…}` errors during play after saving a prefab | hot-reload artifact, transient → verify in a fresh session before chasing it |
| Edited `.et` file seems ignored | Workbench caches parsed prefabs in RAM; autosave may overwrite your disk edit if the prefab is open in an editor |

## Configs / overrides

| Symptom | Cause → Fix |
|---|---|
| Custom input action never fires | override config's meta doesn't carry the **vanilla GUID** → fix meta, restart Workbench |
| Faction arsenal reduced to just your item | missing `+` on `m_aMultiLists` |
| Vanilla keybind pages vanished from options | missing `+` on the keybinding categories list |
| Item absent from arsenal | see the checklist in [Arsenal & catalogs](../60-publishing/arsenal-and-catalogs.md) — game mode/rank gate first |
| Key does nothing on German/other layouts | US-position keycodes (`KC_EQUALS`…) → use layout-stable keys |

## Scripts

| Symptom | Cause → Fix |
|---|---|
| Hundreds of "Can't find class" errors | cascade from the **first** real error above them |
| Component never ticks | `SetEventMask` + `SetFlags(ACTIVE)` missing — or the entity is a carried item (frame events don't fire; use the call queue) |
| `NULL pointer to instance` crash | teardown-mid-tick: re-check references after any call that can close/delete what you're using |
| Script-spawned entity won't move | `RigidBody Static 1` freezes it → Static 0, Kinematic 1, Gravity 0; call `Update()` after `SetTransform` |
| Runtime-spawned destructible takes no damage | wrong destruction setup for runtime spawns; enable the (shipped-disabled) multi-phase destruction config |
| Changes don't take effect in play | stale session — stop and restart Play; verify a fresh `Module: Game … CRC32` line |

## Audio

| Symptom | Cause → Fix |
|---|---|
| `AUDIO (E): Compile error: Item @"….acp|id" not found` | broken audio graph → rebuild in Audio Editor ([Custom sounds](../20-weapons/custom-sounds.md)) |
| Game crashes when a sound would play; Audio Editor crashes on open | hand-edited `.acp` node graph → restore/rebuild it properly |
| No sound, no errors | measure the sample's **peak level** (silent WAV!), check mono/48 kHz, then wiring |
| `SignalManagerComponent not present` at spawn | add `SignalsManagerComponent` to the entity |

## Terrain / world

| Symptom | Cause → Fix |
|---|---|
| Instant crash on surface-mask import (`toGamma`/NVTT in crash log) | inconsistent per-tile layer textures on disk → close WB, delete all `*_layer.edds`/`*_layer.dds` tiles, reopen, re-import ([Surfaces & satmap](../40-terrain/surfaces-and-satmap.md)) |
| `TERRAIN (E): Unable to load a terrain tile texture of correct size` | same as above — the tell-tale line |
| Vehicles bounce on flat terrain | bare `GenericTerrainEntity` used → use the default terrain prefab |
| `Generator is not a direct child of a shape` | reparent the generator under its spline/polygon |
| In-game map empty despite placed content | missing 2D export (`.topo`) / map background ([Map finishing](../40-terrain/map-finishing.md)) |
| World's shore config fails to load | shore map never built → Manage → Build shore map |
| Trees float above ground when bulk-placed | wrong Y convention for that entity class (some classes are terrain-relative, others absolute) — place one by hand, save, and copy what the editor wrote |

## Workbench itself

| Symptom | Cause → Fix |
|---|---|
| Workbench appears frozen after triggering an import | a **modal dialog** is waiting (possibly behind other windows) — find it; don't kill the process |
| "Resources are leaking" assert on closing/stopping | diagnostic-executable assert, not your mod; the non-diag exe doesn't show it |
| Port/bridge tools can't connect after a crash | leftover crash-reporter process holds the port → end it |
| Log seems to say nothing happened | you're reading a stale `logs_*` directory — take the newest |

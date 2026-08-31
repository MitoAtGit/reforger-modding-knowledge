# Blender Pipeline & Tools

Blender is the de-facto DCC for Reforger modding. This page covers the tooling landscape and the
version traps.

## The two key addons

### Enfusion Blender Tools (EBT) — official

Bohemia's own addon, shipped with the Tools install
(`Arma Reforger Tools\Blender\` — install the zip **without unpacking**, plus the Data.zip with
animation export profiles).

- The only path for **TXA animation export** (Blender action → `.txa` → engine compiles `.anm`).
  Requires the Workbench running with **Net API enabled** in Workbench options.
- Export enforces conventions: file must live inside a loaded Workbench project, a gamemat on
  visual geometry is a hard export error, material names restricted to `[A-Za-z0-9_]`, and its
  material autopilot creates missing `.emat` files automatically.
- Includes a far stricter Model QA (40+ checks: LOD density uniformity, collider fit sampling,
  texture naming) than any community tool — use it as the acceptance gate.
- ⚠️ **Version support:** officially targets **Blender 3.6 LTS** and is tested with **4.5 LTS**;
  4.x generally works. **Do not run it on Blender ≥5.2** — Blender 5.2 removed the legacy
  `Action.fcurves` API that EBT's TXA exporter relies on. Keep a 4.5/5.1 install for EBT work.

### bk-reforger-blender-addons — community

[steffenbk/bk-reforger-blender-addons](https://github.com/steffenbk/bk-reforger-blender-addons):
a large operator suite (vehicle/weapon rig scaffolding, collider creation with `usage` properties,
socket/empty creation, LOD generation, FBX exporter with Enfusion orientation, NLA helpers,
building destruction, crater generator).

- Modes VEHICLE/WEAPON/CUSTOM drive bone prefixes (`v_` / `w_`).
- ⚠️ Its collider operators assign placeholder material names (e.g. `Collision_Material`) — exactly
  the fantasy names that produce broken gamemat references on import
  ([FBX import rules](fbx-import-rules.md)). Assign real `name_GUID` gamemats afterwards, or fix
  the `.xob.meta` before reimport.
- ⚠️ The repo's `bl_info` version string lags behind actual releases — compare file contents, not
  the version number, when checking for updates.

Also useful: **simple_collider** (Weisl) for interactive collider shapes with configurable naming
presets (set up an Enfusion preset: UBX_/UCX_/USP_/UCS_/UCL_/UTM_), including VHACD auto-convex
decomposition. ⚠️ It re-applies its last-used naming preset on every file switch — check prefixes
after switching files.

## Blender version notes

- **Blender 4.5 LTS**: the safe all-round choice (EBT-supported).
- **Blender 5.2+**: fine for modeling/rigging and bk-tools, but:
  - `Action.fcurves` is **gone** — F-curves live under `action.layers[].strips[].channelbag(slot)`.
    Scripts and addons using the old API silently break.
  - **Slotted Actions:** an action copied via `action.copy()` does **not** bind its slot — the pose
    silently doesn't evaluate (character stays in T-pose). Fix in Python:
    `ad.action = act; ad.action_slot = act.slots[0]`.
  - FBX export is Python-only (`bpy.ops.export_scene.fbx`); import has a native path.

## Workflow skeleton (new asset)

1. Model (or clean up a purchased model — check licensing!).
2. Normalize: real-world scale (measure!), orientation Y+, apply rotation & scale, ASCII names,
   purge `.001` suffixes and orphan data.
3. Rig: armature named `Armature` at origin; `w_`/`v_` bones per convention; full skinning
   (rest → root bone), max 4 bones/vertex.
4. Sockets/memory points as empties (exact names — see the per-asset pipeline docs).
5. Colliders with correct prefixes + `usage` custom property + real gamemat names.
6. LODs (`_LODx`).
7. Export FBX (Empty+Armature+Mesh, Custom Properties ON, Leaf Bones OFF, bake_anim off).
8. Workbench: Register and Import → fix warnings → iterate.

## Hygiene rules (learned the expensive way)

- **Never use linked duplicates for placed copies** (`Alt+D` / `obj.copy()` sharing mesh data).
  One edit-mode change propagates to *every* copy — it looks like the whole scene broke. Use full
  copies (`Ctrl+D` / `o.data = o.data.copy()`); keep a small palette of originals as the single
  source of truth.
- Big `.blend` bloat is usually **packed textures held by orphan materials** — a file with 2
  meshes can hide hundreds of MB in unused 4K sets. Purge orphans + unpack to trim (observed:
  −650 MB on one weapon file).
- Trust `matrix_world`, not object scale assumptions — imported armatures often carry non-1 scale
  (e.g. 0.33), and bones with trailing spaces in their names do exist in the wild. Measure.
- Watch for actions accidentally baked into model exports (`bake_anim`) — they bloat files and one
  stray single-frame action can force a wrong object scale on every export.
- Headless (`blender -b`) quirks: operator availability via `hasattr(bpy.ops.x, y)` lies — check
  `dir()`; `report({'ERROR'})` raises `RuntimeError` in background mode.

## Related

- [FBX import rules](fbx-import-rules.md)
- [Hard-surface modeling](hardsurface-modeling.md)
- [Weapon animation](../20-weapons/weapon-animation.md)

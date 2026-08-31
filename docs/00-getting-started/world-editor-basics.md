# World Editor Basics

The World Editor (WE) is where worlds are assembled: terrain, entities, generators, layers.

## Camera & keys (the ones people get wrong)

- Hold **RMB** = fly mode; move with **W/A/S/D**.
- **Q = up, Z = down.** ⚠️ **E is the rotate *tool*, not "up"** — a very common mix-up.
- **Mouse wheel while flying = camera speed.** If the camera barely moves, raise the speed first.
- Double-clicking an entity row in the **Hierarchy panel frames the camera on it** — the fastest
  reliable way to get somewhere.
- The ViewCube snaps orientation (e.g. TOP) but not framing.

Full keybind list: BIKI *World Editor* pages (see
[Official resources](../90-reference/official-resources.md)).

## Worlds, sub-scenes, layers

- A world is an `.ent` file. **Worlds are binary** — author them in the editor, not by hand.
- A world can be a **SubScene** of another world (e.g. your scenario world inheriting a vanilla
  terrain). Your changes layer on top.
- Entities live in **layers** (`<World>_Layers\*.layer`). Use layers to organize content
  (roads / vegetation / structures / gameplay) and to keep diffs reviewable.
- Nothing is written to disk until you **save the world** — imports and edits included.

## Placement rules of thumb

Measured from how vanilla maps place fortification-style objects — useful defaults for believable
placement of long objects (walls, trenches, sandbags) on sloped ground:

- Up to ~3° ground slope: place level, no terrain edit needed.
- ~3–9°: tilt the object with the slope (roll/pitch).
- Over ~9°: don't tilt further — **terrace the heightmap** locally instead.
- Long ground-contact objects benefit from a shallow depression (~0.3 m) under the contact line so
  edges don't float.
- After placing on slopes, always inspect from downhill — the downhill edge is where floating
  shows first.

## Generators

Spline/polygon-driven generators (forests, walls, roads, lakes) do the heavy lifting:

- A generator entity must be a **direct child of its Shape entity** (spline or polygon). The
  warning *"Generator is not a direct child of a shape"* means exactly that.
- Roads **carve/affect the heightmap**; river and lake generators do **not** — see
  [Roads, water, vegetation](../40-terrain/roads-water-vegetation.md).
- Generators re-run when their shape changes.

## Terrain tool

Select the terrain entity, open **Window → Current Tool** (docks right, tabs
Manage / Sculpt / Paint / Holes / Info). The **Manage** tab hosts the real import/export buttons
(heightmap, satellite map, surface masks, shore map). Details in the
[terrain pipeline](../40-terrain/terrain-pipeline.md).

## Playtesting

- **Play** starts a session in the editor. Remember: running sessions hold **stale scripts and
  prefabs** — stop and restart Play after edits.
- The **diag menu** (`Win+Alt` or `Win+Ctrl`, on Win11 try all three keys) exposes invaluable
  debug overlays: vehicle center-of-mass/forces/suspension, compartment positions, AI info,
  "Reset vehicle", and much more.
- Log directories are per-session — always read the newest
  (see [Workbench setup](workbench-setup.md)).

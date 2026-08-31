# Hard-Surface Modeling for Game Weapons

A reference-driven workflow for building accurate, animation-ready weapons in Blender — distilled
from building several pistols/rifles from photo and blueprint references.

**TL;DR: measure, don't eyeball. Form first, detail second. Verify every step against the
reference.** And: single-image AI-to-3D is *not* a shortcut for hard-surface weapons — it produces
blobby, unseparable meshes; clean reference modeling beats it every time.

## 1. Calibrate references

- Load a clean side profile as an image empty; scale it so the real-world length matches
  (manufacturer specs beat guesswork — overall length, barrel length, height, width).
- ⚠️ Marketplace beauty renders are often **distorted** (wrong aspect). If proportions fight you,
  cross-check against a second source (orthographic wireframe sheets, a reference mesh, front-on
  photos). One real case: a stretched side image put the grip 30 mm off.
- Cast shadows in reference photos corrupt silhouette readings near the bottom — use stricter
  thresholds or ignore the affected band.

## 2. Silhouette first

Build the side contour as a single profile face and verify it as a wireframe overlay on the
reference **before** adding any detail.

## 3. Real cross-sections, not extruded slabs

A profile extrusion with rounded caps looks "blocky" no matter how much you bevel. Build the true
cross-section ring (curved top → top-edge chamfer → flat slab sides → bottom chamfer — read it
from a front-on reference) and loft it along the length. This single change is the difference
between "toy" and "believable".

## 4. Features one boolean at a time

Serrations, milled reliefs, ejection ports, sights, holes: apply each cutter **individually**
(EXACT solver). A merged non-manifold cutter destroys the mesh. Angled features (serrations at
~18°) read much better than axis-aligned ones.

## 5. Shading

- Always `Shade Auto Smooth` (~25–30°) on hard-surface parts — plain smooth shading "melts" edges.
- A small bevel modifier (`clamp overlap`, angle-limited) for edge highlights.

## 6. Part separation & pivots (for animation)

Slide, barrel, hammer, trigger, magazine, bolt, charging handle… as **separate objects**, each with
its origin exactly on its motion axis (hammer = pin, slide = bore axis, barrel = tilt lug). Test
the motion early with a quick pose (slide back, hammer cocked, mag dropped).

## 7. High-poly → low-poly (the pro pipeline)

- High-poly: boolean blockout + **subdivision with support loops** (control loops near hard edges
  keep them sharp while surfaces smooth) — that's the step beyond bevel-modifier looks.
- Low-poly: apply booleans on a copy, strip non-silhouette edges, bake detail into a normal map.
  The low-poly + bakes is the actual game deliverable.
- UVs: mark sharp edges as seams, hide seams from view directions, straighten islands, uniform
  texel density, **triangulate before export**.

## 8. Fine detail is texture, not geometry

Engravings, stampings, wear: normal/roughness maps, not modeled geometry — unless it changes the
silhouette.

## Verification habits

- Overlay renders against the reference at every stage; wireframe-over-photo catches drift early.
- Measure with data, not eyes: bounding boxes, bore height, key distances. If a part "looks off",
  find the number that proves it.
- When a real 3D reference mesh exists (bought scan/model), use it *as a measuring stick* for
  proportions — and model your own clean topology; don't retopo-copy someone else's mesh into a
  mod unless its license explicitly allows redistribution.

## Related

- [Blender pipeline](blender-pipeline.md) — the rig/collider/export half.
- [Weapon pipeline](../20-weapons/weapon-pipeline.md) — what the finished model must contain.

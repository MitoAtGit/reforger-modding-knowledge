# Surfaces & Satmap

How terrain gets its ground materials and its color — and the crash to avoid.
Verified hands-on on a real island build (1.8-era Workbench).

## The model

- **Detail surfaces** = per-surface *visual* `.emat` + *physical* `.gamemat`, painted via
  weight masks. What you see up close and what footsteps/ballistics feel.
- **Satmap (satellite / super-texture)** = the terrain's albedo at distance. The engine blends
  satmap (far) with detail surfaces (near).
- ⚠️ **Hard limit: max 5 surfaces per terrain block** (64 m block). More *global* surfaces are
  fine as long as no single block exceeds 5 overlapping. Exceeding it degrades/breaks rendering.
  Start with ~3 surfaces, save and reload after big imports.

## Adding surfaces

Terrain tool → **Paint** tab → right-click the surface list → *Add Material(s)* → pick vanilla
surface emats (`Terrains/Common/Surfaces/` has forest floor, grass, sand, gravel, dirt, seabed…)
or your own. Layer 0 is the base — it fills 100 % wherever nothing else is painted.

## Surface masks: full round-trip (the important correction)

Per-surface weight masks **export and import**:

- Paint tab, bottom: *Import/export masks from/to* + a directory picker.
- **Export first** — it teaches you the exact expected filenames (`<SurfaceMaterialName>.png`)
  and size. Masks are 8-bit grayscale (black = 0, white = full, greys blend), one per surface.
- Edit or generate masks externally (classification from an orthophoto, erosion outputs,
  hand-painting) → **Import** → weights apply immediately → save.
- Masks are **resampled on import** (don't need to match the grid exactly).
- First-ever import pops *"Init Surface Map"* (choose tile texture size, e.g. 256 → ~1 m/texel)
  and a *"Nonundoable action"* confirm.
- Make the masks a clean **partition** (weights per texel summing to full) — e.g. beach mask +
  (255 − mask) for the base. Adding a surface somewhere then automatically *removes* the base
  there — that's the partition working for you.

### Mask generation tips

- Classify from evidence, not guesses: sample your orthophoto (brightness / excess-green
  percentiles) to pick thresholds; a measured classification beats hours of eyeballing.
- Enforce the 5-per-block limit in your generator: per 64 m block, zero out the weakest surface
  while over the limit.
- Keep a signed-distance **shore band** (a few meters of sand straddling the waterline, then
  seabed across all underwater area) — it reads instantly as "coast".

## ⚠️ The surface-import crash (hours lost; save yourself)

Symptom: surface-mask import **instantly crashes** the Workbench (crash log shows an NVTT/texture
call; earlier in the log: `TERRAIN (E): Unable to load a terrain tile texture of correct size`,
e.g. "Expected 256, Loaded 128").

Cause: **inconsistent per-tile layer textures on disk**, left behind by an aborted
*"Change surface map size"* or a half-finished import. The importer reads every existing tile and
dies on wrong-sized ones. It is *not* about surface count or memory.

Fix: close Workbench → delete **all** `<Terrain>\.Data\*_layer.edds` **and**
`.EditorData\*_layer.dds` → relaunch → the terrain loads (base surface only) → re-import masks
(they recreate all tiles cleanly).

Prevention: **don't use "Change surface map size"** once tiles exist; keep tile texture size
consistent.

## Satmap

- The satmap is **import-only** in the terrain tool (*Import satellite map*; there is no export —
  "Export Map Data → Rasterization" is the 2D-map background, *not* the satmap).
- Import with **"In linear color space" OFF** for normal sRGB images; "Render roads" on if you
  want road overlays baked.
- **Author it externally.** The engine cannot bake a whole-map texture from your surfaces — you
  create the satmap so it *matches* the painted surfaces:
  - Real-terrain maps: start from the orthophoto, blend your surface scheme's colors into changed
    areas (clearings, shore band, paths), modulated by ortho luminance so it stays natural.
  - Fictional maps: stack your exported surface masks × each surface's average albedo color
    (sample the surface textures), plus hillshade. Community tools exist that do exactly this
    tiling job (see [Official resources](../90-reference/official-resources.md)).
- Same pixel orientation as heightmap/masks, or land and color drift apart.

## Verifying

- After import: fly low — grid-looking ground = only the default base surface is present.
- Surface vs satmap agreement: a clearing painted grass but satmap-dark reads as a bug from the
  air; fix the satmap, not the surface.
- Physical check: footstep sounds/decals per surface tell you the `.gamemat` side works.

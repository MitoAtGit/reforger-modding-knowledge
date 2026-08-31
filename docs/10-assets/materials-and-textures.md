# Materials & Textures

How Enfusion materials work and how to feed them standard PBR maps — including without Substance
Painter.

## The three layers

| File | Role |
|---|---|
| `.emat` | **visual material** — shader class + parameters + texture map references (text file) |
| texture (imported → `.edds`) | the pixel data; you author PNG/TGA/TIF, Workbench compiles `.edds` on import |
| `.gamemat` | **physical surface material** — ballistics/penetration/sound/particles; assigned on colliders and terrain surfaces |

A surface (terrain) needs BOTH a visual `.emat` and a physical `.gamemat`.

## Channel packing (the part everyone gets wrong)

Enfusion's standard PBR shader (`MatPBRBasic` and friends) uses packed maps:

- **BCR** = Base Color (RGB) + **Roughness in Alpha**
- **NMO** = **DirectX** normal (R,G) + **Metalness (B)** + **Occlusion (A)** — normal Z is
  reconstructed in the shader
- **NHO** = normal (R,G) + **Height (B)** + Occlusion (A) — the parallax variant (B is height,
  not metal)

Rules:

- Enfusion samples **DirectX-style tangent normals (green down)**. Most marketplace/free packs
  ship OpenGL (green up) → **flip the green channel** or lighting breaks subtly.
- No roughness map? Invert a gloss/smoothness map.
- A minimal working `.emat` is just a `MatPBRBasic` with `BCRMap` and `NMOMap` referencing your
  textures by `{GUID}path`.
- Verify your packing with pixel checks: in a correct NMO, B is metalness (0 or ~1 zones), A is AO.

## Authoring paths

- **Substance Painter:** Bohemia ships official Enfusion export presets (`.spexp`) in the
  [Arma-Reforger-Misc](https://github.com/BohemiaInteractive/Arma-Reforger-Misc) repository —
  install them into Painter's export presets and you get correct BCR/NMO in one click.
- **Substance-free:** entirely viable. Bake AO/normal/curvature/position in Blender (Cycles bakes,
  GPU accelerated), then channel-pack with any image tool (script or GIMP). A flat mesh's baked
  tangent normal being uniform is *correct*, not a failed bake.
- Texture sizes: 4K for hero weapon sets is common; mind the total.

## Import behavior

- Drop the PNG/TGA into the project → the file watcher imports it (`.edds` + `.meta` with a fresh
  GUID). Read the GUID from the generated `.meta`.
- Texture *type* matters: color maps and normal maps get different import configs (the importer
  keys off suffixes like `_BCR`, `_NMO`/`_Normal`). Follow the naming convention and the right
  config is chosen automatically.
- Compile formats are handled for you (BC7 for color, BC5/BC7 for normals, mips).

## Gamemats

- Assign on colliders via the material-name convention (`name_GUID16` — see
  [FBX import rules](fbx-import-rules.md)) or in the model's meta `GeometryParams`.
- Vanilla ships a rich set: generic metals, glass, rubber, concrete, dirt, plus **part-specific
  vehicle gamemats** (engine, fuel_tank, differential, battery…) with proper penetration/damage
  behavior — prefer the specific one for the zone you're building.
- Browse them under `Common/Materials/Game/` in the Resource Browser; grab GUIDs via right-click →
  *Get Resource GUID(s)*.

## Pitfalls

- ⚠️ **Never write Enfusion text files with a UTF-8 BOM.** PowerShell `Set-Content -Encoding utf8`
  writes a BOM; the `.emat` parser then fails with *"Material file has incorrect format"* and the
  model renders **invisible**. Write BOM-less UTF-8 (`[IO.File]::WriteAllText` with
  `UTF8Encoding($false)`), LF line endings to match vanilla.
- Invisible model = broken material. **Red/white checker = missing texture.** Different failures,
  different fixes.
- Visual meshes need material slots — a mesh with no material can't be wired to an `.emat` on
  import.
- Shared "middle" detail textures: many vanilla emats reference shared `_SharedData` tiling maps;
  your own materials can do the same pattern (base color + tiling detail) for terrain surfaces.

## Related

- [Surfaces & satmap](../40-terrain/surfaces-and-satmap.md) — terrain-side material usage.
- [Collision & LODs](collision-and-lods.md) — gamemat assignment on colliders.

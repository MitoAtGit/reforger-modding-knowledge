# Official Resources & Community Tools

The links that matter. Official sources first — they are the ground truth this repo paraphrases.

## Official documentation

- **BIKI — Arma Reforger hub:**
  <https://community.bistudio.com/wiki/Category:Arma_Reforger>
  Key entry points: *Weapon Creation*, *Car Creation*, *Terrain Creation/Tutorial*, *FBX Import*,
  *Directory Structure*, *World Editor*, *Weapon Animation*, *Mod Publishing Process*,
  *Collision Layers*, *Textures*.
- **Enforce Script syntax & API:** the BIKI scripting pages + the public Script API reference
  (also shipped offline in `<Arma Reforger Tools>\Workbench\docs\`).
- **Arma Platform dev hub:** <https://reforger.armaplatform.com/> (news, Boot Camps, workshops).
- **Licenses / Game Content Usage Rules:** <https://www.bohemia.net/community/licenses>

## Official sample repositories (GitHub)

- **Arma-Reforger-Samples** — <https://github.com/BohemiaInteractive/Arma-Reforger-Samples>
  Sample weapon/character/vehicle projects, the character animation rig example
  (`Character_AnimationRig_Example.blend`), reference budgets.
- **Arma-Reforger-Misc** — <https://github.com/BohemiaInteractive/Arma-Reforger-Misc>
  Official weapon animation source .blends (M16, AK74, M60, SVD, M9, RPG-7…), pose libraries,
  vehicle animation sources, collider shape examples, Substance export presets (`.spexp`),
  ballistics spreadsheet.
  Tip: these repos are large — use git sparse-checkout for the folder you need.

## Official video series

- **Modding Boot Camp playlist** (Arma Platform YouTube) — especially:
  #4 HUD/UI slot system · #8 Blender weapon workflow · #9 Animation in Workbench · #10 Scripting.

## Community tools (use, don't copy from)

Linking ≠ endorsement of content licenses — check each project's license before reusing anything.

### Blender
- **bk-reforger-blender-addons** (steffenbk) —
  <https://github.com/steffenbk/bk-reforger-blender-addons> — rigs, colliders, sockets, LODs,
  Enfusion FBX export ([notes](../10-assets/blender-pipeline.md)).
- **simple_collider** (Weisl) — interactive collider authoring with naming presets.
- **AutoGrip** (Jetpack-Crow) — <https://github.com/Jetpack-Crow/autogrip> — MIT, shrinkwrap-based
  hand-grip posing for Rigify rigs.

### Terrain / GIS
- **ARTE** (QGIS plugin) — <https://github.com/Rendszerguru/ARTE-QGIS-plugin> — DEM + satellite +
  OSM → Enfusion-ready terrain inputs.
- **GTT — GEO Terrain Tool** — community exporter for real-world terrain data.
- **tilw-terrain-tools** (TilW) — <https://github.com/Til-Weimann/tilw-terrain-tools> — includes
  the Seamless Satmap Tool (tiles surface albedos per exported masks).
- **QGIS** — <https://qgis.org/> — the open-source GIS everything real-world starts in.

### Reference / browsing
- **AR Explorer** — <https://arexplorer.zeroy.com/> — community Doxygen browser of the Reforger
  script API with a source browser. Note: its class list can lag the game version, and a missing
  class page does not mean the class doesn't exist — the file/source index is the reliable path;
  the in-Workbench Script Editor remains authoritative.

### Modeling references
- 80.lv breakdowns and BlenderBros hard-surface material for the high-poly→bake pipeline
  ([Hard-surface modeling](../10-assets/hardsurface-modeling.md)).

## Communities

- Official Arma Discord (modding channels) and the Arma Reforger Workshop for studying *what*
  exists (remember: study ideas, never copy other authors' files/scripts —
  [COMPLIANCE.md](../../COMPLIANCE.md)).

# Terrain Pipeline — Overview

Building a Reforger map is a **dependency-ordered** pipeline. Do the stages in order — every
stage constrains the next, and redoing an early stage invalidates everything after it.

```
land (heightmap) → water (ocean/rivers/lakes) → surfaces & satmap → life (vegetation)
     → people (settlements) → roads → detail props → gameplay (navmesh, 2D map, scenario)
```

The causal chain mirrors how real landscapes form: terrain decides where water flows, water
decides where life and people go, people build roads, roads attract detail.

## Stage summary

| Stage | Doc | Key facts |
|---|---|---|
| 1. World & terrain entity | below | grid size × cell size = world meters; grid must be a multiple of 128 |
| 2. Heightmap | [Heightmaps & real-world data](heightmap-and-real-world-data.md) | 16-bit import, resample min/max, Invert-Z default |
| 3. Surfaces & satmap | [Surfaces & satmap](surfaces-and-satmap.md) | ≤5 surfaces per block; satmap is import-only; masks round-trip |
| 4. Water | [Roads, water, vegetation](roads-water-vegetation.md) | one global ocean plane; rivers/lakes are generators that do NOT carve terrain |
| 5. Vegetation | same doc | Forest Generator, not per-tree placement |
| 6. Settlements & roads | same doc | road generators DO affect the heightmap |
| 7. Finishing | [Map finishing](map-finishing.md) | shore map, 2D map, navmesh, believability checklist |

## Creating the world

1. New world in World Editor; place `GenericTerrain_Default.et` at 0 0 0.
   ⚠️ Use the **default terrain prefab**, not a bare `GenericTerrainEntity` — the bare entity
   lacks physics material setup and vehicle wheels bounce.
2. Right-click the terrain entity → *Custom → Create new terrain…* → set grid size (multiple of
   128) and cell size. Examples: 2048 × 2 m = 4096 m world; 4096 × 3.75 m ≈ 15 km;
   8192 × 3.75 m ≈ 30 km. Cell size 2–4 m is the practical range — vanilla-quality maps sit
   around 2 m; large worlds trade resolution for area.
3. Terrain tool = **Window → Current Tool** with the terrain selected
   (tabs: Manage / Sculpt / Paint / Holes / Info). The **Manage** tab holds the import/export
   buttons (heightmap, satellite map, normal map, shore map, surface map size…).
4. **Save the world after every successful import** — imports live only in memory until saved.

## Golden rules

- **Work strictly in pipeline order.** Terrain → roads → buildings → decoration. Never re-sort.
- **Design from real-world logic:** pick a real climate/region archetype, get real elevation data
  or model believable landforms, place settlements where people would actually build (flat,
  near water, sheltered), route roads along contours.
- Keep every generated input (heightmap, masks, satmap) **outside** the project in a source
  folder, versioned — you will re-import many times.
- Non-undoable actions are called out by the editor ("Nonundoable action") — take the hint:
  back up the world folder before terrain-level operations.
- ⚠️ **Never run "Change surface map size" casually** — see
  [Surfaces & satmap](surfaces-and-satmap.md) for the crash it can cause.

## Scale sanity

A 4 km² map with believable density is months of solo work; 15 km+ maps are team/procedural
territory. Procedural pipelines (QGIS/GDAL scripting, external generators) shine for the
*inputs* (heightmap, masks, layouts) — final assembly and polish stay in the World Editor.

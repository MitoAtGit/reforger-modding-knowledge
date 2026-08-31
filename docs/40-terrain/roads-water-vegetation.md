# Roads, Water, Vegetation

The generator-driven middle of the map pipeline.

## Roads

Roads are spline-driven generators:

- Structure: a `SplineShapeEntity` (the polyline) with a **RoadGeneratorEntity child** (from the
  vanilla road generator prefab library: asphalt classes, dirt, forest tracks, trails, runways),
  which materializes the `RoadEntity` (mesh + material).
- Road generators can **affect the heightmap** (grade the road into terrain) — the one generator
  family that does.
- Set beginning/ending piece options deliberately — default end pieces can add unwanted stubs at
  spline ends.
- Geometry quality: resample your polylines to a uniform spacing (~8–12 m) and apply a light
  corner-rounding smooth **before** sampling terrain heights — raw GIS polylines produce kinks
  and gaps.
- Route like a road engineer: follow contours, avoid >8–10 % sustained grades, cross drainage at
  distinct points (bridge/ford), stagger junctions.
- Real-world road networks: OSM (via QGIS) gives you centerline geometry to convert into splines;
  classify (primary/secondary/track/path) → map each class to a road generator prefab + width.

## Water

- **One global ocean plane.** Height 0 is sea level by convention; the shore gets its polish from
  the shore map ([Map finishing](map-finishing.md)).
- **Rivers/lakes are generator entities** (river spline generators; lake generator from a closed
  polyline) — they place water surfaces + underwater volumes. ⚠️ **They do NOT carve the
  heightfield.** Carve the bed first (in the heightmap, or with the road-generator trick below),
  then let the generator fill it.
- **River recipe (community-proven):** draw the river spline, snap to terrain → temporarily
  attach a dirt-road generator with *Affect Height Map* → lower the spline 0.5–1 m → bake road to
  heightmap → delete the road generator → add the river generator → paint the riverbed surface.
- No fluid simulation: water placement is cosmetic + physics volumes. Use erosion/flow-accumulation
  masks to decide where water *should* run so it looks inevitable.
- Line-shaper generators (flatten/terrace along a spline with configurable width and falloff)
  are the precision tool for ditches, berms, and terraces.

## Vegetation

- **Forest Generator** (`FG_` prefabs) is the scalable tool: define a polygon, assign species
  mixes/densities, and it fills — with auto-avoid flags for objects, roads, power lines, rivers,
  lakes, and land/ocean selection. Its optimized clutter/instancing is what makes 100k+ tree
  forests performant. The generator area is drawn in the viewport (child of its shape).
- Manual placement is for hero trees and small clusters.
- **Density reality check (measured):** a closed-canopy boreal forest needs ~**6 m spacing**
  (≈ 41k trees on a small island's forest area). 11 m spacing reads as bare ground between trees
  from eye level. Match 3D density to what your satmap shows, or the map lies at distance.
- Species selection: match your climate archetype (vanilla flora is temperate/northern European —
  pines, spruces, birches, hazel, junipers…). There is no palm-tree equivalent; Mediterranean and
  tropical maps need asset packs or compromises.
- Undergrowth (ferns, berry shrubs, grass tufts) around settlements and forest edges sells scale;
  clutter (per-surface grass) comes from the surface definitions themselves.

## Settlements & props (brief)

- Study scale from real data: villages are *small* (a farm is 3–6 buildings; a village 10–40).
  Composition skews heavily residential (~⅔+).
- Setbacks: rural buildings sit back from roads; town centers front the street.
- Reforger building footprints are large — transfer *ratios* from real-world references, not
  absolute meter spacings, then re-multiply with actual prefab footprints.
- Place in order: terrain flatten → roads → buildings → walls/fences → props/clutter.
- Compositions (prefab groups) and the vanilla prefab library's `E_`-editable variants save
  enormous time — browse `Prefabs/Compositions/` and the PrefabLibrary before building your own.

## Bulk/programmatic placement

For mass placement from external data (procedural layouts, GIS-derived object sets), the official
**Object Import Tool** consumes CSV
(`"resourceHash" x y z quatX quatY quatZ quatW scale` — convert Euler yaw to quaternions). That is
the supported bridge from external pipelines into the editor; use positions from external tools as
a *density guide* for generators rather than importing 100k individual entities (each placed
entity is a streamed object; generators instance far cheaper).

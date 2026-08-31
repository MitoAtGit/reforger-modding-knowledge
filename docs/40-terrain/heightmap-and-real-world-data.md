# Heightmaps & Real-World Data

Getting elevation into Enfusion — from real DEMs or procedural generation.

## Import format & parameters

The terrain tool imports **`.asc` (ESRI ASCII grid) or 16-bit PNG**:

- 16-bit PNG needs **resample min/max** values on import (the real-world elevation range the
  0..65535 values map to). Record these numbers — you need them again for every re-import and for
  any external tool that must match world heights.
- Import defaults: **Invert Z checked** (image row 0 = north), Invert X unchecked. This defines
  your image↔world transform:
  `world_X = image_x_m` and `world_Z = world_size − image_z_m`. Apply the **same** orientation to
  satmap and surface masks or nothing lines up.
- After import the editor auto-opens normal-map generation (defaults are fine).
- Verify immediately: probe a few known points (summit, sea level) with the terrain height under
  the cursor / a placed entity. A wrong resample range shows up as a subtly wrong world.

## Real-world DEM workflow (QGIS/GDAL)

Proven path for real-terrain maps:

1. **Source DEM:** national LiDAR portals (1 m for small areas), or global sources
   (SRTM/ASTER ~30 m) for large maps.
2. **QGIS/GDAL:** clip to your area (projected CRS, meters!), resample to your target grid
   (`gdalwarp -tr <cellsize> <cellsize> -r cubic`), optionally smooth, then export both a
   Float32 GeoTIFF (archive/analysis) and the 16-bit PNG (import) with a **linear scale over a
   recorded min/max**.
3. Ocean: clamp sea areas to a below-zero constant so the ocean plane reads correctly.
4. Flatten construction zones **in the heightmap** (airfields, large compounds) before import —
   easier than in-editor sculpting and repeatable.
5. Keep the whole thing scripted (PyQGIS/GDAL CLI) so re-runs are one command.

QGIS quirks worth knowing: run Python via the OSGeo environment wrapper (the bare interpreter
mis-resolves its prefix), and GDAL's PNG driver can't `Create()` — go through `CreateCopy()` from
a MEM/GTiff dataset.

Community tools that package this: **ARTE** (QGIS plugin) and **GTT (GEO Terrain Tool)** export
DEM + satellite + OSM in Enfusion-ready parameters — see
[Official resources](../90-reference/official-resources.md).

## Procedural terrain

For fictional maps, generate the heightmap externally (noise + hydraulic/thermal erosion), then
import. Two practical notes:

- **Erosion is what sells realism** — flow/wear/deposit masks from the erosion pass double as
  input for surface masks (rock on wear zones, sediment in deposits) and river routing.
- Dedicated terrain tools (Gaea, World Machine…) export 16-bit heightmaps directly; only
  heightmaps/masks transfer between engines — never scenes or materials.

## Terrain modification in the editor

- Sculpt tab for local fixes; roads with *Affect Height Map* imprint themselves
  ([Roads, water, vegetation](roads-water-vegetation.md)).
- Building placement auto-flattens locally under footprints ("Bake Selection" for the rest).
- Terracing beats tilting for structures on >9° slopes
  ([World Editor basics](../00-getting-started/world-editor-basics.md)).
- The heightmap is **immutable at runtime** — no gameplay system can dig terrain while playing;
  design trenches/berms as placed objects instead.

## Slope & hydrology sanity (before you commit)

From the analysis side, cheap checks that predict whether the map will "work":

- Slope map: buildable land (<6–10 % slope) should exist where you plan settlements.
- Flow accumulation (D8): your rivers should follow the accumulation channels, not fight them.
- Coastline at your chosen sea level should look like a coastline at map scale — fix the DEM,
  not the shore, if it doesn't.

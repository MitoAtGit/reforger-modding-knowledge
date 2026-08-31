# Map Finishing

The last 10 % that separates "terrain demo" from "playable map".

## Shore map

Terrain tool → Manage → **Build shore map**. Generates the shore data + foam atlas the
ocean/shore simulation needs. Symptom of a missing one: a log error about the world's `_SMD.conf`
failing to load. Rebuild after coastline-affecting heightmap changes.

## In-game 2D map

The paper map is assembled from:

1. **MapEntity** configured on the world.
2. **Map geometry** — *Export Map Data → Export Geometry 2D* produces the `.topo` the map
   renders (contours, roads, forests as symbols).
3. **Satellite background** — *Export Map Data → Rasterization* renders a top-down image
   (a 4096² TGA; note it comes out flipped — flip it back), convert to a texture in your project
   and reference it as the map background. This rasterization is a deliverable of its own: it's
   also the correct source for exterior top-down documentation shots.

Without the `.topo` export the in-game map shows no roads/features even though the world has them.

## Navmesh (AI)

- The world needs an AI world entity (`SCR_AIWorld`) + a `NavmeshWorldComponent`; the navmesh
  builds into an `.nmn` resource.
- Rebuild after placing/removing large structures; runtime rebuild requests around dynamic
  buildings exist (`RequestNavmeshRebuildEntity/Areas`) — relevant for construction gameplay.
- Water is hard-impassable for AI. Bridges need valid navmesh; test AI pathing across every
  bridge and through every village before release.

## Minimap/world registration details

- World lat/long and year settings drive the sky, sun path, and map grid — set them to your real
  region for believable light.
- Test the map screen: background aligned with geometry, grid readable, your locations named
  (map descriptor components on location entities).

## Performance pass

- Prefer generators (instancing) over individually placed vegetation.
- Watch entity counts: hundreds of thousands of individually placed entities load, but cost
  memory and streaming — split content into layers by region and type; keep heavy decoration
  near playable areas.
- LODs and occluders on custom buildings; oversized FireGeo is a silent CPU tax
  ([Collision & LODs](../10-assets/collision-and-lods.md)).

## Believability checklist (12 points)

Walk the map and score honestly:

1. Water flows downhill everywhere (no uphill rivers, no lakes on slopes).
2. Coastline reads as a coastline at map scale; beaches only where waves would make them.
3. Vegetation density matches the satmap from the air **and** feels closed at eye level.
4. Tree line / species change with elevation and exposure where the map is tall enough.
5. Settlements sit where people would build (flat, water access, sheltered, arable surroundings).
6. Every settlement is reachable; road classes form a hierarchy (main → secondary → tracks).
7. Roads follow terrain logic (contours, passes, fords/bridges at sensible points).
8. Field/forest boundaries look owned (straight fence lines, walls, hedgerows), not noise-blobby.
9. Surfaces match use: paths worn to dirt, farmyards mud/gravel, village greens grass.
10. Landmarks exist for orientation (church tower, water tower, masts, distinctive hills).
11. No floating/buried objects on slopes (inspect from downhill).
12. The 2D map, satmap, and 3D world tell the same story.

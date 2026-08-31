# Glossary

The vocabulary of Reforger/Enfusion modding, in one place.

| Term | Meaning |
|---|---|
| **Addon / mod project** | A folder under `…\ArmaReforgerWorkbench\addons\` with an `addon.gproj` manifest. |
| **`addon.gproj`** | Project manifest: ID, GUID, TITLE (Workshop name), dependencies, platform configurations. |
| **BIKI** | The Bohemia Interactive Community Wiki — the official documentation. |
| **Prefab (`.et`)** | Entity template: an entity class + component configuration. Supports inheritance (child stores only deltas). |
| **Entity** | A world object instance. Root classes: `GenericEntity` → game entities (Vehicle, Character, …). |
| **Component** | Modular behavior attached to entities. Script components come as a two-class pair (`XComponentClass` + `XComponent`). |
| **Instance GUID** | The per-component identity inside a prefab; child prefabs override inherited components **by this GUID**. |
| **Resource GUID** | The identity of a file/resource, stored in its `.meta`. References resolve by GUID, not path. |
| **`.meta`** | Sidecar file per resource: GUID + per-platform import configuration. |
| **`.conf`** | Serialized config object (an instance of a `[BaseContainerProps]` script class). |
| **`.xob`** | Compiled model (from your FBX). Contains LODs, colliders, memory points, skinning. |
| **`.emat`** | Visual material: shader class + parameters + texture references. Text file. |
| **`.edds`** | Compiled texture (from your PNG/TGA/TIF). |
| **`.gamemat`** | Physical surface material: ballistics, penetration, sounds, effects. |
| **`.layout`** | UI widget tree. |
| **`.acp`** | Audio project — node graph authored in the Audio Editor. |
| **`.anm` / `.txa`** | Compiled animation clip / the text animation format EBT exports (engine compiles txa→anm). |
| **`.agr` / `.asi` / `.ast`** | Animation graph / instance / set-template — the animation wiring authored in the Animation Editor. |
| **`.bt`** | AI behavior tree (authored in the Behavior Editor). |
| **`.ent`** | A world (binary; author in the World Editor). |
| **`.layer`** | Part of a world's entity content (`<World>_Layers\*.layer`); worlds are organized in layers. |
| **Memory point** | A named locator in a model (from an empty in Blender): `eye`, `barrel_muzzle`, `slot_optics`, `snap_hand_right`… Referenced by prefabs via **PivotID** (exact-name match). |
| **Slot / EntitySlotInfo** | The universal attach mechanism: a slot on a parent spawns/holds a child prefab at a memory point. |
| **Attachment type** | Marker class (`: BaseAttachmentType`) filtering what fits into an attachment slot; matching is by inheritance. |
| **Magazine well** | Marker class (`: BaseMagazineWell`) matching magazines to weapons. |
| **Layer preset / `usage`** | The collision-layer assignment of a collider (Weapon, FireGeo, VehicleSimple…), set as a custom property on export. |
| **FireGeo** | Fire geometry — the collider projectiles (and inventory raycasts) test against. |
| **LOD** | Level of detail (`_LOD0…`); Enfusion switches automatically by distance. |
| **Satmap** | The terrain's distant albedo texture ("satellite map"); import-only, authored externally. |
| **Surface (terrain)** | A paintable ground material = visual `.emat` + physical `.gamemat`, weighted by masks; max 5 per terrain block. |
| **Generator** | Shape-driven world tool (forest, road, wall, lake) living as a child of a spline/polygon shape. |
| **Composition** | A prefab grouping multiple entities (e.g. a building with props) placed as one. |
| **GM / Game Master** | The in-game real-time editor game mode. |
| **Conflict** | The flagship capture-and-supply game mode (ranks + supply economy live here). |
| **Workbench** | The Enfusion editor suite (World/Script/Prefab/Model/Audio/Animation/Behavior editors). |
| **EBT** | Enfusion Blender Tools — official Blender addon (export, animation TXA, QA). |
| **World Editor (WE)** | The Workbench module for worlds/terrain. |
| **Navmesh (`.nmn`)** | The AI navigation mesh, built per world. |
| **RplComponent / replication** | The multiplayer sync system; server-authoritative. |
| **EnforceScript** | The engine's scripting language (`.c` files). |
| **`modded class`** | The sanctioned mechanism to override/extend a vanilla script class. |
| **Diag menu** | In-game debug overlay menu (`Win+Alt`/`Win+Ctrl`) with vehicle/AI/engine visualizations. |
| **APL / APL-SA / APL-ND** | Arma Public License family — common licenses for community content. |

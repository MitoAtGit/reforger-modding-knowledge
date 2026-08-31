# Mod Project Scaffolding

What an Arma Reforger addon actually is, and the rules that keep it loadable.

## Anatomy of a project

Create projects via Workbench → **New Project**. You get:

```
Documents\My Games\ArmaReforgerWorkbench\addons\MyMod\
├── addon.gproj          ← project manifest (ID, GUID, TITLE, Dependencies, Configurations)
├── Assets\              ← models (.xob compiled from your FBX), textures (.edds from PNG/TGA), data
├── Prefabs\             ← .et entity templates (weapons, vehicles, props…)
├── Configs\             ← .conf files (catalogs, input, gamemode configs…)
├── Scripts\Game\        ← EnforceScript .c files (game module)
├── Scripts\WorkbenchGame\ ← editor-only scripts (Workbench plugins) — never loaded in game
├── Sounds\              ← .wav + .acp audio projects
├── UI\                  ← .layout files, imagesets, fonts
└── Worlds\              ← .ent worlds + <World>_Layers\*.layer
```

Follow the official **Directory Structure** conventions from the BIKI
([Arma_Reforger:Directory_Structure](https://community.bistudio.com/wiki/Arma_Reforger:Directory_Structure)) —
several official automation plugins parse paths by convention, and future-you will thank you.

## `addon.gproj` — the manifest

Key fields (all editable as text):

- `ID` — internal folder-style identifier.
- `GUID` — the project's unique identity. Never reuse another project's GUID.
- `TITLE` — **this is the name shown in the Workshop**, not the folder name. New projects carry a
  placeholder here; fix it before publishing.
- `Dependencies` — list of addon GUIDs. The vanilla game data is the base dependency every mod has.
- `Configurations` — platform blocks (`PC`, `XBOX_ONE`, `XBOX_SERIES`, `PS4`, `PS5`, `HEADLESS`).
  ⚠️ If your mod refuses to load on a **dedicated server**, a missing `HEADLESS` configuration is
  the first suspect.

## GUIDs and `.meta` files — the identity system

Every resource (every `.et`, `.conf`, `.emat`, texture, model…) has a sidecar `.meta` file holding
its **GUID** and per-platform import configuration. Understand these rules and half of all
"broken reference" bugs disappear:

1. **References resolve by GUID, not by path.** The path in `"{GUID}Path/To/File.et"` is a
   human-readable hint; the GUID is what the engine actually loads.
2. **Prefab-to-prefab references need real GUIDs.** A reference written with a path but a zero/no
   GUID is treated as `GUID=0` → *"Broken resource GUID=0"* in the log, silent failure in game.
3. **GUID + path must stay a consistent pair.** Renaming/moving is fine *inside Workbench* (it
   updates metas); doing it in Explorer desyncs them.
4. **Never copy a file together with its `.meta` into the same loadable project twice** — duplicate
   GUIDs cause collisions and "wrong resource wins" bugs. Corollary: two mods that share copied
   GUIDs must never be loaded at the same time.
5. Re-exporting/overwriting a file **at the same path keeps its GUID** (the meta survives). This is
   the safe way to update an asset without breaking references.
6. Getting a GUID: right-click the resource in the Resource Browser → **Get Resource GUID(s)**.

## Naming rules (hard-earned)

- **ASCII only.** Non-ASCII file/material names (e.g. Chinese characters from a bought model)
  break resource registration — the files import but every reference to them fails with
  *"Wrong GUID/name"*. Rename to ASCII **keeping the GUIDs** (edit the meta `Name` path, keep the
  GUID) and references heal.
- No spaces or special characters in asset names; the importer replaces `/\<>:"` with `_`.
- Blender's `.001` duplicate suffixes are poison: on materials the dot is stripped on import
  (name collisions), and on memory points the engine's exact-name lookups silently miss
  (`eye.001` ≠ `eye`). Clean them before export. See
  [FBX import rules](../10-assets/fbx-import-rules.md).
- Pick a short unique **prefix for your script classes and files** (e.g. `ABC_`). Class names are
  global across all loaded mods — two mods defining the same class collide.

## Inheritance is the core workflow

You almost never build entities from scratch. You **inherit** from a vanilla base prefab and
override only what differs:

- Resource Browser → right-click a vanilla prefab → **Inherit in project** creates a child `.et`.
- The child stores *only deltas*. Overriding a component means overriding **the inherited component
  instance (by its instance GUID)** — creating a second component of the same class instead
  produces *"component X cannot be combined with component X"*.
- ⚠️ **Create child prefabs via the Workbench UI**, not by hand-writing the parent line — the UI
  embeds the parent's GUID; a hand-written path-only parent won't load.
- Same for equipment: derive from the closest matching vanilla item (helmet, vest…) via
  **Duplicate to project** — a standalone hand-built item misses invisible essentials (faction
  affiliation, correct item component subclass) and e.g. never shows up in arsenals.

## Loading, testing, iterating

- Only projects registered in the Workbench launcher can be used as dependencies.
- Dependencies must be added in topological order (a dependent before its dependents errors with
  "can't be added").
- Script iteration loop: edit file → recompile (focus/`Reload Scripts`) → check newest
  `console.log` for `SCRIPT (E)`.
- ⚠️ A **running play session keeps the old compiled scripts and parsed prefabs**. After edits:
  stop Play, start fresh. Confirm a new `Module: Game … CRC32` line appeared before trusting a test.
- ⚠️ Workbench caches parsed prefabs in RAM — an edited `.et` may not be re-parsed on respawn.
  When iterating on prefab files externally, watch for the `Entity prefab load` line, or use new
  file names during experiments.
- ⚠️ Workbench autosave can overwrite your on-disk fixes if the same prefab is open in an editor.
  After any Workbench interaction, re-check files you edited externally.

## Related

- [Config overrides](../50-scripting/config-overrides.md) — the GUID-override mechanism for
  vanilla configs.
- [Troubleshooting](../90-reference/troubleshooting.md).

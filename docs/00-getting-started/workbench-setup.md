# Workbench Setup

The **Enfusion Workbench** ships free with Arma Reforger as **"Arma Reforger Tools"** on Steam.
It is the one official tool for everything: importing assets, editing prefabs, building worlds,
scripting, audio, animation, and publishing.

## Install

1. Steam → Library → search **Arma Reforger Tools** → install (it appears under *Tools*).
2. Launch it once — it creates your user folders and registers itself.
3. Optional but recommended: also install the official **Enfusion Blender Tools (EBT)** from the
   Tools installation (`Arma Reforger Tools\Blender\`) — see
   [Blender pipeline](../10-assets/blender-pipeline.md) for version caveats.

⚠️ **Stable vs Experimental Tools:** Steam can install both "Arma Reforger Tools" and
"Arma Reforger Experimental Tools". They are different versions. Always match your Tools version
to the game data you are working against, otherwise you chase phantom errors. Check the exe
version if unsure.

## Where things live

| Thing | Path |
|---|---|
| Your mod projects | `Documents\My Games\ArmaReforgerWorkbench\addons\<YourMod>\` |
| Workbench logs | `Documents\My Games\ArmaReforgerWorkbench\logs\logs_<timestamp>\` |
| Installed Workshop mods (game side) | `Documents\My Games\ArmaReforger\addons\` |
| Official Script API docs (offline) | `<Tools install>\Workbench\docs\` |

**Each Workbench session writes a new `logs_<timestamp>` directory** containing `console.log`,
`error.log`, and `script.log`. Starting a play session creates a new directory too.

⚠️ Always read the **newest** log directory. Reading a stale one makes it look like "nothing
happened" — a classic self-inflicted debugging dead end. The active log can be file-locked; open
it with a shared-read method (e.g. PowerShell `Get-Content`), not an exclusive one.

## Reading logs — the debugging entry point

- Errors are tagged `(E)`, warnings `(W)`. Script compile errors look like
  `SCRIPT (E): @"path/File.c,LINE"`.
- After one real script error, the compiler cascades ("Can't find class X" everywhere). **The
  first error from your own files is the real one**; the rest are fallout.
- A successful script compile logs `Module: Game; loaded Nx files ... CRC32: ...`. The file count
  increases by one for every new script file — a cheap way to prove a new file was actually
  picked up.
- Useful dedup scan for a noisy `error.log`:
  ```bash
  grep "(E)" error.log | sed 's/^[0-9:.]* *//' | sort | uniq -c | sort -rn
  ```

## The file watcher

The Workbench watches your project directory:

- Saving a script file from any editor triggers a recompile on focus/scan.
- Dropping an FBX or PNG into the project triggers an automatic import
  (`Rebuilding <file> ... Build successful` in the log).
- ⚠️ A pure timestamp *touch* does **not** trigger a script rebuild — it needs a real content
  change. FBX reimports, on the other hand, key off the FBX being newer than its compiled output.

## Editors inside Workbench

- **World Editor** — worlds, terrain, entity placement ([basics](world-editor-basics.md)).
- **Resource Browser** — every registered resource; right-click is powerful
  (*Get Resource GUID(s)*, *Register and Import*, *Duplicate to project*, *Inherit in project*).
- **Script Editor** — EnforceScript editing + compile; vanilla script sources are readable here,
  which is the officially supported way to learn from vanilla code.
- **Prefab Editor / Model Editor / Audio Editor / Animation Editor / Behavior Editor** — dedicated
  editors; some data (audio node graphs, animation graphs) can only be authored there, not by
  hand-editing files (see [Custom sounds](../20-weapons/custom-sounds.md) for why).

## Related

- [Mod project scaffolding](mod-project-scaffolding.md) — what a project actually consists of.
- [Troubleshooting](../90-reference/troubleshooting.md) — symptom-indexed fixes.

# Workbench Plugins & Generators

Extending the editor itself: custom tools, batch operations, and shape-driven world generators.

## Workbench plugins

Editor-side scripts live under `Scripts/WorkbenchGame/` — a **separate module** from game scripts
(they never load in game; a broken game-script compile doesn't take plugins down, and vice versa).

Skeleton:

```c
#ifdef WORKBENCH
[WorkbenchPluginAttribute(name: "My Tool", wbModules: {"WorldEditor"}, category: "MyMod",
                          awesomeFontCode: 0xf0ad)]
class MyMod_ToolPlugin : WorldEditorPlugin
{
    [Attribute(defvalue: "10", desc: "How many")]
    int m_iCount;

    [ButtonAttribute("Run")]
    void Execute() { /* runs immediately on click */ }

    override void Run()
    {
        Workbench.ScriptDialog("My Tool", "Description", this);  // shows the attribute dialog
    }
}
#endif
```

- Base classes per module: `WorkbenchPlugin`, `WorldEditorPlugin`, `ScriptEditorPlugin`
  (+ resource context-menu hooks via `OnResourceContextMenu`).
- World manipulation through **WorldEditorAPI**: wrap edits in
  `BeginEntityAction`/`EndEntityAction`, create entities with `CreateEntityExt`, set fields with
  `SetVariableValue` (don't wrap enum values), manage arrays via the object-array calls.
- Files: `$profile:` / `$Addon:` path prefixes, `FileIO.MakeDirectory`; load config presets via
  `BaseContainerTools.LoadContainer` → `CreateInstanceFromContainer`.

⚠️ Practical facts:

- **WorkbenchGame scripts compile at Workbench start** — a hot script reload does *not*
  re-register plugin/handler classes reliably. After adding a plugin: restart the Workbench.
- Plugins run in a specific editor context — the World Editor API is unavailable in game mode and
  in other editors (`wb state` symptom: "WorldEditorAPI not available" when e.g. the Model Editor
  has focus).

## Shape-driven generators

The pattern behind forest/wall/road generators — and yours:

```c
class MyMod_FenceGeneratorEntity : GeneratorBaseEntity
{
    override void OnShapeChangedInternal(ShapeEntity shapeEntity, ...)
    {
        // regenerate: WorldEditorAPI is allowed here, but do NOT open your own
        // Begin/EndEntityAction around it
    }

    override void OnShapeInitInternal(ShapeEntity shapeEntity)
    {
        // called on load: NO WorldEditorAPI here
    }
}
```

- A generator entity must be the **direct child of its ShapeEntity** (spline or polygon area).
- Point-change callbacks tell you what changed (moved/added/removed) so incremental regeneration
  is possible.
- Vanilla references to study in the editor: forest generator, wall generator, road generator,
  lake generator.

## Batch/world tooling patterns

- **Layers are your bulk interface.** Entity layers are text files with a stable structure —
  generating content into a fresh layer (from your own external tools) and loading it in the
  editor scales to tens of thousands of entities where hand-placement can't. Author the format by
  example: place one entity of the kind you need, save, and read what the editor wrote. Keep
  backups; reload layers rather than fighting the editor for a file that's open.
- The official **Object Import Tool** (CSV positions + quaternions) is the supported bulk-import
  path for externally computed placements
  ([Roads, water, vegetation](../40-terrain/roads-water-vegetation.md)).
- Prefabs referenced from scripts/tools need **GUID-qualified** resource names — path-only
  references create empty entities with no model.

## Editor automation limits (honest notes)

- Some editor state is UI-only: certain dialog-driven imports, animation-graph authoring, and
  audio-graph authoring have no scriptable path — plan for a human in the loop there.
- Terrain surface *painting* has no working script API (height and shore map do) — masks via the
  import dialog are the automation path ([Surfaces & satmap](../40-terrain/surfaces-and-satmap.md)).
- Modal dialogs block everything until answered — automation that triggers an import must expect
  the "Register Resource File(s)" dialog and stop for it.

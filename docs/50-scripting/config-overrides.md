# Vanilla Config Overrides

The mechanism behind "add my keybind / my arsenal item / my input action to the game" — and the
two traps that make overrides silently do nothing or wipe vanilla content.

## How overriding works: GUID, not path

A mod config overrides/merges a vanilla config **only** when the mod file's `.meta` carries the
**vanilla file's GUID** (same path is convention; the GUID is what counts).

- A copied config with a *fresh* GUID at the same path is a **dead resource** — nothing references
  it, nothing loads it. Symptom: "my input action does nothing", "my item never appears".
- Getting the vanilla GUID: Resource Browser → find the vanilla config → right-click →
  **Get Resource GUID(s)**. Cleanest workflow: right-click → **Duplicate to project** — the
  Workbench sets up the override meta for you.
- The `.meta` must also use the correct resource class (`CONFResourceClass` with all platform
  configurations) — a wrong class also kills the override.

Configs load at **game start** — after changing an override, restart the play session/Workbench.

## ⚠️ The `+` merge rule (the vanilla-wipe trap)

When your override touches a **list**, the `+` decides between *append* and *replace*:

```
m_aMultiLists +  { ... }     // appends your entries to vanilla's   ✅
m_aMultiLists    { ... }     // REPLACES the whole vanilla list     ⚠️
```

Real-world consequences of a missing `+`:

- Arsenal catalog override without `+` → the faction's arsenal contains **only your item**.
- `Actions { }` (empty, no `+`) in an input context → **deletes all vanilla actions** in that
  context.
- Key-binding menu categories without `+` → movement/weapon/vehicle keybind pages vanish from
  the options menu.
- Same on prefab component lists (`additionalActions +`, `ActionRefs +`).

Rule of thumb: **when adding to anything vanilla, write the `+`.** Only omit it when you truly
mean "replace everything".

## Standard override targets

| Config (path under `Configs/System/` or `Configs/EntityCatalog/`) | Purpose |
|---|---|
| `chimeraInputCommon.conf` | define input **actions** + bind default keys (contexts, `ActionRefs +`) |
| `keyBindingMenu.conf` | expose your actions in Settings → Keybinds (categories with entries; `"separator"` rows make headers; plain display text works) |
| `ControlHints/AvailableActions.conf` | on-screen control hints |
| `EntityCatalog/<Faction>/InventoryItems_EntityCatalog_<Faction>.conf` | faction arsenals ([Arsenal & catalogs](../60-publishing/arsenal-and-catalogs.md)) |

## Input actions in practice

1. Override `chimeraInputCommon.conf` (vanilla GUID!), add your action with a default key, attach
   it to the right context (e.g. the character context) via `ActionRefs +`.
2. Script side: `GetGame().GetInputManager().AddActionListener("MyAction", EActionTrigger.DOWN, Callback);`
   Remove listeners in `OnDelete`.
3. ⚠️ Key codes are **US physical positions** — `KC_EQUALS`/`KC_MINUS` don't exist on many
   layouts (dead keys on QWERTZ). Prefer position-stable keys (letters, `KC_PERIOD`/`KC_COMMA`).
4. ⚠️ A single-key action also fires while a modifier is held — pressing Shift+N triggers both
   "N" and "Shift+N" actions. Disambiguate with a 1-frame `CallLater`, not by polling modifiers.
5. Diagnose "key does nothing": is your action config's meta carrying the vanilla GUID?
   That's the cause in nearly every case. After fixing metas, restart the Workbench (live
   re-registration is unreliable).

## Overriding *component* config inside inherited prefabs

Related but different: inside a child prefab you override inherited components **via their
inherited instance GUIDs** (see
[scaffolding](../00-getting-started/mod-project-scaffolding.md#inheritance-is-the-core-workflow)).
Same idea — identity wins over position — applied at the component level: writing a *new* UIInfo
block instead of overriding the inherited one gives you a second, ignored block.

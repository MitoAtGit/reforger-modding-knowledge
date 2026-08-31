# Workshop Publishing

Getting a mod onto the Reforger Workshop — and the pre-flight checklist that saves you from
shipping embarrassments.

## The mechanics

- Publishing happens in **Workbench → Addon Manager → Publish**, signed in with your Bohemia
  account. There is no file-based or CLI upload path.
- **Title:** the `TITLE` field in `addon.gproj` is the displayed Workshop name (not the folder).
  New projects carry a placeholder — set it.
- **Description/summary live server-side** — entered in the publish dialog, not stored in any
  project file. Draft them in a text file beforehand and paste.
- **Platforms:** the `Configurations` block in `addon.gproj` should cover
  `PC, XBOX_ONE, XBOX_SERIES, PS4, PS5, HEADLESS`. ⚠️ Missing `HEADLESS` = the mod won't load on
  dedicated servers.
- Versioning/updates re-publish through the same dialog; keep your own changelog.

## Pre-publish checklist

- [ ] `TITLE` correct; description/summary drafted.
- [ ] **Remove dev tooling** — editor-bridge scripts, debug/dumper components, experiment
  prefabs. WorkbenchGame-module scripts never load in game, but shipping them is sloppy.
- [ ] Remove diagnostic `Print()` spam from released scripts.
- [ ] **Study-dependencies removed.** If you temporarily added other mods as dependencies for
  reference/testing, revert `Dependencies` to only what the release truly needs. Dependencies you
  ship become hard requirements for every subscriber.
- [ ] Fresh session, full playtest, then scan the newest log: zero `(E)` from your files, no
  `Wrong GUID/name`, no `Broken resource`.
- [ ] Content licensing verified: every asset (models, textures, sounds) is yours, official
  sample content, or licensed for redistribution. "Found it in another mod" is not a license —
  bundling someone else's content needs their **explicit permission**, and conserved-GUID bundles
  must never be loaded together with the source mod (class/GUID collisions).
- [ ] No leftover overrides that nuke vanilla content (search your configs for lists missing
  their `+` — see [Config overrides](../50-scripting/config-overrides.md)).
- [ ] Test with your mod as the **only** enabled mod (catches hidden dependencies) and once with
  a popular mod set (catches collisions).

## Validation false alarms (known)

Automated validators (and some tooling) report issues that are not real:

- *"Script is outside a valid module folder"* for every script — case-sensitivity confusion
  (`Scripts/` vs `scripts/`); Windows doesn't care, the game compiles them. Ignore.
- *"Class X not found in API index"* for vanilla classes — the validator's index simply doesn't
  include the vanilla API. Ignore.
- The reliable success signal remains the game/Workbench log:
  `Module: Game; loaded Nx files … CRC32` with your expected file count.

## Vanilla log noise ≠ your bug

Some warnings/errors appear in every session and come from vanilla content (empty HUD slot
warnings, an inventory-menu widget-slot error that fires while the inventory is open…). Method to
attribute a mystery log line: correlate its occurrences with events (menu open/close), and grep
your own content for the named widget/class — if it's not yours and fires without your mod,
stop chasing it.

## Licensing your own mod

Pick a license deliberately (many Reforger mods use Arma Public License variants — APL-SA/APL-ND).
Understand what you're granting: ND forbids derivative re-uploads; SA requires share-alike.
Respect the same in reverse: another author's license (or lack of one) binds you.

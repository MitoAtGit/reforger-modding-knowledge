# UI & HUD

Building HUD overlays, custom displays, and post-process effects — the architecture plus the
coordinate-system traps that cost the most time. Everything verified in shipped-quality mods
(NVG helmet HUDs, scope FCS overlays).

## Architecture

- **Layouts** (`.layout`): widget trees authored in the Workbench layout editor (or as text).
- **InfoDisplays**: subclass `SCR_InfoDisplayExtended`, register it in the
  `SCR_HUDManagerComponent` list on the player-controller prefab (override the inherited list —
  with the identity rules from [Config overrides](config-overrides.md)). Lifecycle:
  `DisplayStartDraw → DisplayUpdate(owner, timeSlice) → DisplayStopDraw`, plus
  controlled-entity-changed hooks. The HUD manager's slot system re-parents displays into menus.
- **Item-driven HUDs** (a helmet/optic drawing UI): a `ScriptComponent` on the item that creates
  widgets via `GetGame().GetWorkspace().CreateWidgets(layout)` and updates on a **call-queue
  timer** (~30 Hz) — frame events don't fire on carried items
  ([components](enforce-script-and-components.md)).
  Gate on the wearer: only drive the local camera when the item's parent IS the controlled entity,
  or every player wearing one toggles *your* screen.

## Canvas drawing (procedural graphics)

`CanvasWidget.SetDrawCommands` with `LineDrawCommand` & friends draws crosshairs, compasses,
brackets, minimaps — no textures needed.

- ⚠️ **The command array must be a class member.** The canvas keeps only a reference — a local
  array is garbage-collected before rendering and you see *nothing*.
- ⚠️ **Canvas draws in PHYSICAL pixels; widget slots use reference/GUI units.**
  `ProjWorldToScreen` returns GUI units → `ws.DPIScale(x)` before drawing on canvas;
  `ws.DPIUnscale(px)` before `FrameSlot.SetPos`. Mixing them = everything offset by the DPI
  factor toward the top-left.
- World→screen: `ws.ProjWorldToScreen(worldPos, world[, cameraIndex])`; **in front of camera =
  result[2] > 0** — skip points behind the camera.
- The optical screen center ≠ canvas center (workspace aspect can differ from render aspect).
  For anything that must align with the aim point, project a point on the camera axis
  (`camPos + forward × 100`) and use *that* as center.
- Text widgets: slots are size-to-content by default — set `FrameSlot.SetSizeToContent(w, true)`
  or long strings clip at both ends. Center by placing with your own width estimate; position
  labels **from script relative to your canvas drawing** (with DPIUnscale), never by static
  layout anchors, if they must align with drawn graphics.
- Draw-order: a masking/crop canvas must sit **above the canvases it crops but below text** —
  as the last child it eats your text widgets.

## Post-process effects (NVG, thermal, screen tints)

The whole night-vision genre is a **camera post-process, not a mesh effect**:

- `world.SetCameraPostProcessEffect(camId, priority, PostProcessEffectType.X, ematPath)` — HDR
  color grading on one priority slot, film grain on another (vanilla scope effects occupy their
  own slots; don't collide with the ones the base class uses).
- Reset = re-apply the vanilla HDR emat + effect type None + `SetCameraHDRBrightness(camId, -1)`.
- Get the camera id fresh per toggle (`world.GetCurrentCameraId()`) — cached ids go stale on
  camera switches.
- The look lives in `.emat` files (HDREffect: colorization, contrast, filmic curve, vignette,
  scope masks/reticle maps; FilmGrainEffect: intensity/sharpness) — tune without recompiling.
  ⚠️ Avoid LUT color-table grading for NVG — LUTs crush bright skies into blotchy artifacts;
  pure colorization + filmic curve + zero saturation is the clean recipe.
- Item removal: hook the character inventory storage (modded) to reset effects when the item is
  unequipped — otherwise the effect sticks after dropping the helmet.

## PIP / render-target cameras (scopes, drone feeds, mirrors)

- A `RenderTargetWidget` renders a **camera slot of the world**, not a camera entity. Drive the
  slot directly: set FOV/near/far/type once, then `world.SetCameraEx(idx, transform)` per tick;
  wire the widget with `SetWorld(world, idx)` + resolution/refresh settings.
- Diagnosis trick: `SetClearColor(true, magenta)` on the widget — magenta visible = widget fine,
  camera slot empty; nothing visible = widget/PIP-settings problem.
- PIP cameras meter exposure independently (night = black): copy the main camera's HDR brightness
  to the slot per tick, and apply your post-process stack to that camera index too.
- Release slots on close (reset post-process on the index). Slot indices are limited (0–31);
  pick a high one to avoid vanilla users.
- 2D/PIP sights specifics (zeroing type, scope HDR material inheritance):
  [Attachments](../20-weapons/attachments.md) + [Ballistics & zeroing](../20-weapons/ballistics-and-zeroing.md).

## Real game map in custom UI

`SCR_MapEntity.SetupMapConfig(EMapEntityMode.PLAIN, mapPlainConfig, rootWidget)` + `OpenMap(cfg)`
can host the actual map in your HUD panel. Traps: it disables the character camera render
(re-enable it), the host panel must have real size before opening (else division-by-zero in zoom
init), strip menu-context modules from the config, `WorldToScreen` is physical while pan APIs
expect GUI units (DPIUnscale between), and it's a singleton — the M key map takes over.
Markers: `SCR_MapMarkerManagerComponent.GetInstance()` exposes static/dynamic markers with 2D
world positions (`GetWorldPos`), type and text — project them into your HUD like any world point
(surface Y + offset for height).

## Iteration reality

- Script changes: recompile + **fresh play session**.
- Layout/emat/texture changes: resources reload on session restart (or resource rebuild) — a
  byte-identical screenshot after your "fix" means the old build is still running.
- Textures for UI: drop a PNG in the project and let the watcher import it; read the generated
  GUID from the meta. If a texture shows as a white quad, its compiled form didn't build — check
  the import log before suspecting your code. Procedural canvas drawing avoids the whole issue.

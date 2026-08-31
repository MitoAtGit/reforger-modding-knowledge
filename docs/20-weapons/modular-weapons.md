# Modular Weapons

Building weapons where handguard, stock, grips, muzzle devices, optics — even rails on
handguards — are swappable parts. All of the below was proven with **vanilla mechanisms only**
(no framework dependency).

## What vanilla gives you

The vanilla attachment system covers more than people assume:

- `InventoryStorageSlot`-based attachment slots on the weapon (the same mechanism vanilla uses
  for optics/muzzle) work for **any** part category.
- Custom attachment *types* are one-line marker classes:

  ```c
  class MDT_AttachmentPistolGrip : BaseAttachmentType {}
  class MDT_AttachmentForegrip   : BaseAttachmentType {}
  ```

  Slot filters accept an item whose type equals or derives from the slot's type
  ([Attachments](attachments.md)).
- `ShowInInspection` displays slots as points in the in-hand inspection (R).
- A weapon-stats manager component recomputes stats from attached parts.

## Mesh design: plan the slot topology first

Decide **which part hosts which slot** before modeling:

- Weapon body: `slot_Handguard`, `slot_stock`, `slot_grip`, `slot_optic`, `slot_magazine`
  (+ fire points).
- Handguard meshes: a `snap_handguard` (docks onto the body) **and their own sub-slots**
  (`slot_underbarrel`, `slot_barrel_muzzle`, `slot_flashlight`) — because foregrips/muzzle
  devices/lights belong to the *handguard*, not the body.
- Each attachment mesh: one `snap_*` point that docks onto the host's slot.

⚠️ `ChildPivotID` matching is **exact and case-sensitive** (`snap_Stock` ≠ `snap_stock`); a
mismatch silently snaps the part to the scene root. Extract the real point names from your
imported model and wire slots accordingly.

## Nested slots — proven by config, no scripting

**An `AttachmentSlotComponent` directly on the handguard prefab works**, even though the
handguard has no WeaponComponent — its sub-slot spawns its default child correctly
(log: *"spawned by an EntitySlotInfo in Handguard_x.et"*). That makes rail-host handguards a pure
config exercise:

```
Weapon body
└── Handguard slot (type AttachmentHandGuard) → Handguard prefab
    ├── Underbarrel slot (type MDT_AttachmentForegrip) → Foregrip prefab
    └── Muzzle slot (type AttachmentMuzzle…)           → …
```

## Attachment prefab essentials

Inherit a close vanilla attachment base (it provides RplComponent, Hierarchy, and the
equip/pickup actions) and override mesh + type + name. Building standalone loses replication and
actions. Full checklist in [Attachments](attachments.md).

⚠️ When adding actions to an attachment: `additionalActions +` — **without the `+` you replace
the inherited list** and lose equip/pickup.

## Sliding parts on rails (advanced)

A repositionable foregrip/optic along a rail is scriptable with vanilla base classes:

- A custom slot class (`: InventoryStorageSlot`) that stores a rail position and applies it via an
  additive local transform on the attached entity (translate along the slot's local axis, clamped
  to limits), updating on attach.
- A user action (`: SCR_AdjustSignalAction`) exposed in the inspection state
  (`CanBeShownScript` gated on the character's inspect mode) that steps the position; fully
  replicated via the action's save/load hooks.
- ⚠️ **`[Attribute]` fields on an `InventoryStorageSlot` subclass may not deserialize from the
  prefab** (log: `Unknown keyword/data 'm_fMyField'`). Robust alternative: put the tunables in a
  small `ScriptComponent` on the attachment/weapon (an "offset component" that exposes
  `GetOffset()`), and let the slot read it.

Note on expectations: in-hand inspection (R) is the place for *adjustments* (slide, flip, zero,
fire-mode) — category swapping (grip A → grip B) belongs to inventory/arsenal even in the big
modular frameworks.

## Variable barrel/handguard length

Two knock-on problems to plan for:

1. **Muzzle position moves** — muzzle flash/suppressor mount must follow the equipped
   barrel/handguard. Solve with a muzzle-offset script component on the barrel part that the
   weapon queries.
2. **Support-hand IK** — hand pose per handguard length; per-part IK targets (a `snap_hand_left`
   on each handguard) are the clean answer.

## Testing checklist

- Every slot: default part spawns, swap works in arsenal, part persists after weapon drop/pickup.
- Inspection (R): slots visible as points; adjust actions work.
- Animation: fire-mode switch / reload with each part combination (a skinned-by-accident rigid
  part explodes here — see [FBX import rules](../10-assets/fbx-import-rules.md)).
- Multiplayer: parts replicate (inherit vanilla bases and Rpl comes free).

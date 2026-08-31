# Weapon Attachments

Optics, collimators, suppressors — and how the slot/type matching actually works.
Official basis: BIKI *Weapon Optic/Collimator/Suppressor Creation*.

## Ground rules (all attachment kinds)

- One FBX per attachment, orientation Y+, own LODs, own colliders.
- Colliders need **two roles**: Weapon (physical interaction) **and FireGeo** — FireGeo also
  drives inventory action raycasts. ⚠️ **If actions are missing on the item in game, check the
  layer preset first.** Simple path: one `UCX`/`UBX` with a `WeaponFire` preset.
- Follow the official directory structure — automation plugins parse it.
- Attachments are **rigid**: import with *Export Scene Hierarchy* (memory points), **never**
  *Export Skinning* (a skinned rigid part explodes when the weapon animates).

## Attachment type matching

Every attachment carries a `WeaponAttachmentAttributes.AttachmentType` (a marker class derived
from `BaseAttachmentType`); every slot filters by such a type. Matching rule (native
`CanSetAttachment`): **the item's type must equal or derive from the slot's type.**

Practical consequences:

- The vanilla optics hierarchy is a tree (dovetail family, RIS1913 family with
  Medium/Short/VeryShort…). An item typed `RIS1913VeryShort` fits **all** RIS slots; a "universal
  optic" across *different* families is impossible — build one variant prefab per family
  (inherit your base optic, override only the AttachmentType + name).
- Custom types are trivial: `class MyMod_AttachmentForegrip : BaseAttachmentType {}` — an empty
  marker class ([Modular weapons](modular-weapons.md)).

## Anatomy of a working attachment prefab

Learned by comparing working vanilla bases; a standalone hand-built attachment misses invisible
essentials. Minimum set:

- `InventoryItemComponent` with `WeaponAttachmentAttributes` (type!) + `PreviewRenderAttributes`
- `MeshObject`
- `RigidBody` with **Model Geometry enabled** (without it: no collision, no clean pickup,
  "RigidBody without geometry" errors)
- `ActionsManagerComponent` with `SCR_EquipWeaponAttachment` (+ `SCR_PickUpItemAction`)
- `RplComponent` + `Hierarchy`

**Easiest path: inherit a close vanilla attachment base** — it brings Rpl/Hierarchy/actions —
and override mesh, type, and name.
⚠️ Slot-spawned prefabs need the component class **`Hierarchy`** — the log warning suggests
"HierarchyComponent", but that class name doesn't exist; the warning text is misleading.

## Optics (scopes)

- The scope's rear is very close to the camera at ADS — spend polygons there (eyepiece cylinders
  ≥32 sides), reduce hard in LOD1.
- Memory points: the optic's own points + a `snap_weapon` point that docks onto the weapon's
  `slot_optics`.
- **PIP scopes:** the scope-HDR material must **inherit the vanilla optic HDR base material**
  (linear tonemapping + reticle map/color/scale parameters). Assigning a plain reticle material
  as the scope HDR yields a black/empty PIP image. The `pip` mesh's own material can stay an
  empty `MatPBRBasic`.
- Zeroing on the optic: see [Ballistics & zeroing](ballistics-and-zeroing.md) (traps #2 and #3).

## Collimators (red dots)

- Grayscale reticle texture, square, aimpoint centered; the projection plane's UV extends to the
  texture edge; the plane must not poke out of the housing (shape open designs accordingly).
- The projection surface's top/bottom edges are defined by two empties:
  `collimator_BR` (bottom right) + `collimator_TL` (top left) — import with scene hierarchy so
  they survive.

## Suppressors

- Modeled like the optic tutorial (`Suppressor_LOD0` naming), mounts to `slot_barrel_muzzle`.
- Weapon obstruction: `SCR_WeaponAttachmentSuppressorAttributes` (extra obstruction length) in
  the InventoryItemComponent's custom attributes. Bayonets: analogous attribute class.

## In-hand inspection (holding R)

The R-inspection shows and operates **fixed categories** (muzzle/mag/fire mode/safety/optic/
bayonet via dedicated user actions). Custom slots appear as points once `ShowInInspection` is set,
but category *swapping* of arbitrary custom slots is not part of the vanilla flow — full swap
stays in arsenal/inventory; in-hand inspection is for adjustments
(see [Modular weapons](modular-weapons.md) for what *is* achievable).

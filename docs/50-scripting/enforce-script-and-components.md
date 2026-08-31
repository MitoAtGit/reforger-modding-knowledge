# EnforceScript & Components

The core patterns for gameplay scripting. All verified against real, compiling mods.
Language traps live in [EnforceScript gotchas](../90-reference/enforce-gotchas.md).

## The two-class component pattern

Every script component is a **pair**:

```c
[EntityEditorProps(category: "MyMod", description: "What it does")]
class MyMod_ThingComponentClass : ScriptComponentClass
{
}

class MyMod_ThingComponent : ScriptComponent
{
    [Attribute(defvalue: "1.5", desc: "Tunable shown in the editor")]
    protected float m_fSpeed;

    override void OnPostInit(IEntity owner)
    {
        super.OnPostInit(owner);
        SetEventMask(owner, EntityEvent.INIT | EntityEvent.FRAME);
        owner.SetFlags(EntityFlags.ACTIVE, true);   // without this: no FRAME events
    }

    override void EOnFrame(IEntity owner, float timeSlice)
    {
    }
}
```

- The `...Class` class is the attribute container (what the prefab serializes); the other is the
  runtime instance.
- `[Attribute(...)]` fields appear in the editor and serialize into prefabs.
- Access the world through `GetOwner()`, siblings via
  `FindComponent(TypeName)`, children via the hierarchy.

⚠️ **Frame events do not fire on carried/inventory items.** A component on a helmet or weapon in
someone's inventory gets no `EOnFrame`. Use the call queue instead:
`GetGame().GetCallqueue().CallLater(Tick, 33, true)` (≈30 Hz) — and clean up in `OnDelete`.

## Modding vanilla classes

`modded class` is the sanctioned override mechanism:

```c
modded class SCR_ChimeraCharacter
{
    override void EOnInit(IEntity owner)
    {
        super.EOnInit(owner);   // ALWAYS call super
        // your hook
    }
}
```

- Great for attaching behavior to every instance of a vanilla entity without touching prefabs.
- Only classes with scripted bases can be modded; native-backed components often have no
  overridable lifecycle (e.g. the character *controller* has no `OnPostInit` — hook the character
  *entity* instead).
- Two loaded mods modding the same method need `super` discipline to coexist.

## Configs as script objects

`.conf` files are serialized instances of `[BaseContainerProps]` script classes. Define a config
class → create the `.conf` in the Resource Browser → load it via `BaseContainerTools`. This is
the pattern behind catalogs, ballistic tables, gamemode settings — prefer it over hardcoding.

## Replication in one page

- Reforger is **server-authoritative**. The server owns game state; clients own their input and
  their player controller.
- `RplComponent` on the prefab makes an entity replicable. `[RplProp]` members sync
  automatically; `[RplRpc]` methods are remote calls.
- Put client→server RPC endpoints on a **`modded class SCR_PlayerController`** — the player
  controller is replicated and client-owned, exactly the ownership RPCs need.
- Guard authority: `if (!Replication.IsServer()) return;` for server logic;
  `RplComponent.IsMaster()` per entity.
- Spawning: `GetGame().SpawnEntityPrefab(...)` on the **server** auto-replicates to clients.
  A client-side spawn exists only locally (invisible to others, can't be shot).
- ⚠️ Runtime-spawned destructibles: the designtime destruction component logs
  *"spawned at runtime… won't take damage"* — runtime spawns need the multi-phase destruction
  component (+Rpl), and shipped config templates may come **disabled** (`Enabled 1` explicitly).
- ⚠️ RPC results arrive **asynchronously** — an entity id can arrive before the entity streams
  in; resolve with retries over a few frames, not once.
- ⚠️ A `RigidBody` set *Static* freezes script-driven movement (static bodies join the static
  collision world at spawn). Kinematic movers: `Static 0, Kinematic 1, Gravity 0`, and call
  `Update()` after `SetTransform`.

## Physics & queries (safe idioms)

- Area queries: `world.QueryEntitiesBySphere(pos, r, addCallback, filterCallback, flags)` —
  callbacks are plain `bool(IEntity)` methods; **return true = keep enumerating**. There is no
  exclude list — filter inside the callback. Flags matter: `STATIC|DYNAMIC` skips foliage
  features that `ALL` would include.
- Raycasts: `TraceParam` + `world.TraceMove(trace, cb)` → fraction <1 = hit; traces (unlike
  queries) support an exclude entity.
- Terrain: surface Y lookups and terrain-basis helpers (slope-aligned placement) are available
  through the script API — use them for placement validation (slope/water/overlap checks) before
  spawning structures.

## Teardown discipline (crash class #1)

Repeated real-world crash pattern (`NULL pointer to instance`): a call chain triggers a cleanup
(closes a UI, deletes an entity), then the *caller* continues using now-null members.

**Rule: after any call that can tear down state you're using, re-check the references** before
touching them. Corollary: never schedule a delayed create (`CallLater(Create…)`) that a cleanup
path is supposed to cancel — the pair cancels itself into never-creating. Clean up call-queue
entries in `OnDelete`.

## Compile-check workflow

A clean recompile is your unit test: every vanilla symbol you use is verified at compile time.
Edit → recompile (focus / Reload Scripts, **edit mode only**) → newest log:
`SCRIPT (E)` = fix the first error from your files; `Module: Game … CRC32` = success. Editing a
file in several rapid saves sprays transient errors between saves — only the final validation
counts.

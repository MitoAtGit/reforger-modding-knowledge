# AI Scripting

How Reforger's AI is architected and the **safe extension points** for influencing it from a mod —
based on the public script API, official docs, and a long-running "observe and nudge vanilla AI"
project that shipped working behavior.

## Architecture in one page

- AI is a **two-entity pair**: a *brain* agent entity (`SCR_ChimeraAIAgent`) bound to a *body*
  (character/vehicle) via the body's `AIControlComponent`.
- The brain runs a **behavior tree** (`.bt`, authored in the Workbench **Behavior Editor** — the
  compiled trees are not hand-editable) plus utility-scored activities, fed by a message/goal bus
  (`SCR_AIMessage…`). `SCR_AIConfigComponent` routes danger events → reactions and goals → trees.
- **Groups are agents too** (a group brain coordinates members; the leader matters).
- A world needs a placed **`SCR_AIWorld`** — it hosts the navmesh and the **AI population
  budget**, the main server-performance lever.
- Skill (`EAISkill`, from NOOB to CYLON) scales aim/perception per unit — there is no single
  difficulty slider.
- Perception: per-body target lists with visibility/aging; groups aggregate members' contacts
  into shared group perception on a ~2 s tick.

## Reading AI state (safe, read-only)

A modded character class can observe every AI in the mission with zero risk:

```c
modded class SCR_ChimeraCharacter { override void EOnInit(IEntity owner) { super.EOnInit(owner); /* start observer */ } }
```

Useful reads (all public API): current target + visibility (combat component), current behavior
(utility component), threat/suppression level (threat system), life state (controller), group +
leader (`AIAgent.GetParentGroup()` → `AIGroup.GetLeaderAgent()`), weapon type
(`GetCurrentWeaponType()` / `HasWeaponOfType(...)`), skill (`GetAISkill()`).

Gotchas discovered doing this at scale:

- Gate on authority (server) and let player characters fall out naturally (they never resolve an
  AI agent).
- **Leadership resolves late** — wait a few seconds after spawn before classifying roles.
- `GetCurrentWeaponType()` reflects the *held* weapon — check `HasWeaponOfType` for launchers/GLs
  carried as secondaries; underbarrel GLs register as muzzles, not weapons.
- Throttle observation (~1 s) — per-frame reads across dozens of agents is wasted budget.

## Influencing AI (ranked by safety)

Proven-safe levers, mildest first — all *nudge* the utility system rather than overriding it:

1. **Waypoints on groups** — the vanilla command layer; always safe.
2. **Share a contact into group perception** (`SCR_AIGroupPerception.AddOrUpdateTarget`) — mimics
   what members do naturally; self-expires; never forces motion. Only share targets the unit has
   *actually perceived*, with faction/position taken from the target itself.
3. **Orders** (stance / movement type / weapon-raised / return-to-default) via the agent's
   mailbox (`AICommunicationComponent.RequestBroadcast(order, agent)`) — consumed by the vanilla
   soldier trees' order handling; transient by design. Clear orders before issuing new ones and
   send a return-to-default when done.
4. **Goal messages** (attack/defend/move/suppress) with a priority — they *compete* in the
   utility pool like vanilla-generated goals; the AI can still refuse in favor of survival.
5. **Target assignment bias** (`SetAssignedTargets`) — strongest tool; **maintain it**: re-point
   when the target changes, reset the moment it dies/vanishes, or the aim solver logs null-entity
   errors. Only assign targets the body itself can perceive.

Design principles that made this stable: every mutation has a **revert path**; commands are
rate-limited and gated behind observation state; everything stays server-side; features ship with
a dry-run flag first (log the would-be command, mutate nothing) and only then go live.

## Spawning & building via AI (advanced)

- There is **no vanilla "AI builds a base" behavior** — but the campaign building system is fully
  script-drivable server-side: spawn a composition in its unfinished state (suppress auto-spawn
  during creation), bind it to a building provider, then drive the layout's building value until
  it completes — with real supply-cost consumption through the resource consumer API when the
  gamemode has the supply economy enabled (Conflict yes, Game Master no).
- Validate placement before spawning anything: terrain slope (align + reject tilt), above-water
  check, and an overlap query over the footprint. Rebuild navmesh around new structures.

## Performance & limits

- The population budget in `SCR_AIWorld` is the lever — more AI than budget degrades everyone.
- Vanilla wildlife is birds/insect ambience; there is no land-animal AI content.
- Vehicle AI usage is limited — verify against vanilla capabilities before designing missions
  around AI drivers.
- Behavior *trees* are Workbench-authored; deep custom behaviors mean the Behavior Editor +
  custom node classes, a bigger commitment than the nudge-layer above.

## Testing recipe

Game Master on a small world → place normal vanilla groups → watch your log tags in the newest
`console.log`. GM-placed units are uniform (REGULAR skill, no ranks) — differentiation comes from
your own logic. For supply-economy features, test in a **Conflict** scenario (GM has no economy).

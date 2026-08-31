# EnforceScript Gotchas

Language-level traps, each one verified by an actual compile error or runtime behavior.
EnforceScript looks like C#/Java — these are the places where it isn't.

## Syntax & operators

- **No ternary operator.** `x = cond ? a : b;` → *"Invalid statement ':'"* / *"Broken expression
  (missing ';'?)"*. Write if/else.
- **No `>>` operator** (*"Unknown operator '>>'"*). `<<` exists and compiles (`alpha << 24` is
  fine) — you can pack, but not unpack, bits by shifting right. Unpack via division/masks or
  store components separately.
- `string.Empty` doesn't exist — use `""`.
- Avoid chained casts/calls like `((int)x).ToString()` — use an intermediate variable.
- No inline array literals as arguments: `Fn({ent})` is invalid — build the array
  (`array<IEntity> a = {}; a.Insert(ent);`).
- `array<float>` parameters: construct with `new` + `Insert`, not from variable literals.
- *"Multiple declaration of variable"* — one declaration per name per method scope, even in
  disjoint blocks.

## Formatting & strings

- `PrintFormat` / `string.Format` support at most **`%1`…`%9`**. A `%10` is a compile error —
  and one compile error takes the whole Game module down with a cascade. Split long log lines.

## Types & enums

- Enums are strongly typed — no direct arithmetic on an enum value (`skill/100.0` fails). Map via
  a `switch` helper.
- A misspelled enum member is a **compile error** — which makes a clean compile a cheap
  verification that every vanilla symbol you referenced actually exists.

## Inheritance & overrides

- `override` requires the method to exist on the base — *"marked as override, but there is no
  function with this name"* means the base class doesn't have it (check the actual API; e.g.
  player controllers expose `OnDestroyed`, not `OnDelete`).
- Some vanilla classes are `sealed` with private constructors — no `new`, no member initializers
  of that type; treat them as opaque handles and null-check via helpers.
- In `modded class`, always call `super.<method>()` unless you deliberately replace behavior —
  other mods chain through the same override.

## Runtime behavior

- **Frame events don't fire on carried/inventory items** → call queue
  ([components](../50-scripting/enforce-script-and-components.md)).
- `GetGame().GetCallqueue().CallLater(fn, ms, repeat)` — remove in `OnDelete`; a repeat timer on
  a deleted owner is a crash waiting.
- Query callbacks (`QueryEntitiesBySphere` etc.): bare `bool(IEntity)` methods; return **true to
  continue** enumerating. Queries have no exclude parameter (filter inside); traces do.
- Deprecated API pairs exist (e.g. replication id lookup functions) — when the compiler warns,
  switch; when only one direction is deprecated, the other is still fine.
- After `SetTransform` on a moved entity call `Update()` — otherwise world bounds/queries see the
  stale transform.

## Tooling interactions

- One real error → cascade. Fix the **first** `SCRIPT (E)` from your files.
- Sequential rapid file saves spray transient errors between saves (the watcher compiles each
  intermediate state) — only the final validation counts.
- WorkbenchGame (plugin) module compiles **only at Workbench start** — game-module hot reload
  doesn't re-register plugin classes.

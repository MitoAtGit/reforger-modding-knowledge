# Vehicle Limits & Frontiers

An honest map of what the engine supports for vehicles **today**, based on what the official
tools expose and what the community has shipped. Versions move — re-verify against current
release notes before betting a project on this page.

## Wheeled — fully supported ✅

The complete, documented path. See [Vehicle pipeline](vehicle-pipeline.md).

## Helicopters — supported ✅

Vanilla ships flyable helicopters; the helicopter simulation and controller components are
scriptable/configurable, and inheriting a vanilla helicopter base is the practical route (its
animation graph node names and vehicle-part conventions come with it). Expect the animation graph
(rotor nodes etc.) to be the fiddly part — authoring graph node setups is Animation Editor work.

## Tracked (tanks) — engine scaffold, no vanilla content ⚠️

- A `TrackedVehicle_Base` prefab and a tracked simulation exist in the base game data, but **no
  finished vanilla tracked vehicle ships**, and as of the 1.7–1.8 era, community attempts to
  drive the native tracked simulation hit engine-side instability. Treat native tracked physics
  as **not yet moddable in practice** until Bohemia ships or fixes it.
- **The workaround the community actually ships:** build the tank on the **wheeled simulation** —
  road wheels as wheel positions (typically 6–8 per side on the `v_wheel_l/rNN` bones), locked/LSD
  differentials tuned toward skid-steer feel, tracks as static (or shader-scrolled) visual mesh.
  It drives, it replicates, it's publishable. It is not true skid-steer physics.
- Searching for the base prefab: the resource is named `TrackedVehicle_Base` (searching
  "Tracked_Base" finds third-party frameworks instead — not core).

## Naval (boats, submarines) — no simulation, but a real buoyancy layer ⚠️

- There is **no naval simulation class** — official statements confirm boats are not a supported
  feature. But the engine has a sim-independent **buoyancy layer** used by vanilla amphibious
  vehicles (BTR-70 / BRDM-2 / LAV-25): buoyancy value + thrust points + forward/reverse/steering
  thrust parameters, all visible in those vanilla prefabs.
- **Boat recipe that works:** derive from the wheeled base, hide the wheels, configure
  boat-like buoyancy/thrust values following the amphibious vanilla prefabs. MP sync comes free.
  Community boat mods with huge download counts follow this pattern.
- Submarines: watertight compartments exist (occupants don't drown), but controllable diving is
  not modeled — community subs use ballast-style tricks. Characters can only swim on the surface;
  AI treats water as impassable (navmesh) — plan for player-only boats.
- Large **walkable moving ships** remain unsolved in Reforger (no vehicle-linking of standing
  characters to a moving deck).

## Fixed-wing aircraft — experimental stub 🚫

A fixed-wing simulation class exists but is explicitly experimental: no vanilla content, no
config-driven aerodynamics — a flight model would have to live almost entirely in script. The
most green-field vehicle domain; expect engine support to evolve.

## Practical advice

- Before starting a vehicle project outside "wheeled/helicopter", spawn-test the base prefab you
  plan to inherit **first** — five minutes of testing beats weeks of asset work on a dead end.
- Keep a small regression test world; after every game update, re-test blocked areas (engine
  fixes arrive without fanfare).
- AI can only use vehicles vanilla AI can use — check before promising AI drivers/pilots.

# Vehicle Pipeline (Wheeled)

From Blender to a drivable car/truck. Official basis: BIKI *Car Creation*
(Asset Preparation / Prefab Configuration / Simulation Configuration) + real builds.

## Stage 1 — Blender asset

- Nose along **Y+**. Real-world scale (verify overall dimensions!).
- **Split into own FBX files:** accessories/variants (roof, cargo bed), destructible parts
  (windows!), emissive lights. Unlike weapons, vehicle parts usually share the parent's origin
  instead of using snap points. Simple parts need no skeleton; complex ones (MG mount) do.
- Up to 8 LODs. Part naming follows the official *Vehicle Pipeline — Parts* conventions — the
  vanilla base prefabs expect those names (wheels, doors, steering wheel…).
- `COM_` empty for center of mass.
- Rig conventions (verified against vanilla-style rigs): root bone `v_root` at origin; wheels
  `v_wheel_l01…/r01…`; doors as bones with handle bones as children; memory-point empties
  bone-parented to `v_root` (`driver_getIn`, `driver_idle`, `codriver_*`, `passengerL/R_*`…).
  Bone tail direction does **not** encode the spin axis — the axis comes from conventions, keep
  tails uniform.
- **Collision:** very simple, few large shapes (convex/cylinder/box). **FireGeo:** may be
  detailed, trimesh, per-part (`UCX_FG_Engine`, `UCX_FG_Fueltank`, …) with the specific vanilla
  vehicle-part gamemats (engine, fuel_tank, differential…) for correct damage behavior.
- Full skinning rules as with weapons; FBX export: Empty+Armature+Mesh, Custom Properties ON,
  Leaf Bones OFF. Import with **Export Skinning** (otherwise no sockets/skinning at all).

## Stage 2 — Prefab

Inherit `Prefabs/Vehicles/Core/Wheeled_Car_Base.et` — or duplicate/inherit a similar vanilla
vehicle (UAZ-class for light 4x4s), which brings SlotManager, sounds, damage, compartments.

- **SlotManagerComponent:** windows, add-ons, MG mounts. ⚠️ Slots **merge by name** through the
  inheritance chain — inheriting a full vehicle keeps its rotor/glass/decal slots spawning; empty
  them by overriding same-named slots with `Prefab ""`, and hang your parts on the inherited slot
  names ([scaffolding](../00-getting-started/mod-project-scaffolding.md)).
- **ActionsManagerComponent:** contexts per door (`door_l01`, get-in at door center), interior
  variants (`door_l01_int`, get-out), seat-switch contexts (`driver_idle`, `passenger_*`),
  `fuel_cap`, `starter_switch`, light switches. RMB → *Create user action context(s) from bones*
  generates contexts — ⚠️ it writes to the **world entity instance**, not the prefab; apply to
  prefab afterwards. Additional actions: engine start (`SCR_EngineAction` on starter_switch),
  refueling (`SCR_FuelReceiverUserAction` on fuel_cap).
- **Compartments (BaseCompartmentManagerComponent):** per seat — compartment action (get in,
  door context), get-out action (interior context), switch-seat action, position info (pivot),
  door info list (links contexts to doors; teleport checkboxes when no animation), entry/exit
  position (e.g. `driver_getin` socket). Add seats by dropping the vanilla compartment configs
  (`CargoCompartment_Base.conf` etc.) onto compartment slots. Seat naming: `Passenger_l/r/m_01/02`
  (side + row).

## Stage 3 — Simulation (VehicleWheeledSimulation)

Real-world data works nearly 1:1:

- **Engine:** Max Power [kW] @ rpm, Max Torque [N·m] @ rpm, Inertia (rpm spool speed), Steepness,
  Idle/Redline/Max rpm. Realistic example (light 4x4): 55 kW / 172 N·m / idle 800 / redline 6000.
- **Clutch:** Max Clutch Torque ≈ 1.6 × engine max torque as a starting value.
- **Gearbox:** forward ratio array (e.g. `3.455 1.944 1.370 1.032 0.850`), reverse (`3.167`).
  Final drive lives in the axle differential.
- **Axles/Suspension:** per axle a differential type — Open (daily driving; dumps torque to the
  slipping wheel), Locked (both wheels same speed), LSD (compromise, strength parameter) — plus
  ratio; Output0/1 = left/right wheel. Multiple driven axles need an extra differential under
  Simulation → Differentials. ⚠️ **Torque Share per axle must be 1/N — at 0 the axle simply isn't
  driven.** Suspension: spring rate, compression/relax dampers, travel up/down.
- **Wheels:** radius [m], mass [kg], brake torque [N·m]. **Tyres:** longitudinal (acceleration) /
  lateral (cornering) friction; Pacejka curves per the official *Wheeled Simulation* page.
- **Aerodynamics:** reference area + drag coefficient (public car cd tables are fine).
- Simulation configs compose from reusable `.conf` files (engines/transmissions/suspensions/
  wheels) — build a library instead of repeating numbers.

## Stage 4 — Debugging

Diag menu (`Win+Alt` / `Win+Ctrl`) → Vehicles: show center of mass, forces, engine, suspension,
wheels, bones, raycasts; Compartments: positions/entry points; **Reset vehicle**. This overlay
answers 90 % of "why does it drive weird".

Build order that avoids frustration: **colliders → FireGeo → sim wheels → engine/gearbox/mass →
compartments.** A visual mesh import is not a vehicle yet.

## Turrets / mounted weapons

Turrets are their own prefabs (Turret base classes) mounted via slots; crew uses compartment
slots of turret type. Start from a vanilla armed vehicle and study its slot/compartment wiring
in the prefab editor — that's the official reference implementation.

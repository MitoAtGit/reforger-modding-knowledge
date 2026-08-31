# Weapon Animation (Blender → Enfusion)

The official workflow for authoring weapon/character animations, plus hard-won Blender practice.
Sources: BIKI *Weapon Animation* (Setup/Basic/Advanced), official Boot Camp videos, and the
official example files.

## Official assets you need

All free and official:

- **Enfusion Blender Tools (EBT)** — exports actions as `.txa` (engine compiles to `.anm`).
  Requires Workbench running with **Net API on**. Blender 4.5/5.1 only — **not 5.2+**
  ([Blender pipeline](../10-assets/blender-pipeline.md)).
- **Character_AnimationRig_Example.blend** — the Rigify control rig on the Reforger skeleton
  ([Arma-Reforger-Samples](https://github.com/BohemiaInteractive/Arma-Reforger-Samples)).
- **Arma-Reforger-Misc** — a goldmine: official weapon animation source .blends
  (M16/AK74/M60/SVD/M9/RPG7…), the BlenderPoses.blend pose library (Rifle/Pistol/LMG/RPG/LAW
  archetypes), vehicle animation sources, collider shape examples, ballistic spreadsheets.
- Boot Camp videos #8 (Blender workflow), #9 (Animation in Workbench), #10 (Scripting).

## Core rules

- **Animate only the Rigify control rig** ("rig"), never modify the Reforger deform skeleton.
  No "Force Connect Children", no "Auto Bone Orientation" on import.
- Weapon attaches via constraint to the **RightHandProp** prop bone; magazine to `LeftHandProp`.
- Every animation phase exists as a **pair**: `p_*` (character action) + `w_*` (weapon action) of
  identical length — the engine plays them in sync (verified across all official sources).
- Weapon mechanics conventions: `w_trigger` ~25° pull; `w_bolt` 3 states (closed/open/
  closed-on-last); `w_fire_mode` one key per mode; safety = a 3-frame pose animation.
- The **IK pose** (`p_<weapon>_ik`) defines the first-person hold; it's exported with dedicated
  profiles and wired via `ItemAnimationAttributes → Animation IK Pose`
  ([prefab configuration](weapon-prefab-configuration.md)).
- Export profiles map actions to engine slots (idle/trigger/bolt/fire-mode/IK…) — use the
  official profile set from EBT's data package; the default-function field stays **empty** in all
  official profiles.

## Reference timings (from the official sources, 30 fps)

Useful as pacing references when authoring your own reloads:

| Weapon family | remove mag | insert mag | extra |
|---|---|---|---|
| Pistol (M9) | 60 | 60 | slide open/close variants ~30–45 |
| AR15 (M16) | 59 | 72 | bolt release ~39–47 |
| AK74 | 59 | 72 | **always racks the bolt (~43)** — no bolt catch |
| MG (M249) | open 131 | insert 149 | sweep 143, close 80, rack 76, cancel anim |
| UGL (M203) | remove 99 | reload 106 | hand-switch anim |
| Grenades | unpin/repin ~35–40 | | |

Vanilla has **no revolver, bolt-action, or pump-action** animation sets — for those you design
your own cycle (cylinder swing-out + eject up + load down; bolt rotate+translate; pump = slide
analog to the M203 pattern with single-round loops).

## Animation craft (what makes reloads look right)

Collected from professional workflows and verified in practice:

- Block with key poses first; animate the mag **backwards from its seated pose**; end insertions
  with an edge *bump* (overshoot + settle), never a smooth glide.
- Weapon dip + spring-back when the mag is pulled; slight desync between hand and weapon; small
  end-of-anim wiggles sell weight.
- Fingers: delay finger closing slightly after grip contact; keep a hold key one frame before any
  finger change; use a trigger-discipline intermediate pose.
- Constraint handovers (hand takes mag, hand racks bolt): key the **ChildOf influence** with
  visual keyframing (frame N−1: old parent keys; frame N: visual loc/rot key, then new influence).
  For "hand follows bolt", constrain the hand to the bolt bone (reverse constraint) and snap back
  with a 1-frame release.
- Fewer keyframes = smoother curves; never fully static holds; keep the end key slightly before
  the motion truly ends (gameplay may cut early).
- **For human poses, start from official animations, not from procedural construction** — posing
  fingers algorithmically produces "crippled hand" results; official pose assets + small
  corrections win.

## Prefab wiring

`WeaponAnimationComponent`: graph `.agr`, weapon `.asi`, *Always Active* + *Bind With Injection*,
injection binding name exactly **"Weapon"**, player `.asi` on the character side. The animation
workspace/graph/set files are authored in the Workbench Animation Editor; instances bind named
sources (e.g. a reload clip name) to your exported `.anm` clips.

Pitfalls: duplicating a graph file requires fixing its internal template reference; transition
durations are floats; "Generate Default Profile" should be switched off immediately.

## Verifying

- Log errors `starting node X doesn't exist` = the assigned graph lacks the node names the
  component expects — when inheriting a vanilla weapon, the cheapest fix is using its inherited
  graph instead of overriding with your own.
- `Animation source invalid, skipping` = the animation set references clips that aren't loaded.
- In-game test needs a **fresh play session** (stale sessions keep old anims).

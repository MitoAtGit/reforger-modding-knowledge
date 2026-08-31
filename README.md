# Arma Reforger / Enfusion Modding Knowledge

A curated, field-tested knowledge base for modding **Arma Reforger** on the **Enfusion** engine —
distilled from months of hands-on modding: weapons, vehicles, terrain, scripting, publishing, and
all the gotchas in between.

Everything here was learned by building real mods (weapons, optics, night-vision gear, terrains,
AI experiments) and verified against the official tools. Where something is a hard-won lesson
("this crashes", "this silently fails"), it is marked as such.

> **Unofficial project.** Not affiliated with or endorsed by Bohemia Interactive.
> *Arma Reforger* and *Enfusion* are products of [Bohemia Interactive a.s.](https://www.bohemia.net/)
> The primary source of truth is always the official
> [Bohemia Interactive Community Wiki (BIKI)](https://community.bistudio.com/wiki/Category:Arma_Reforger)
> — this repo complements it with practice, ordering, and pitfalls, it does not replace it.
>
> See [COMPLIANCE.md](COMPLIANCE.md) for what this repo deliberately does **not** contain.

---

## Contents

### 00 — Getting started
| Doc | What it covers |
|---|---|
| [Workbench setup](docs/00-getting-started/workbench-setup.md) | Installing the tools, project anatomy, where logs live |
| [Mod project scaffolding](docs/00-getting-started/mod-project-scaffolding.md) | `addon.gproj`, GUIDs, `.meta` files, directory structure, naming rules |
| [World Editor basics](docs/00-getting-started/world-editor-basics.md) | Camera keys, workflow, layers, saving |

### 10 — Assets (Blender → Enfusion)
| Doc | What it covers |
|---|---|
| [FBX import rules](docs/10-assets/fbx-import-rules.md) | Orientation, LODs, collider prefixes, `usage` property, export settings |
| [Blender pipeline & tools](docs/10-assets/blender-pipeline.md) | Recommended addons, version caveats, headless workflows |
| [Collision & LODs](docs/10-assets/collision-and-lods.md) | Collider types, limits, FireGeo, common import failures |
| [Materials & textures](docs/10-assets/materials-and-textures.md) | `.emat`, channel packing (BCR/NMO/NHO), game materials |
| [Hard-surface modeling](docs/10-assets/hardsurface-modeling.md) | Reference-driven weapon modeling workflow that actually works |

### 20 — Weapons
| Doc | What it covers |
|---|---|
| [Weapon pipeline overview](docs/20-weapons/weapon-pipeline.md) | The full path: Blender asset → prefab → in game |
| [Weapon prefab configuration](docs/20-weapons/weapon-prefab-configuration.md) | Rifle_Base, components, fire modes, recoil, magazine wells |
| [Ballistics & zeroing](docs/20-weapons/ballistics-and-zeroing.md) | Sights ranges, air drag calibration, the zeroing modifier trap |
| [Attachments](docs/20-weapons/attachments.md) | Optics, collimators, suppressors, attachment type matching |
| [Custom sounds](docs/20-weapons/custom-sounds.md) | `.acp` projects, sample swapping, audio pitfalls |
| [Weapon animation](docs/20-weapons/weapon-animation.md) | Official Blender workflow, Enfusion Blender Tools, TXA export |
| [Modular weapons](docs/20-weapons/modular-weapons.md) | Attachment slots, nested slots, custom attachment types |

### 30 — Vehicles
| Doc | What it covers |
|---|---|
| [Vehicle pipeline](docs/30-vehicles/vehicle-pipeline.md) | Asset prep, compartments, actions, wheeled simulation |
| [Limits & frontiers](docs/30-vehicles/vehicle-limits-and-frontiers.md) | What the engine supports today (tracked, naval, aircraft) |

### 40 — Terrain & maps
| Doc | What it covers |
|---|---|
| [Terrain pipeline](docs/40-terrain/terrain-pipeline.md) | The dependency-ordered build: land → water → life → people → roads |
| [Heightmaps & real-world data](docs/40-terrain/heightmap-and-real-world-data.md) | DEM/QGIS/GDAL workflows, import parameters |
| [Surfaces & satmap](docs/40-terrain/surfaces-and-satmap.md) | Surface masks, the 5-per-block limit, satmap authoring |
| [Roads, water, vegetation](docs/40-terrain/roads-water-vegetation.md) | Spline generators, rivers, forest generators |
| [Map finishing](docs/40-terrain/map-finishing.md) | 2D map, shore map, navmesh, believability checklist |

### 50 — Scripting (EnforceScript)
| Doc | What it covers |
|---|---|
| [EnforceScript & components](docs/50-scripting/enforce-script-and-components.md) | The two-class component pattern, events, lifecycle |
| [Config overrides](docs/50-scripting/config-overrides.md) | Overriding vanilla configs by GUID, input actions, the `+` merge rule |
| [UI & HUD](docs/50-scripting/ui-and-hud.md) | Layouts, InfoDisplays, canvas drawing, the two coordinate systems |
| [AI scripting](docs/50-scripting/ai-scripting.md) | Agent architecture, perception, messages/orders, safe extension points |
| [Workbench plugins & generators](docs/50-scripting/workbench-plugins-and-generators.md) | Editor tooling, shape-driven generators |

### 60 — Publishing
| Doc | What it covers |
|---|---|
| [Arsenal & catalogs](docs/60-publishing/arsenal-and-catalogs.md) | Getting items into faction arsenals, ranks, supply costs |
| [Workshop publishing](docs/60-publishing/workshop-publishing.md) | Pre-publish checklist, validation false alarms, platform configs |

### 90 — Reference
| Doc | What it covers |
|---|---|
| [Troubleshooting](docs/90-reference/troubleshooting.md) | Symptom → cause → fix, collected from real debugging sessions |
| [EnforceScript gotchas](docs/90-reference/enforce-gotchas.md) | Verified language traps (no ternary, no `>>`, 9-arg format cap…) |
| [Glossary](docs/90-reference/glossary.md) | The vocabulary: prefab, `.et`, `.emat`, memory point, gamemat… |
| [Official resources](docs/90-reference/official-resources.md) | Links to BIKI, samples, Boot Camp videos, community tools |

---

## How to read this repo

- **New to Reforger modding?** Start at [Workbench setup](docs/00-getting-started/workbench-setup.md),
  then follow the pipeline for the thing you want to build (weapon / vehicle / terrain).
- **Stuck on an error?** Go straight to [Troubleshooting](docs/90-reference/troubleshooting.md) —
  it is organized by symptom.
- **Placeholder GUIDs:** examples use `{VANILLA-GUID}` placeholders instead of real vanilla resource
  GUIDs. Resolve any of them yourself in seconds: right-click the resource in Workbench's Resource
  Browser → **Get Resource GUID(s)**. See [COMPLIANCE.md](COMPLIANCE.md) for why.

## Contributing

Corrections and additions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).
The bar: only official sources and your own verified, hands-on findings. No content extracted from
game files, no third-party mod code.

## License

Documentation and examples are released under the [MIT License](LICENSE).
This license covers **only the original content of this repository** — not Arma Reforger, the
Enfusion engine, or any Bohemia Interactive content, which remain governed by their own EULAs and
the [Bohemia Interactive Game Content Usage Rules](https://www.bohemia.net/community/licenses).

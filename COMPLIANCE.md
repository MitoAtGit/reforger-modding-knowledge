# Compliance & Content Policy

This repository is written to comply with the Bohemia Interactive EULAs for Arma Reforger and the
Arma Reforger Tools, and with the
[Bohemia Interactive Game Content Usage Rules](https://www.bohemia.net/community/licenses).
That is a deliberate, enforced editorial line — not an afterthought.

## What this repo contains

- **Own explanations and workflows**, written from hands-on modding experience.
- **Paraphrased knowledge from official public sources**, always with the source named:
  the [Bohemia Community Wiki (BIKI)](https://community.bistudio.com/wiki/Category:Arma_Reforger),
  the public Enforce Script API reference, official Bohemia sample repositories
  ([Arma-Reforger-Samples](https://github.com/BohemiaInteractive/Arma-Reforger-Samples),
  [Arma-Reforger-Misc](https://github.com/BohemiaInteractive/Arma-Reforger-Misc)), and official
  Boot Camp videos.
- **Observations made through the official tools** — what the Workbench shows any modder in its
  own UI (component fields, import warnings, log messages, editor behavior).
- **Original example code** written for this repo.

## What this repo deliberately does NOT contain

1. **No extracted game files or data derived from extraction.** No file dumps, no bulk lists of
   paths/GUIDs/class inventories harvested from game data, no repackaged vanilla content.
2. **No file-format reverse engineering.** You will not find binary format specs for proprietary
   Enfusion formats here, nor tools or instructions to unpack game archives. The supported way to
   read game content is the official Enfusion Workbench.
3. **No third-party mod source code.** Other authors' mods are at most *named and linked*, never
   quoted or reproduced. Respect for other modders' IP is part of the editorial line.
4. **No verbatim vanilla script code.** Vanilla classes and members are *referenced by name* (as
   the public API docs do) and their behavior is described in our own words.
5. **No leaked, internal, or unreleased material.** Nothing about unreleased content, internal
   codenames, or experimental-branch internals.

## The GUID placeholder rule

Working prefab/config examples need vanilla resource references. Real vanilla GUIDs in bulk are an
extraction derivative, so examples here use a placeholder:

```
Rifle "{VANILLA-GUID}Prefabs/Weapons/Rifles/M16/Rifle_M16A2_base.et" {
```

To resolve a placeholder: open the path in Workbench's Resource Browser, right-click →
**Get Resource GUID(s)**, paste. Every referenced vanilla path in this repo can be resolved that
way in seconds, from the official tool, on your own legally owned copy.

## Ground rules for contributors

Before adding anything, apply this test (it decides every borderline case):

> *"Could I obtain and cite this from an official public source, or from my own original work,
> without extracting or reverse-engineering game content?"*

If the answer is no — it does not go in. When in doubt, leave it out.

Additional rules:

- Non-commercial spirit: this knowledge base is free and must stay free.
- Never present this project as affiliated with Bohemia Interactive.
- If Bohemia Interactive requests changes or removal of any content, that request wins.

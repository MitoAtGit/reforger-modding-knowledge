# Custom Weapon Sounds

How Reforger audio is wired, and how to give a weapon a custom firing sound without breaking
anything — including the crash traps.

## The pieces

| File | What it is |
|---|---|
| `.wav` | the sample. Target format: **48 kHz, mono, 16-bit PCM** |
| `.acp` | an **Audio Project** — a node graph (signals, banks, mixers, shaders) authored in the Workbench **Audio Editor** |
| `WeaponSoundComponent` | the prefab component listing the `.acp` banks a weapon uses |

Weapon shot `.acp`s are complex, signal-driven graphs (~30 banks: first-person body, mechanics,
tails, distance layers, environment variants, caliber layers). **Do not author one from scratch.**

## The sample-swap technique (works, low risk)

1. In Workbench, **duplicate a vanilla same-caliber shot `.acp` into your project**
   (Resource Browser → right-click → *Duplicate to project*; e.g. a 9 mm pistol shot project for
   a 9 mm SMG).
2. In your copy, replace the sample file references of the **"Body FP" bank** — the first-person
   shot body the shooter hears — with your own imported `.wav`.
3. Leave mechanics/tails/distance layers vanilla → your sound keeps the full spatial treatment
   (occlusion, distance, environment) for free. Third-person distance layers use separate
   Mid/Far/caliber samples — swap those too if others should hear your sound.
4. Override the weapon's `WeaponSoundComponent` file list: your shot `.acp` first, keep the
   generic handling/drop/pickup/melee/deployment banks. Overriding the list **replaces** it, so
   re-list everything you want to keep.

Sample prep tips:

- Full 1-second shot samples overlap badly at high RPM (1200 rpm = a shot every 50 ms) — trim to
  ~0.15–0.25 s (just the crack) for clean full-auto.
- Stereo samples can't be spatialized — **mono only** for world sounds.
- ⚠️ **Measure the sample's peak level after any conversion.** A silent WAV is formally valid and
  looks exactly like a routing bug — hours have been lost to "why is there no sound" when the
  file itself contained silence.

## ⚠️ Hand-editing `.acp` graphs: don't

`.acp` files are text and *look* editable. Simple sample-path swaps in a copied project are fine.
Deleting or rewiring graph **nodes** by hand is not:

- Node IDs/ports are cross-linked. A dangling connection can *load and compile fine* and then
  **crash the game natively** the first time it actually plays (no crash dump, log just ends).
- A single broken `.acp` anywhere in the project also **crashes the Workbench Audio Editor on
  open** (it loads all audio projects), even if nothing references the file.
- Symptom of a broken graph before the crash stage:
  `AUDIO (E): Compile error: Item @"....acp|<id>" not found` → **no sound at all**.

Rule: structural audio work happens in the **Audio Editor**; hand edits are for sample paths only.

## Non-weapon sounds (props, devices, loops)

- A device sound (switch click, drone loop): Bank → Shader + Amplitude → Sound event; trigger via
  `SoundComponent` events from script. Loops that don't self-loop need re-triggering on a timer.
- The entity needs `SignalsManagerComponent` (with an s!) if the `.acp` graph reads signals —
  the log warning `SignalManagerComponent not present` at spawn time is the tell.
- Event names for vanilla sounds are discoverable via the official docs/samples
  (e.g. flashlight on/off style click events for device toggles).

## Diagnosis order for "no sound"

1. Newest `console.log` → any `AUDIO (E)`? → broken graph.
2. `SignalManagerComponent not present` warning at spawn? → add SignalsManagerComponent.
3. Sample peak level > 0? (really.)
4. Mono? 48 kHz?
5. `Print()` at the trigger site — proves the event fires; then it's wiring vs. playback.
6. Audio is only truly verifiable **in game** — Workbench preview doesn't cover the full chain.

## Import formats

Drop the `.wav` in the project — the file watcher imports it. Converting from MP3/etc.: any
ffmpeg build (`-ar 48000 -ac 1 -c:a pcm_s16le`).

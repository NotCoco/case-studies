# Custom Wolf — Case Study (Uncached Runtime)

**Role:** Independent/Freelance Software Engineer (2025)
**Stack:** Blender, Python, Java/Kotlin (RuneLite/OpenOSRS), Gradle
**Demo:** (GIF below)

![Wolf – quick demo](wolves.gif)

## TL;DR

Custom creature (**wolf**) loaded **outside the OSRS cache** using a tiny pipeline: **Blender → GLTF → Python baker → JSON → client pre‑warm**. First‑spawn hitch is eliminated by pre‑warming on login.

## What I built

* **Python “baker”** that reads GLB clips, extracts bone poses/weights, **quantizes** data, and writes per‑clip **JSON** + a `meta.json` (scale/bbox/fps/clip map).
* **Gradle staging** task (`packWolf`) that copies baked files to `Cache/runtime/wolf/` for hot‑load.
* Client‑side **RuntimeAnimLoader** + **NpcComposerHook** to load JSON at runtime and **compose** the animated mesh per NPC state.
* **Pre‑warm on login** to parse/upload once so the first on‑map spawn is smooth.

## Why it matters (impact)

* **No cache repacks** → much faster iteration for meshes/animations.
* Can **hot‑swap** content for dev/whitelisted users without touching indices.
* **Smooth spawn** after pre‑warm; feels native in‑game.

## How it works (short)

* **Export** GLB per clip in Blender (idle/run/attack/death).
* **Bake**: `python tools/gltf_to_runtime.py --in source --out baked --fps 24` → `baked/*.json` + `meta.json`.
* **Stage**: `./gradlew :runelite:cache:packWolf` → `Cache/runtime/wolf/`.
* **Client** on login: `prewarm("wolf", [idle, run, attack, death])` → **spawn wolf (ID 9901)**; state maps to clip.

## Pitfalls I solved

* **Bone order & axis** mismatches → stable bone list, enforced **+Y up** on export & in loader.
* **Weight renorm** after 4‑influence trim/quantize (u8) → renormalized on load.
* **Foot sliding / speed** → store **fps** in `meta.json` and drive sampling by time; consistent root‑motion policy.

## Notes

* Private repo; reference available on request.

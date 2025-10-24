# Wolf — Case Study (Uncached Runtime + Ability Engine)

**Role:** Independent/Freelance Software Engineer (2025)
**Stack:** Blender, Python, Java/Kotlin (Server + RuneLite/OpenOSRS Client), Gradle
**Demo:** (GIF below)

![Wolf – quick demo](wolves.gif)

## TL;DR

A custom creature (**wolf**) and a new **Wolf Rush** special built on a lightweight **Ability Engine**. The client loads wolf assets **outside the OSRS cache** (Blender → GLTF → Python baker → JSON) and **pre‑warms** them on login for smooth first spawn. The server centralises preload/warm‑up, projectile listeners, and NPC movement overrides so abilities stay self‑contained.

## What I built

* **Ability framework** (Ability, AbilityRegistry, AbilityEngine) that centralises **preload/warm‑up hooks**, **projectile listeners**, and **movement overrides**.
* **Wolf Rush ability** on top of that engine: spawns **preloaded virtual wolves**, tracks active spawns, and cleans them without leaking into projectile/movement code.
* **Core integrations** that call the engine (server boot preload, player login warm‑up, projectile dispatch, NPC tile occupancy/pathfinding, admin commands).
* **Safety utilities** (non‑blocking NPC tracking, random‑walk suppression, safe despawn tooling) to avoid deadlocks and lingering entities.

## Why it matters (impact)

* Core gameplay code stays **agnostic** to specific abilities → fewer regressions when adding content.
* New abilities **register once** and automatically get lifecycle, projectile, and movement helpers.
* **Hot‑disable/debug** per ability for fast live testing and support.
* Cleaner separation → easier review/audit; future content slots in alongside Wolf Rush with minimal wiring.

## How it works (short)

* **Server side**

  * Abilities register with **AbilityRegistry**.
  * **AbilityEngine.ensurePreloaded()** runs at server start; **AbilityEngine.warmFor(player)** runs on login.
  * Projectiles call **AbilityEngine.onProjectileSent(attacker, target, gfx)**; the engine dispatches to listening abilities (Wolf Rush reacts to specific GFX IDs).
  * When Wolf Rush spawns wolves, the engine marks them **non‑blocking**; movement, tile occupancy, and routing respect that shared flag.
  * Admins can **despawn safely** via **AbilityEngine.despawnAll(<abilityKey>)** (e.g., `geclear`).

* **Client side (uncached content path)**

  * **Blender → GLTF** clips (idle/run/attack/death) → **Python baker** writes per‑clip **JSON** + `meta.json` (scale/bbox/fps).
  * Gradle task **`packWolf`** stages to `Cache/runtime/wolf/`.
  * On login the client **pre‑warms** (`RuntimeAnimLoader.prewarm(...)`) and composes via **NpcComposerHook**; first spawn is smooth.

## Pitfalls I solved

* Removed **Wolf‑specific hooks** from Projectile/NPCMovement/Tile/RouteFinder; replaced with **engine checks** to avoid copy‑paste bugs.
* Made preload/warm‑up **idempotent & failure‑tolerant** so one buggy ability can’t stall startup.
* Managed concurrent NPC tracking with **weak sets / COW lists** to prevent leaks and races during despawn/logout.
* Preserved legacy behaviour (**smooth spawn, non‑blocking movement**) while making it **configurable per ability** instead of hard‑coded.

## Notes

* Private repo; reference available on request. Asset/code names trimmed for privacy.

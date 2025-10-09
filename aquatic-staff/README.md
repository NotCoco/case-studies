# Aquatic Staff — Case Study

**Role:** Independent/Freelance Software Engineer (Jul–Aug 2025)  
**Stack:** Java/Kotlin, RuneLite plugin UI, Spring/Hibernate, Gradle  
**Demo (0:11):** https://youtu.be/XEU0gbTAwLM

![Aquatic Staff – quick demo](demo.gif)

## TL;DR
Custom client–server **combat item** with a lightweight **RuneLite plugin UI**.  
Server stays **authoritative** (no dup-casts); client delivers a **fast, clear UX**.

## What I built
- **Novel interaction model** (multi-mode actions with cooldown/state).
- **Plugin UI** with hotkeys, overlay, tooltips, and cooldown timer.
- **Smart hover targeting:** overlay lets you select **tiles behind walls/occlusion**, making targeting much easier.
- **Server handlers** that validate intents (mode/range/cooldown) and broadcast state.

## Why it matters (impact)
- Reduced user actions (multi-step → **one hotkey + confirm**).
- Consistent “server truth” UI; stable under rapid toggling.
- *(Optional to add numbers later: median RTT ~N ms; 0 dup-casts in X hours.)*

## How it works (short)
- Client keeps a small **state machine** (Idle → Charged → Cooldown).  
- Client sends an **Intent**; server validates and replies with a **StateUpdate** that the UI mirrors.  
- **Occlusion-agnostic hover:** cursor→tile resolution uses overlay/world-to-screen projection (not LOS), with range checks to prevent illegal targets.

## Notes
- Private repo; reference available on request. Names/assets omitted for privacy.

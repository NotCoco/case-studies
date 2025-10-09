# Aquatic Staff — Design & Implementation (Case Study)

**Role:** Freelance / Independent Software Engineer (2023–2024)  
**Stack:** Java/Kotlin, RuneLite plugin UI, Java server module (Spring/Hibernate), Gradle  
**Links:** **Demo (≈1:12):** 
*Reference from the server owner available on request. Private code not published.*

![Aquatic Staff – quick demo](demo.gif)

---

## Overview
I designed and shipped **Aquatic Staff**, a custom **client–server combat item** for a private MMO server. The feature introduced **novel interaction modes** not present in the base game and a dedicated **RuneLite plugin UI** for control and feedback. The server remains **authoritative** over effects and cooldowns; the client provides a smooth UX with hotkeys, tooltips, and an on-screen status overlay.

---

## Goals & Constraints
- **Discoverable UX:** clear affordances; minimal clicks/keypresses to use special actions.  
- **Low perceived latency:** responsive feel while preserving server authority.  
- **No desync/exploits:** client cannot trigger illegal states or spam packets.  
- **Operate within tick timing:** align with the game’s ~600 ms tick cadence.  
- **Maintainability:** clean state machine, small packet surface, logs/telemetry for QA.

---

## User Experience (RuneLite Plugin)
- **Contextual UI:** compact panel + overlay with action name, mode, and cooldown notification.  
- **Hotkeys:** configurable keybinds for *mode toggle* and *special action*.  
- **Feedback:** disabled states during cooldown; tooltip explains requirements.  
- **Edge handling:** UI re-syncs from server state on login/world hop.

---

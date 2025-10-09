# Aquatic Staff — Design & Implementation (Case Study)

**Role:** Freelance / Independent Software Engineer (2023–2024)  
**Stack:** Java/Kotlin, RuneLite plugin UI, Java server module (Spring/Hibernate), Gradle  
**Links:** **Demo (≈1:12):** https://youtu.be/YOUR_UNLISTED_LINK  
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
- **Contextual UI:** compact panel + overlay with action name, mode, and cooldown timer.  
- **Hotkeys:** configurable keybinds for *mode toggle* and *special action*.  
- **Feedback:** disabled states during cooldown; tooltip explains requirements.  
- **Edge handling:** UI re-syncs from server state on login/world hop.

---

## Architecture
~~~text
Player Input / Hotkey
   |
   v
[RuneLite Plugin]
  - Local state machine
  - UI/overlay + tooltips
  - Sends IntentPacket(seq, mode, target, clientTick)
   |
   v
[Server Handlers]
  - Validate intent (mode, cooldown, range, resources)
  - Apply authoritative effects
  - Emit Ack/StateUpdate(seq, serverTick, cooldownMs, newState)
   |
   v
[Client]
  - Renders effects and cooldown from server update
  - Drops stale acks by seq/tick
~~~

**Why server-authoritative?** Prevents dup-casts and race conditions during fast toggles; guarantees consistency across clients.

---

## Protocol (pseudocode)
~~~java
// Client → Server
class IntentPacket {
  int seq;         // monotonically increasing per client
  Mode mode;       // e.g., CHARGE, RELEASE, ALT
  int targetId;    // optional
  int clientTick;  // last observed tick to detect staleness
}

// Server → Client
class AckOrStateUpdate {
  int seq;          // mirrors last accepted seq
  boolean accepted; // rejected intents return reason code
  int serverTick;   // server-side tick for sync
  int cooldownMs;   // remaining cooldown
  State state;      // e.g., IDLE, CHARGED, COOLDOWN
}
~~~

**Guards**
- Drop packets with **stale `seq`** or **mismatched tick**.  
- **Rate-limit** per player; exponential backoff on repeated rejects.  
- Audit log: `playerId, seq, mode, result, reason, serverTick`.

---

## State Machine (client mirrors; server is source of truth)
~~~text
IDLE
 ├─(hotkey & requirements)→ CHARGED
 ├─(illegal)──────────────→ stay(IDLE)

CHARGED
 ├─(release action)───────→ COOLDOWN
 ├─(cancel/timeout)───────→ IDLE

COOLDOWN
 ├─(timer elapsed)────────→ IDLE
 └─(any input)────────────→ ignored
~~~

- Client pre-validates transitions; **server re-validates** before accepting.  
- Cooldown displayed using **server timestamps** with small drift correction.

---

## Implementation Highlights
- **RuneLite plugin:** overlay panel + hotkeys, debounced inputs, state mirror, reconnection re-sync.  
- **Server module:** intent validator, combat/timing integration, effect application, update broadcaster.  
- **Packets:** compact, sequenced; rejection reasons surfaced to the UI.  
- **Resilience:** relog/reconnect replays no intents; client requests a fresh state snapshot.

---

## Testing & Telemetry
- **Unit tests:** state transitions and cooldown math.  
- **Integration tests:** packet round-trips and reject paths.  
- **QA tooling:** debug overlay with seq/tick; log sampling for anomaly rates.  
- **Operational logs:** accepts/rejects, cooldown violations, average RTT.

---

## Results
- Reduced user actions to trigger the effect (multi-step → **single hotkey + confirm**).  
- Stable behavior under rapid toggling; no dup-casts observed in internal testing.  
- Perceived responsiveness acceptable at EU latency; client always reflects **server truth**.  
- Lower QA churn due to clear reject reasons and UI hints.

> Replace with your numbers if you want:  
> Median RTT ~**[N] ms**; 95p ~**[N] ms**. Zero desync bugs in **[X]** hours of live play; **[Y]** total uses first week.

---

## Future Work
- A/B testing for alternative cooldowns.  
- Localized UI strings + accessibility options.  
- Optional metrics export to a dashboard for live monitoring.

---

## Notes on IP/Privacy
- Server name and assets intentionally omitted.  
- Code walk-through available privately with client permission.

---

## Resume Blurb (copy/paste)
**Freelance Software Engineer — Private MMO Server (client)** | 2023–2024 | Java/Kotlin, RuneLite, Spring  
- Shipped **Aquatic Staff**: custom combat item with a novel interaction model (multi-mode actions, state machine, server-authoritative cooldowns).  
- Built a RuneLite plugin UI (hotkeys, overlays) and designed sequenced packets to prevent desyncs.  
- Result: fewer user steps and consistent server-truth UX. **Technical write-up** · **Demo**

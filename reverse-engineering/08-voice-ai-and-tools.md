# 08 - Voice AI and Tools (behavioural reverse-engineering)

**Purpose.** This document describes, as observable behaviour, how the reference product's hands-free voice assistant works: how a session starts, connects, listens, answers, is interrupted, is metered and is torn down; the complete set of things the assistant can do by voice (grouped by purpose, with inputs, outputs and guards); how analyst-style questions are answered; how the short on-screen "intel" summary line is generated; how cost is shown and throttled; what data leaves the machine; and which tools are also offered to external AI assistants through an open tool protocol (MCP). It is written so a clean-room team can rebuild an original equivalent. No code, file names, identifiers or routes are reproduced; provider and product names and numeric behaviour values are used where they matter.

Confidence tags used below: [OBSERVED-IN-SOURCE] read directly in the reference source; [FROM-TESTS] stated by the reference's automated tests; [FROM-DOCS-ONLY] stated only in its prose docs; [INFERRED]. Untagged sections were checked against source in the 2026 revision pass unless a note says otherwise. Text merged in Pass 3 comes from the independent feature census (13f for voice, tools and MCP; 13e for annotation hints) and is tagged [CENSUS] where it adds a mechanism not previously stated; census claims are source-level, no live run.

Conventions: "the assistant" = the realtime voice agent; "the card" = the small on-screen voice card; "the runner" = the in-browser component that actually executes tool calls; "contacts" = the nearby-aircraft roster mode; "scope" = where a question applies (view, place, radius, pointer, anywhere).

---

## 1. Architecture in one page

| Piece | Responsibility |
|---|---|
| Browser voice control (bottom command dock) | Mic button, status text, level meters, model-tier toggle, running cost readout, help tray, error tray, voice card. |
| Realtime session (browser <-> cloud) | A direct browser-to-cloud real-time audio connection (WebRTC peer + data channel) to OpenAI's Realtime service. Audio never passes through the product's own server. |
| Local server "key broker" | Mints a short-lived client secret for the browser; holds the long-lived credential; applies per-IP throttle; writes a redacted debug log; produces the periodic HUD summary line. |
| Tool runner (browser) | Executes every tool call against the live app state and returns a structured result plus a spoken line plus a card display payload. |
| Tool catalogue (portable) | Read-only "query" tools that answer from public data anywhere in the world; the same definitions are exposed to voice and to external assistants through MCP, filtered per surface. |
| Instructions + tool schemas | Fixed behavioural prompt and strict argument schemas (enumerations, numeric ranges, length caps) sent when the secret is minted. |

Design principles visible in behaviour:
1. The model never executes anything itself; it can only call a fixed list of tools with schema-validated arguments. Unknown tools or arguments are ignored/refused.
2. Results are composed by code (short spoken line + richer card), not left to the model, so numbers and caveats stay exact.
3. The assistant must only claim success for actions whose result says it succeeded.
4. The conversation window is deliberately short; live map state is fetched fresh per question instead of remembered.

---

## 2. Voice session lifecycle

### 2.1 Controls the user sees
- A round mic button labelled ON/OFF with an orbiting ring; kicker "AI AGENT"; a status word; a 15-bar level visualizer; a one-line detail readout ("VOICE STANDBY" / "VOICE ACTIVE" / error text); a help tray; the voice card; an error tray with a DISMISS button.
- Status words: OFF, CONNECTING, LISTENING, EXECUTING, ERROR.
- The voice card shows a live caption (long captions keep only the last 220 characters), up to 6 progress steps, up to 6 detail lines and up to 6 notes (extra notes are merged into the last one), and at most 5 numbered rows (the ones that can be referenced by number). The card fades out about 14 seconds after its last update. [CENSUS] One adapter turns every tool result into the card shape (title, lines, chips, notes, sources); analyst results map their semantic fields (count, completeness, scope label, partial, caveat) onto the same shape, and the same adapter feeds the numbered-reference registry, so the card and "the second one" can never disagree. Mechanism detail: 13f section 1A (voice card).
- A two-state model toggle (STD / MINI) and a cost readout (e.g. "~$0.42") sit in the heading row. The toggle's tooltip names the model and says the change applies next session.
- Help text: "Hold Space to speak - tap Space to activate focused controls"; while a hold is active: "Release Space to send"; in push-to-talk idle: "Hold Space to talk".
- Accessible label on the mic button explains the toggle plus the hold-Space shortcut.

### 2.2 Starting a session - two ways
| Mode | Gesture | Behaviour |
|---|---|---|
| **Open-mic (click-started)** | Click the mic button, grant browser microphone permission | The assistant listens continuously; end of each utterance is detected by the cloud service (semantic voice-activity detection, low eagerness, with noise reduction tuned for a near-field mic). The mic stays hot until Stop. Space never mutes a click-started session. |
| **Push-to-talk** | Press and hold Space for 500 ms | Claims the gesture: starts a session if idle (or, if a push-to-talk session is live, opens the mic and, if the assistant is speaking, interrupts it). The mic is un-muted only while the key is held; on release the mic is muted but the connection stays up so the reply still plays. Status line says "Release Space to send" while held, "Hold Space to talk" after. |

Space arbitration rules (important to reproduce):
- A tap shorter than the 500 ms threshold does nothing for voice; it keeps its normal job (activating a focused button, typing a space in a text field, etc.).
- Space inside any text-entry surface (text, search, number, date, textarea, editable regions, sliders-as-text) is never claimed. Modified presses (Alt/Ctrl/Meta/Shift) are never claimed.
- Space on the map canvas counts as "background" and is reserved for push-to-talk even though the canvas is focusable.
- If the user changes focus (pointer click or Tab) during the 500 ms delay, the hold is abandoned. If the tab is hidden or the window loses focus, the hold is abandoned.
- If a control was focused when the hold was claimed, it is blurred first so the eventual key release cannot also activate it.
- A click on the mic button that races a held Space is ignored.
- Window blur or tab hide releases push-to-talk immediately.
- A claimed hold while the assistant is talking cuts it off (barge-in) - except during a radio-station handoff (see 2.7).

### 2.3 Connection sequence (each step cancellable)
1. Ask the local server for a session credential, passing the chosen model tier and auth mode. Cache disabled, redirects refused.
2. Server composes the full session configuration (model, reasoning effort, voice, instructions, tool list, context truncation, input transcription, voice-activity settings) and asks the cloud provider to mint a one-time client secret. 30 s server-side timeout.
3. The browser validates the secret (present; not already expired) and records which model/tier/caption-model the server actually used (echoed in response headers).
4. The browser opens a peer connection, attaches the microphone, and exchanges its session description (SDP offer/answer) **directly with the cloud provider** using the short-lived secret. Overall token+negotiation steps are bounded by a 30 s timeout and a lifetime abort signal.
5. Audio plays through a hidden playback element; level analysers start for mic and assistant.
6. Status becomes LISTENING.

Failure handling: a failed token, offer or network step produces a plain-language message, an entry in a bounded error history (last 30 persisted locally), and the ERROR state with the error tray; if the connection is still alive after an error the UI drops back to LISTENING so the user can retry. A stale start attempt can never leak into a replacement session (generation counting). Disconnects get a 6-second grace period before the session is declared lost.

### 2.4 Session configuration defaults (server-tunable)
| Setting | Default | Range / note |
|---|---|---|
| Standard model | OpenAI "gpt-realtime-2" | overridable |
| Mini model | OpenAI "gpt-realtime-2.1-mini" | overridable; about 3.2x cheaper per audio token, about 6-10x cheaper for text and image tokens (see 7.2) |
| Voice | "marin" | overridable |
| Reasoning effort | low | overridable |
| Context window | about 3,000 tokens after instructions | clamped 1,000-12,000; retention ratio 0.5 (clamped 0.1-1) |
| Turn detection | semantic VAD, eagerness low, auto-create response, **no auto-interrupt** | interruption is handled by the app (see 2.6) |
| Input transcription (captions of what the user said) | gpt-4o-mini-transcribe | can be turned off; billed separately and counted in the cost meter |
| Tool choice | automatic | |
| Safety identifier header | fixed local-dev tag | sent to the provider on minting |

Model tier is chosen by name only; an unknown/hostile tier string silently resolves to Standard and the response flags the fallback. The tier preference is stored in the browser and **applies next session** - a live session keeps the model it connected with.

### 2.5 Turn behaviour
- User speaks (or types a command into the debug/text path); the assistant may speak, call tools, or both. Tools run while it talks.
- **Progress narration while tools run and nothing is audible** (code-authored, not model-authored):
  - soft two-note "working" tick after about 0.7 s of silence (notes about 1.1 s apart at very low gain; only when the mic is muted, i.e. after a push-to-talk release - with an open mic a local sound could be mistaken for the user speaking, so open-mic sessions rely on the card's visual progress);
  - first spoken progress line after about 1.5 s still running;
  - at most 2 step lines, at least 3 s apart;
  - one "still working" line at about 8 s;
  - tick never longer than about 15 s per turn; after the last tool ends it waits about 3 s for the reply to begin.
  - Step vocabulary: finding places, tracing outline, looking up a place, flying, turning on a layer, loading aircraft, picking the nearest. Lines are silent if the model already spoke a preamble. A progress line that nobody has heard is retracted when the tools finish; an audible one finishes.
  - Progress responses are capped at 60 output tokens.
- **Preamble rule given to the model:** instant commands (layers, styles, HUD, zoom, panels, basemap, radio, camera nudges, lookups) get no preamble. Slow multi-step work may get one sentence of at most 12 words naming the steps, in the same response as the tool calls.
- **Multiple changes in one utterance** ("night vision and flights on"): all matching tools are called before any confirmation; a partial subset is never confirmed; failures are named.
- **After tools:** the assistant speaks the result's prepared line once, lightly rephrased at most; lists/rankings add up to three named items in returned order; caveats (stale, degraded, unavailable, lower bound, partial) are preserved; no filler/hedges; exact numbers never get "approximately".
- **Typed input supersedes older spoken intent:** the in-flight response is marked superseded, its late tool calls are refused with a terminal "superseded" answer (every tool call must still receive exactly one final output or the model stalls), and its queued follow-up is dropped. Bursts of typed commands coalesce into one response.
- **Duplicate tool calls** (same call/item/arguments) within a 2.5 s window are de-duplicated. [CENSUS] Response creation is queued behind any active response and deferred while the user is mid-speech, and each tool carries its own follow-up instruction so the spoken confirmation echoes the RESULTING state (for example the layout that ended up active), never merely the request.
- **Two ordering rules that prevent deadlock and lost spend** [CENSUS]: (a) every tool call must be answered exactly once, even a refused one - a call arriving from a superseded response still receives a terminal "superseded" output, because an unanswered call leaves the conversation waiting forever and deadlocks the model; that refusal output must itself not trigger a new response. (b) The cost tracker may be replaced only when the session is fully settled (not active, no data channel and no peer connection); see 7.5. Mechanism detail: 13f GAP-4.
- Tool calls are refused once the spend cap has tripped (see section 7).
- The model is told to ignore instructions embedded in feed text (callsigns, place names) - such text is untrusted data.

### 2.6 Interruption and barge-in
- Speaking over the assistant: because automatic interruption is off, the app itself detects user speech and truncates the assistant reply to what was actually heard (the cloud conversation record keeps only the heard portion).
- Claimed push-to-talk hold while it speaks: stops playback immediately.
- Superseded responses are tracked (memory of the last 8 spoken responses) so their late audio or tool calls cannot affect the new turn.

### 2.7 Radio hand-off (voice vs. internet radio)
- Asking for a station: the assistant first says a short confirmation ("Turning on the radio" - never "it is playing"), the radio is held muted, and only when voice playback is verified quiet does the radio start; the voice session is closed before the radio volume is restored.
- Starting any voice reply silences/pauses the radio; holding Space pauses it too.
- During the hand-off a Space hold does not interrupt.
- Radio control calls are "silent" tools: they do not trigger a separate spoken follow-up beyond the confirmation.

### 2.8 Point-and-ask (deixis)
- [OBSERVED-IN-SOURCE][FROM-TESTS] The pointer is tracked continuously over the globe only (not over a panel sitting on top of it); hovering still (moving less than 12 px for 250 ms) "picks" what is under it: the nearest entity, else a ground point. Picking is lazy (on dwell and at snapshot time, never on every mouse move); a snapshot reuses a dwell pick only if it is under 1 s old (contacts move).
- A pick counts as a referent only while **fresh**: pointer over the map, at least 8 px away from the map edge (a cursor parked in a corner is not "this"), and moved within the last 30 s.
- **Reticle:** a subtle non-interactive ring is drawn at the picked screen position while the turn is live, so the user sees what was understood before the answer. It differs slightly for an entity pick versus a ground pick. It marks where the user pointed, not a tracked world position (it does not follow a moving aircraft). It disappears when: any camera-moving action starts (fly to place, track, relative zoom, globe view, camera motion, route flight, frame overhead, nearest-aircraft selection, cockpit), the reply completes (after a 1.5 s linger, and only once no tool is still running), the session goes idle or errors, or 10 s have passed at most. Clean-UI and presentation modes hide it.
- The card shows a "THIS: <label>" chip (label capped at about 32 characters) for an entity pick, or a "HERE: <lat>, <lon>" chip (two decimals) for a ground pick.
- The pick used for a question is the **snapshot taken at the moment the user decided to ask** (key-down for push-to-talk; recent pointer within 5 s for open-mic). Snapshots are fresh for 30 s and held for 20 s.
- Words like "this", "here", "that plane", "around here" make the model pass the marker word "pointer" (or a list index) instead of coordinates; the app substitutes the real target. If nothing was under the cursor the tool returns a plain sentence saying so. The model never copies coordinates.
- For a ground pick the "around here" radius is three quarters of the camera height, rounded to whole kilometres and clamped between 5 and 400 km (50 km if the height is unknown). A question that says "around here" uses that radius as its scope.
- Works in: entity context, tracking, flying, annotations, analyst scope.
- A 512-pixel crop around the pointer can be attached as an image (local/city zoom only, snapshot no older than 20 s).

### 2.9 Numbered references ("the second one")
- When a result lists items, the card numbers up to 5 of them. The model can pass "item n" (or -1 for the last) to fly to / track / get context for it. Only the **latest** result list is addressable; each new list replaces the previous one, and the list is session-scoped.

### 2.10 Visual grounding
- On demand (street-level scale, no structured identity available) the app captures the 3D view, (rendering twice to guarantee a fresh frame) downscales it to at most about 1.08 megapixels (1200x900 budget), JPEG-encodes it, and refuses to send it if the encoded size exceeds 200 KB, if no fresh frame was rendered, or if the frame is almost entirely black.
- It is sent with a strict instruction not to invent labels; only legible text counts.
- Only **one** image is retained in the conversation; the previous image is deleted before a new one is added (images are the costliest items and are re-billed every turn).
- The capture includes anything drawn on screen (including camera feeds and imported files shown on the globe).

### 2.11 Context freshness
- Before answering location questions the assistant calls the scene-context tool: coordinates, view-scale class, street/city/country, nearby named places, landmarks, enabled layers, selected entity (see 4.1).
- No client-side per-turn trimming of conversation items (it would defeat the provider's prompt cache); trimming is done by the provider via the retention policy above.

### 2.12 Stopping and teardown
Order: invalidate pending work and actions; close data channel and peer connection; stop audio analysis; release the microphone; clear conversation memory (analyst follow-up memory, list references, pointer snapshots). A full removal also detaches keyboard, annotation and radio observers. Late offers, peer callbacks, screenshot completions or meter frames cannot enter a replacement session. Cost cap and Stop use the same path.

### 2.13 Diagnostics (developer-facing)
- Every client/server event is posted to a local log sink (fire-and-forget; never affects voice). Secrets, bearer tokens, client secrets and image data are redacted. By default transcripts, typed/spoken text and tool arguments/results are **omitted**; an opt-in switch includes them. The log rotates at 32 MB keeping one previous generation, never expires, and the sink is rate-limited (120 per client / 400 global per minute).
- Per-turn latency and silence spans (speech end, first audio, longest silence) are logged.
- Last 30 errors are kept in browser storage and a diagnostics getter exists on a debug handle.
- A recorded-audio test mode exists for both open-mic and push-to-talk (checks mic acquire/release, tool outcome and cleanup).

---

## 3. Providers and authentication

| Option | Credential | Where it lives | Notes |
|---|---|---|---|
| **OpenAI API key (default)** | Operator's key | Server-side only | Metered per use; dollar estimate + warning + hard cap apply. Missing key -> token request answers "service unavailable" and the mic reports an error; the rest of the app works. |
| **ChatGPT-login option (experimental, opt-in)** | The user's existing local ChatGPT/Codex sign-in | Read from the Codex tool's own credential file on the same machine; never written or refreshed by the product | Chosen in Provider Settings ("USE CHATGPT OAUTH"). If no usable sign-in exists, the product launches the Codex command-line browser sign-in (one pending attempt at a time, two-minute timeout, cancelled on server stop) and selects the mode when it finishes. Token is used server-side only to mint the same kind of short-lived client secret. Only answers to loopback connections; status/login/token routes never return the access token. Expired token -> "re-authentication required". Cost shown as "COST UNKNOWN" with only counts of responses/tokens/caption transcriptions; **no dollar estimate and no spend cap**. A deployment flag can forbid API-key sessions and force this mode (fails closed if no sign-in). Documented caveat: the vendor has not confirmed this usage contract for the realtime service; it may stop working or violate account terms. The selected mode is a browser preference; focus stays on the control when it changes. |
| Pluggable backend | n/a | n/a | The session/connection layer is protocol-independent; alternate speech back-ends can plug into the same controls and tool runner (this is the hook for a local-voice stack). [CENSUS] The seam is two replaceable operations (obtain a token, negotiate the media connection); the connection owns the microphone, peer and data channel, while turns, radio, input, cost, viewport and diagnostics each have a single owner. |

Short-lived secret: the browser receives only a one-time client secret with an expiry; connection is refused if it is already expired. Both long-lived credentials stay on the server.

Rate limit on minting and on the HUD-summary endpoint: 30 requests per minute per client IP by default (see 7.4).

---

## 4. Catalogue of voice-callable tools

Two families: (A) **30 app-control/interrogation actions** executed in the browser against the live app; (B) **25 read-only world-query tools** from the shared catalogue, available without any layer being on. A voice session therefore offers **55 tools** in total. [OBSERVED-IN-SOURCE] Counts were re-derived from the schema list (30 actions, of which one is the analyst query) and from the shared catalogue (40 definitions, 15 switched off for voice = 25). DOC vs CODE: the project's status notes still contain an old line saying voice has 28 tools and an old milestone history of 17 then 20; the README and the newer status paragraph say 30 actions, which matches the code; the 28 and the 17-to-20 figures are stale history. All argument schemas are strict: no extra fields, enumerations and numeric ranges enforced, string length caps.

**One tool-name namespace, actions win.** [CENSUS] The voice session lists the 30 browser actions first and then appends the catalogue's function-style tools, skipping any name an action already uses; so where both exist (satellite passes are the example) the action wins and the catalogue version is never offered. Calls are routed by name: action names go to the browser action runner, everything else to the catalogue. See 4.6 for how the catalogue is built and joined. Mechanism detail: 13f GAP-2.

Common guards across A:
- Every call gets exactly one terminal result (success, refusal, cancellation or "superseded").
- While **cockpit mode** owns the camera, tools that would move the camera (track, fly, flight to place, frame, camera motion) are *refused by design*; the model is told this is correct and must not retry - exit cockpit first.
- Starting any navigation releases the previous motion owner; manual pointer/wheel input cancels automatic moves.
- Drawing mode owns the pointer: tools that rely on pointer picking respect that claim.
- Results carry a prepared spoken line ("say") of at most about 25 words and a card payload (title, chips, lines, numbered items, sources/notes). a null spoken line means "continue naturally".

### 4.1 Navigation and camera

| Purpose | Inputs | Output / behaviour | Guards |
|---|---|---|---|
| **Fly to a place** | one of: preset city (8: Austin, San Francisco, New York, Tokyo, London, Paris, Dubai, Washington DC), free-text query (landmark/city/country), latitude+longitude, "pointer", numbered item; optional framing close/overview; optional explicit range 100 m to 20,000 km; optional wait-for-arrival | Result: found/not, label, arrived flag, range. Spoken: "Over X" if arrived else "Flying to X"; failure: "Couldn't find X". Whole-country/whole-city framing and close landmark framing are automatic unless a numeric range is requested. | Refused while cockpit owns camera. Flying while tracking ends tracking. Cancelling returns not-ok. Geocoder chain (offline coordinates/bundled names, Google if configured, Photon, then Nominatim) with 12 s overall bound. The voice schema lists exactly eight preset cities; the on-screen LOCATION tray has nine pills because Tallinn is tray-only (voice reaches it only by free text). [CENSUS] |
| **Select nearest aircraft near a place** | civil or military layer; place by preset, free text or coordinates | One atomic step: fly there, wait for arrival, enable and load that aircraft layer for the new viewport, exclude on-ground records, select/follow the nearest airborne aircraft. A healthy fallback feed counts as valid. | Does *not* open contacts or cockpit. The model is told not to chain other fly/layer/track tools for the same request. Progress lines: finding, flying, turning on layer, loading aircraft, picking nearest. |
| **Relative zoom** | direction in/out; amount little/medium/lot | Confirms only on success. | Cannot reach full-globe view. |
| **Full-globe view** | none | Absolute pull-back to about 18,000 km altitude keeping the region centred ("globe view", "whole earth"). | |
| **Camera motion (drone verbs)** | motion: orbit / pan / tilt / rotate / stop; direction left/right/up/down; speed slow/normal/fast; mode once or continuous | One-shot nudges: eased, under a second, bounded (about 30 deg orbit, 15 deg tilt/rotate, about a quarter of view height pan). Continuous speeds: orbit 2/6/15 deg per second; tilt 4/10/20; pan 8/20/45 percent of view height per second. Orbit circles the screen-centre ground point holding range and pitch. Pitch clamped between nearly straight down and slightly below horizontal. Stop reports whether anything was moving. | One continuous motion at a time; any manual input, new navigation or tool restores manual control cleanly. A bare "stop" while a motion or route flight is active means stop the motion (not the scene, not tracking); if nothing was moving and an entity is tracked, the model then stops tracking. |
| **Fly a drawn route** | optional route label match; speed slow/normal/fast | Cinematic dolly along the newest (or named) route annotation: trapezoid speed profile with eased ends, mean speeds about 20/40/90 m per second, banked turns up to about 10 degrees, pitch locked about -32 degrees, gentle altitude breathing, heading leading the path, minimum duration 0.5 s. Roll always levelled on any exit. Reduced-motion preference removes roll and altitude shaping. | Fails with guidance if no route is drawn. Ground clearance: arms for up to about 1.2 s waiting for terrain data, then probes rendered surface, otherwise holds launch altitude - never descends blind. |
| **Frame what is overhead** | target: aircraft / military aircraft / satellites / ships; optional radius | Pulls back and angles the camera so nearby entities are visible together. Default radius: 150 km aircraft, 120 km ships, 3,000 km satellites. Speaks the count framed. | Refused in cockpit. |
| **Track an entity** | query (callsign, hex address, registration, ship name or MMSI, satellite name or NORAD id, "biggest fire", or "pointer") or numbered item; optional layer | Camera locks on, trail drawn, telemetry card. Match tiers: hex exact, callsign exact, registration exact, callsign prefix, registration prefix, callsign substring, registration substring; ties break on hex ascending (always commits to one). Separators ignored in registrations. | Searches all enabled layers if layer omitted (a wrong layer gives "nothing matched"). Refused in cockpit. Not for non-trackable layers. |
| **Stop tracking** | none | Stops follow; clears a selected ship. | |

### 4.2 Console operation (layers, panels, display)

| Purpose | Inputs | Output / behaviour | Guards |
|---|---|---|---|
| **Turn a data layer on/off** | layer (from a manifest of every shipped layer with spoken aliases such as planes/aircraft, ships/boats/AIS, fires/wildfires, quakes, bases, datacenters, dams, cables, buses/trains, bikes, cameras/webcams, license-plate readers, radio, street level, weather radar/clouds/lightning/wind/cyclones, recent imagery, directions); on/off | Spoken: "<Layer> on/off", with feed tag if stale/degraded/unavailable ("on, but its feed is unavailable"). Card chip shows feed state and source. | "Satellites" always means the satellites data layer, never a basemap. "Vessels" misheard as "visuals" still means the ships layer. Context-mode exclusivity can refuse enabling other layers (space-missions isolates its data). Layer lacking a required key reports unavailable with the reason. |
| **Open the layers menu / scroll to a row** | optional layer | Opens the menu and highlights the row without toggling. | "Show me the X layers" = this, not enable. |
| **Open/close a panel** | panel (data layers, location bar, display controls, CCTV, radio, scenes, post-processing toggles, global context, weather, recent imagery) and open/close | Aliases accepted. | "Open Context" opens the parent panel only; does not activate a sub-mode. |
| **Context mode** | off / contacts (alias "flights") / space-missions (alias "missions") | Contacts shows the roster of everything within 250 km of the subject; replies report a window count. Mode names reported in the vocabulary the user used, with the internal id kept separately. | Only on explicit request; selecting an aircraft does not imply it. |
| **Cockpit** | action: enter / exit / previous / next / status; optional contact filter (civil, military, ships, installations) and aircraft class (e.g. helicopter) | Enter establishes contacts itself, then rides a selected/tracked aircraft; next/previous cycles nearby contacts with filters. | Selecting an aircraft never enters cockpit. Camera-moving tools refused while active. |
| **Visual style** | normal / retro (CRT) / surveillance (night vision) / thermal (FLIR) / anime / noir / snow | Reports resulting style. | Only for style names or "filter/effect/look". |
| **Heads-up display** | visible on/off/auto; layout from the four layouts (tactical, operator, minimal, cyber) | Confirms resulting layout. "Auto" restores style-driven show/hide. | |
| **Cyber sonar** | on/off; rings 3-12; range 60-120 percent; power 0-100; contact-opacity floor 35-100 percent; sector 8-60 degrees | Returns the saved settings and whether the map sweep / contact sweep is actually active (saved setting does not equal active effect). | Requires the Cyber HUD layout; never switches layout, shows HUD or enables sonar as a side-effect. If both are requested, layout is set first, awaited, then sonar. Sonar is not detection. |
| **Detection overlay** | on/off; profile sparse/balanced/dense; density percent (snaps to 0/25/50/75/100 and derives the profile; "panoptic" is a legacy alias of dense); allocation elastic or weighted | Confirms resulting density/profile. | |
| **Basemap / imagery source** | photoreal (Google 3D), bing-aerial, bing-labels, esri-imagery, osm | Confirms resulting source or refusal reason. | Needs an explicit stack name. Unavailable stacks are refused with the same reason the UI shows. Retired road style resolves to OSM. |
| **Post-processing** | bloom on/off + intensity 0-200; sharpen on/off + intensity 0-100 | Confirms values. | Older links used a 0-100 bloom scale and are migrated. |
| **Scenes (cinematic playback)** | list / play (by id or partial title) / stop / next / status | Play starts a single named scene and returns immediately. | Playback takes camera ownership; see doc 09. |
| **CCTV panel control** | enable, disable, select by name/id, next, previous, nearest, focus, coverage toggle, viewshed (colour-coded coverage volumes), calibration adjust, projection toggle, auto-hop; optional explicit on/off | Operates the camera layer and panel. | The CCTV layer must be enabled first. Omit explicit flag to toggle. |
| **Radio control** | enable, disable, play, resume, pause, stop, next, previous, volume 0-100, select, status; category (all, news, talk, weather, public-safety, aviation-marine, traffic-transit, music); place by preset/free text/coordinates; country; station-name text | "Select" is used whenever any qualifier (category/place/country/name) is present; "play" only for an unqualified "turn on the radio"; "enable" only reveals station markers with no audio. Selecting never moves the camera. | Silent tool (confirmation is spoken by the hand-off, see 2.7). |
| **Report current view state** | none | Camera, style, context mode, cockpit, HUD (incl. sonar), detection, basemap, post-processing, scene playback, tracked entity (incl. transponder emergency broadcast wording), enabled layers, contacts-window counts. | Emergency flags are broadcasts, not confirmed incidents; quote the broadcast text and never infer a hijack or its resolution. |

**Named-view shorthand and disambiguation (instruction-only, no extra tool).** [CENSUS] "Show me global infrastructure" is turned into three layers on plus a pull-back to the globe view; "environmental mode" into fires plus earthquakes plus globe view; vaguer phrasing is treated as an ordinary question. A fixed table resolves confusable words: vessel words mean the ships layer unless a visual style is named; a basemap change needs an explicit stack name; "satellite(s)" means the data layer except for recent/latest imagery or the weather satellite; "orbit" means camera orbit.

### 4.3 Annotation whiteboard

| Purpose | Inputs | Output / behaviour | Guards |
|---|---|---|---|
| **Annotate the map** | 1-24 marks per call; each: type (pin, highlight, area, arrow, route, label); target place name (<=200 chars) or "pointer" or coordinates or screen position (0-1); for routes 2-12 ordered waypoints and mode (walking, driving, cycling; default walking); arrows need a destination; colour (primary, amber, cyan, green, red); label (<=120); footprint flag; intent ("the thing itself" vs "around the thing"); entity kind (building, compound, district, street, point feature); fly-to flag; persist flag (default keep; otherwise fades after about 20 s) | Resolver turns names into real anchors and **real outlines** (building footprints, campuses, districts, state/county/country boundaries from bundled administrative polygons - no network for "Texas" - natural regions such as "the Alps", bundled neighbourhoods, or online map data); routes follow streets and return distance and time; arrows return straight-line distance. Marks drape on terrain/3D tiles and stay anchored while orbiting; route dashes animate. Partial failures are reported honestly ("couldn't find X"); a straight "direct line" fallback is labelled; an outline still being traced is flagged pending and later confirmed/corrected by a system note. | Up to 120 live marks; duplicates detected; marks accumulate and persist across flights/topics; never auto-clear. Deferred outline upgrades run two at a time; stale work is dropped on clear/new topic. Region-scale natural features never get a lone point pin (use area). Navigation-only requests do not draw marks. Without a detail-feature source the mark keeps its pin and says detailed outline is unavailable. |
| **Clear annotations** | none | Erases all marks. | Only on an explicit user request to clear/reset. |
| *(Steering hints)* | Intent ("the thing itself" or "around the thing"), entity kind, footprint flag, screen-fraction coordinates [CENSUS] | These steer which outline is sought: compound-sized outlines for airports, hospitals, universities, parks, stadiums and campuses; the surrounding envelope for districts; a mark can also be placed purely by pointing. Airport-to-airport distance arrows rely on the centre of the compound-sized outline. | Mechanism detail: 13e gap G8. |
| *(Manual equivalent without voice)* | Draw tool: Area / Line / Pin, click vertices, double-click or Enter finishes, Backspace undoes a vertex, Esc cancels then exits; up to 512 vertices; ring auto-closed; live length/area readout | Same persistence and clearing as voice marks; line labelled with length (never travel time). | Holds the pointer exclusively while open. |

### 4.4 Interrogation (answers without moving the map)

| Purpose | Inputs | Output / behaviour |
|---|---|---|
| **Scene/entity context** | scope auto / selected / in view / pointer; numbered item; optional layer; limit 1-12 (default 5) | Selected object's properties (aircraft: callsign, operator, type, route read **only from returned fields**, airport codes verbatim, missing items named not inferred), or up to a shortlist of visible entities ranked by distance (only when camera altitude is under about 100 km), plus basemap context: view-scale class (global, continental, regional, metro, city, local); reverse-geocoded place (centre up to 750 km altitude; viewport samples up to 3,000 km); seven-point viewport sampling; Google nearby places within 25 km; matches against known landmarks; coarse country fallback. Assembly capped at 1.5 s with cached fallback. Latest selection from any layer wins the single "selected" slot. [CENSUS] Invariants of that slot: every selectable tracking layer (civil and military aircraft, satellites) writes one shared slot on select, poll and deselect without firing the generic selection event (the two event lanes would otherwise fight over one subject); there is exactly one record per layer, so a frozen snapshot of a contact that has since moved never enters the visible scan; when a satellite catalogue rebuilds, the subject is re-resolved and released if it has vanished. |
| **Analyst query** | see section 5 | |
| **ISS pass** | optional observer lat/lon (default: camera position); minimum peak elevation 5-60 degrees (default 10) | Next pass: rise time and direction, peak elevation, duration, estimated visibility. |
| **Any-satellite pass** | satellite by exact NORAD id or name; optional observer; minimum elevation; visible-only flag | Searches the next 24 hours for a satellite in the loaded catalogue. Ambiguous names return candidates and the assistant asks which. "Visible" = satellite sunlit and observer's Sun at or below -6 degrees; ignores weather, brightness, element age. Needs the satellites layer to have loaded once. |

### 4.5 World-query tools (shared catalogue, voice surface)

These work anywhere in the world, do not change the map, and need no layer on. Location tools take an area: a place name, a bounding box (may cross the antimeridian), or point plus radius. Lists default to 25 rows (max 200) and report total / returned / truncated.

| Purpose | Notes |
|---|---|
| Recent orbital launches (30 days) | Provider, rocket, pad, mission, outcome. |
| Aircraft track | Recent positions of one aircraft, thinned to 200 points. |
| Aircraft type, registration and flight route | From a community database; route data carries a no-copy notice. |
| Ship track | Recent positions of one ship, thinned to 200 points. |
| Public camera search | Nearest-first list of public traffic/city cameras (no image on voice). |
| Plate-reader camera search | OpenStreetMap-mapped locations only, US/Canada area up to 3 degrees. |
| Radio station search | By area and/or search terms; returns stream targets. |
| Place search / places nearby | Needs a Google key; reports when none is configured. |
| Route planning | Walking/driving/cycling over OpenStreetMap with distance, duration and simplified path. |
| Bike-share stations | Live bikes and docks. |
| Transit vehicles | Live positions, optionally one route. |
| Traffic flow | City-sized area: speed vs free flow, congested and closed road length by road class (needs traffic key). |
| Current weather | Conditions, temperature, feels-like, precipitation, cloud, wind, visibility. |
| Forecast wind | 10 m model wind at a point. |
| Submarine cables | By area or name. |
| Event pack (Nepal flood) | Evidence trail, flood path, imagery dates. |
| Regional brief | What/where a place is, weather, recent headlines. |
| Tropical cyclones | Active storms, optionally in an area. |
| Fire perimeters | US wildfire perimeters, largest first. |
| Terrain height | Up to 20 points: ground, geoid, ellipsoid heights. |
| Military installations | OpenStreetMap sites in an area up to 10 degrees per side. |
| Map features | Admin areas, parks, land use, water, monuments within 2.5 km (needs a configured detailed-map source). |
| Situation brief | Weather, earthquakes, fires, aircraft, ships, cyclones for an area, section by section; unavailable sections are marked. |
| Military awareness | Military and other aircraft, ships and installations within 250 km of a point, by section. |

**Public-camera catalogue rules (shared by the voice camera search and the MCP camera tools).** [OBSERVED-IN-SOURCE]
- The camera catalogue is assembled from independent regional "packs" of public traffic/city cameras (15 exist in the reference: Austin, California Caltrans, London TfL, Ontario, Finland, British Columbia, Texas DOT, Tallinn, Estonian road-weather, Warendorf, New South Wales, Calgary, Delaware, Norway, New York DOT). Each pack can be switched off on its own; all but New York are on unless switched off, and New York is opt-in only because its feed needs the operator's own data-sharing agreement.
- Each pack has its own default cap (for example Austin 250, Caltrans 300, TfL 250, Texas DOT 500, Ontario 1,000, Finland 300, British Columbia 250, Tallinn 255, Estonian road-weather 179, New South Wales 250, Calgary 220, Delaware 300, Norway 900, New York 250), nearest-to-anchor-city first; the whole catalogue has a ceiling of 4,000 cameras (adjustable between 8 and 5,000). When packs together exceed the ceiling, it is filled round-robin, one camera per pack per turn, so no region is wiped out; a duplicate camera id is won by the later pack.
- Each pack fails on its own; one outage never empties the catalogue.
- Search honesty: results list nearest first with name, city, provider, position, facing, feed type and credit. If a pack serves only part of what it holds in the searched area, the spoken line names it ("N of M from <pack>"), and the data marks the answer as not complete. An empty answer means "the catalogue has no cameras there", not "none exist". Voice receives no image; image snapshots (JPEG/PNG/WebP, at most 3 MiB) exist only on the MCP surface.

Deliberately **not** on voice (to avoid duplicates or because voice receives text only): link-to-app tool, panel loader, ISS/satellite pass duplicate, image-returning tools (weather map image, recent satellite image, camera snapshot), HUD caption, and the aircraft/ship/quake/fire/infrastructure/satellite-overhead lookups that the analyst query already answers from loaded layers.

Rule for the model: for aircraft, ships, quakes, fires, datacentres, dams and satellites use the analyst query over loaded layers (turn the layer on first if the user wants to see them); for other places or layers that are off use the world-query tools; to *show* something also call the matching app action.

### 4.6 How the shared catalogue is built and joined to voice [CENSUS]
- A tool record is: name, kind (query or action), title, description, an input schema in a restricted JSON-Schema subset (unsupported keywords are rejected when the tool is defined), the services it requires, and a run step that returns a one-sentence summary plus structured data (and optionally images). Query tools come from portable source factories that request data through an injected fetch against relative paths, so the identical code runs in the browser, on a server and in tests.
- The catalogue is composed from core tools plus app tools. A name clash fails the build unless the later tool is explicitly declared a replacement. A tool is dropped when any service it requires is absent (a deployment with no places key simply has no place search, and combined briefs mark those sections unavailable). Call interceptors wrap each call outermost-first; composite tools (situation brief, military awareness) call other tools through the same catalogue, so replacements and interceptors apply to the sub-calls, which are tagged with the parent tool.
- Per-surface exposure is a table that switches individual tools off for voice or for MCP (40 definitions: voice offers 25, MCP lists 30, 29 model-visible).
- The browser does not build the catalogue at start. The first time voice calls a non-action tool, the page builds the services over its own fetch, composes the catalogue, and routes by name, passing the call's abort signal so a barge-in cancels in-flight lookups. Results are converted to an ok flag, tool name, summary and data; image results are not sent to voice, only counted.
- Portable modules (infrastructure, application, view, the catalogue and MCP, source factories, selected server providers) are separately importable and checked by a boundary test to stay free of DOM, engine and Node-only imports; this is why the same tool code serves browser, server and tests. Mechanism detail: 13f GAP-2 and GAP-10.

### 4.7 One manifest drives voice reach [CENSUS]
A single plain-data manifest lists every layer with its spoken aliases and, separately, whether it can be toggled by voice, can register in-view context, can be tracked, and which analyst fields (with type and unit) it exposes - or why it has no analyst query (GNSS integrity is cells, not countable records). The server reads it to build the model's enumerations and the browser reads it to resolve aliases; a test fails if a catalogue layer is neither listed nor explicitly excluded. Effect: a new layer is voice-reachable the day it ships. Mechanism detail: 13f GAP-3.

---

## 5. Analyst answers (counts, filters, rankings)

**What it answers.** "How many flights over Texas", "anything above 40,000 feet", "biggest fire near LA", "ships headed to Oakland", "next rocket launch", "how many datacenters in view". It never moves the camera or starts tracking.

**Layers and fields it can query** (field types: number with unit, text, flag, time):
- Aircraft (civil & military): altitude (m), speed (m/s), vertical rate, heading, callsign, address, origin country, operator, aircraft class, military flag, on-ground flag, squawk, emergency flag/kind/source/broadcast text, last contact time; civil also route origin and destination.
- Local receiver aircraft: altitude, speed (kt), heading, callsign, address, on-ground.
- Ships: speed (kt), course, name, MMSI, ship type, destination.
- Satellites: altitude, speed, name, NORAD id, class, group (distance is ground distance, not "overhead").
- Rocket launches (past 30 days and upcoming): name, provider, status, site, mission, launch time, hours until.
- Earthquakes (last 24 h): magnitude, depth, place, time.
- Active fires (last 24 h): radiative power (MW), confidence, satellite, sensor, acquisition time.
- Fire perimeters: acres, containment %, personnel, cost, name, state, county, cause, behaviour, complexity, discovery time.
- Cyclones: name, classification, basin, wind (kt), pressure (hPa).
- Transit: mode, route, speed, course, status, occupancy.
- Bike share: bikes/docks available, capacity, name, city, renting flag.
- License-plate cameras: operator, manufacturer, type, facing direction.
- Military installations: name, class, kind.
- Datacenters: name, operator, capacity. Dams: name, operator, river, output.
- No analyst query (map overlays or other tools): GNSS integrity, weather radar / clouds / lightning, wind, traffic, cameras (use the CCTV tool), radio (use the radio tool), street level, recent imagery, directions, submarine cables.

**Arguments.** Layers (list); scope (view, region by name, radius with optional centre, anywhere, pointer); filters (ANDed; operators greater/greater-or-equal/less/less-or-equal/equal/not-equal/contains; typed values; altitude in metres, so 40,000 ft = 12,192 m; times as ISO or "hours ago"); sort field or "distance" and direction; limit (hard maximum 50; the voice action asks for 12 when the model gives none, while the underlying engine's own default is 10); follow-up flag.

**Scope rules (WHERE order).** (1) a named place; (2) explicit pointing ("around here" = pointer, never view); (3) the contacts subject while contacts mode is active, or the selection when referred to; (4) the current view. A selected datacentre/dam/fire/cable never silently becomes the centre. The model must never invent centre coordinates. There is no drawn-area scope yet - the assistant says so.

**Counting contract.** Contacts active: "near/nearby/how many aircraft" = the contacts window (panel numbers, 250 km), spoken verbatim; a requested list/ranking still needs a real query; the window count only applies if the requested radius matches. Contacts off: "nearby" = in view; "near <place>" = radius around the place. Every count names its scope in words ("42 in your window", "8 in view", "30 within 250 km of Austin"). Counts are spoken exactly - never rounded or hedged. Different scoped numbers are not contradictions. [CENSUS] Three numbers can legitimately differ: the Contacts cohort within 250 km, the loaded records for the asked scope, and the layer-wide loaded total. One proximity snapshot is computed once and read by both the panel and the analyst, because they once diverged in the field (111 on screen against 15 spoken). Centre-of-query precedence, spoken aloud: explicit place, then pointing, then the Contacts subject (a selected non-contact never silently becomes the centre), then a named entity, then the current view; Contacts active with no subject uses the view.

**Honesty fields.** Results say whether the count is complete (otherwise "at least N" and rankings cover only the records examined), which layers were unanswered (partial), the feed state of each layer (stale/degraded/unavailable words are spoken; fallback feeds are valid data and are named only when asked), whether the flights layer is viewport-loaded (a note that counts cover loaded data). A cohort that cannot answer reports "unknown", never zero. Saturated cohorts (20,000-row retention per layer) are reported as lower bounds. Per-layer record cap for queries is very large (250,000) with default 2,000 for satellites/datacentres/dams in the bounded lists (omitted records can change nearest/count answers).

**Error codes the model understands:** layer off (offer to turn it on), still loading (first poll warm-up window of about 45 s), follow-up mismatch (ask again without follow-up), unknown field / unknown scope / bad value (retry with allowed values).

**Follow-up memory.** "Which of THOSE ..." with follow-up set re-filters the last answer's layers using only the new filter/sort; the memory is session-scoped and cleared on stop.

**Hand-off to tracking.** Items carry latitude/longitude and the aircraft address / ship MMSI so "follow the nearest aircraft" can run: ask for the nearest one (sort by distance, limit 1), then track it in the same turn. A lookup alone never fulfils a follow/track command.

**Emergency wording.** Aircraft emergency fields describe broadcast codes; kinds are general, unlawful, no-radio, downed, minimum fuel and lifeguard; the last two are treated as priorities, not emergencies. Time field means last transponder contact.

---

## 6. Intel summary generation (the HUD line)

- **What:** a five-word, intelligence-style readout under the HUD's "summary" slot describing the current view (e.g. a place plus a relevant live layer).
- **Cadence:** requested every 15 seconds while the HUD is visible and the view changes; typewriter animation on update.
- **Input to the model:** only labels - place, street, nearby places, names of enabled layers, plus per-layer feed provenance (state, source, count). Instructions forbid inferring from coordinates or inventing places.
- **Rules:** exactly five words, no punctuation/markdown; if any enabled layer is not nominal, one of the tokens STALE, DEGRADED, FALLBACK, LOADING or UNAVAILABLE must appear and the view must not be presented as live.
- **Model:** a small cheap text model (default "gpt-5-nano", minimal reasoning, output capped at 100 tokens); output is stripped to letters/numbers/hyphens and cut to five words.
- **Timeouts/fallbacks:** the browser aborts after 5 seconds and shows the deterministic telemetry summary instead. With no key configured the service answers a deliberate 200 "not configured" payload (not an error) and the HUD uses the telemetry summary. Upstream errors return a generic message (never the provider's wording).
- **Throttle:** shares the 30/min/IP OpenAI throttle (keyless responses do not consume it); 64 KB request cap.
- **Cost:** not metered by the voice meter.
- [CENSUS] The fallback telemetry summary is a deterministic local line (mode, band, locality tag, region, altitude, window, sun elevation); it says "NEAR <landmark>" only within 150 km of a catalogued point of interest (an eight-city catalogue), otherwise a sector coordinate pair, and carries a provenance tag when feeds are stale. Stale AI summaries are dropped if provenance no longer matches. Known upstream defect: its region logic labels Abu Dhabi as Africa.
- Cockpit also has a separate "regional briefing" (weather + headlines), described in doc 10.

---

## 7. Cost display and throttling

### 7.1 Live meter (API-key mode)
- Readout like "~$0.42"; sub-cent totals show "~$0.01"; a trailing asterisk marks an **incomplete** estimate (a response was still in flight at teardown so its usage never arrived). The asterisk is a see-note, not a direction claim (estimates can run high or low).
- Level colours: ok, warn (amber), cap.
- Tooltip: model name, response count, warn and cap values.

### 7.2 How money is computed
- Usage arrives per response (not cumulative) and is summed. Cached tokens are priced at the cheaper cached rate; uncached = modality total minus cached.
- Rates are looked up for the **model the server actually used** (echoed back), not the one requested. Unknown models are priced at the most expensive known rate (ranked by audio-output price) and a console warning is logged.
- Missing or partial usage detail is attributed to **audio** (the priciest modality) so uncertainty always errs high.
- Caption transcription is metered separately with its own rate table (token-billed and duration-billed models both handled; unknown models priced at the priciest).
- Rate cards (USD per million tokens), verified by the reference on 2026-08-18 and flagged "verify at release":

| | Standard | Mini |
|---|---|---|
| text in / cached / out | 4 / 0.4 / 24 | 0.6 / 0.06 / 2.4 |
| audio in / cached / out | 32 / 0.4 / 64 | 10 / 0.3 / 20 |
| image in / cached | 5 / 0.5 | 0.8 / 0.08 |

Transcription [OBSERVED-IN-SOURCE]: mini-transcribe 1.25 text in, 3 audio in, 5 text out; full transcribe 2.5 / 6 / 10; duration-billed 0.006 USD per minute. Cached transcription input is deliberately priced at the uncached rate, and transcription output is priced as text. The two voice-model rate cards and the transcription rates were re-verified against source in this pass and are unchanged. Ratio standard/mini: audio in 3.2x, audio out 3.2x, text in 6.7x, text out 10x, image in 6.25x.

### 7.3 Thresholds
- **Soft warning at $2:** amber readout and exactly one log line; session continues.
- **Hard cap at $5:** session ends through the ordinary stop path; readout says "Session ended - cost cap"; **no new tool is dispatched after the cap trips**; tools already running finish (no rollback of half-applied camera/layer/annotation changes).
- Either threshold can be disabled (0, negative, or "off"); a disabled value round-trips through storage so it does not silently re-arm; a corrupt stored value falls back to the defaults (never disarms the cap).
- Thresholds are a runaway guard (hot mic left open, feedback loop), not a budget.
- The meter's model binding and lifetime equal the session's: toggling STD/MINI mid-session only records the next-session preference and cannot reset accrued spend.
- ChatGPT-login mode: no dollar estimate, no cap; counts only.

### 7.4 Server-side throttles
- Per client IP per minute: 30 for token minting and HUD summary (default); 120 for Google Places lookups. Zero disables; an unreadable value falls back to the default. Over-limit answers a sanitised 429 with a 5-second retry hint. Client address is the raw socket address (forwarding headers are not trusted). These are in-memory, per-process guards and **not billing caps** - real protection is the provider's own budget/usage limits.
- Cross-site gate on these endpoints (foreign Origin, wrong fetch-site metadata, or proxy headers -> 403).

### 7.5 Cost-tracker lifetime and swap rule [CENSUS]
The meter is created at session start and is immutable until stop. It may be replaced only when the session is fully settled: inactive, no data channel and no peer connection. After an error status the transport can still deliver a late completion event carrying usage; swapping earlier would drop that spend, or let repeated tier toggles dodge the cap. Prices use the model id the server echoed, not the tier requested. Mechanism detail: 13f SYS-008 and GAP-4.

---

## 8. Privacy of audio and data

| Data | Where it goes | Retention |
|---|---|---|
| Microphone audio | Browser -> OpenAI Realtime directly over the encrypted peer connection. Not routed through the product server, not recorded by it. | Provider's own terms apply; product keeps none. |
| Assistant audio | OpenAI -> browser playback element. | Not stored. |
| Transcripts (user captions, assistant text) | From OpenAI to the card; user captions require the separately billed caption model. | Not logged by default; logged only if the content-logging switch is on. |
| Tool arguments/results | Executed in the browser; summaries (not raw secrets) go back to the model as tool outputs. | Omitted from the debug log by default. |
| Viewport screenshots | Sent to the model on demand only (see 2.10), one retained at a time. | Redacted from logs. |
| Scene context (coordinates, place labels, enabled layers, feed-sourced names) | Sent to the model as text to answer. | Short rolling window only. |
| Credentials | API key or sign-in token stay server-side. Browser sees only a one-time client secret. | Client secret is short-lived and redacted from logs. |
| Local debug log | Local disk, git-ignored, redacted, 32 MB rotation, never auto-deleted (the docs advise deleting it). | Operator-controlled. |
| Error history | Browser storage (last 30). | Local. |

No telemetry or analytics is sent to the product vendor by the voice feature. The app's stated ethics line applies to voice too: no named-person search, no face recognition, no tracking of individuals; camera frames are never analysed by the product, but a screenshot that happens to include a camera plane would go to the voice model if the visual-grounding path fires.

---

## 9. MCP server tools (external assistants)

The same tool definitions are exposed to external AI assistants (Claude, Codex, ChatGPT desktop) through the Model Context Protocol:
- **Transports:** local standard-input/output process, and a stateless local-only HTTP endpoint on the running app (accepts only direct loopback requests naming a loopback host on the port reached, a same-host Origin if any, no proxy headers, JSON bodies only, and only while launcher sharing is off). This is transport safety, not authentication - any local program can use it.
- **Protocol revisions supported:** 2025-11-25, 2025-06-18, 2025-03-26; operations: initialize, ping, list tools, call tool, list/read resources.
- **Tool surface for MCP** [OBSERVED-IN-SOURCE]: exactly **30 tools** are listed, out of the 40 definitions in the shared catalogue. Ten are withheld from MCP because assistants already do them or they add little without the globe: place search, nearby places, route planning, plain current weather, forecast wind, regional brief, radio station search, bike share, transit vehicles, and the HUD caption (also withheld from voice). The 30 are made up of: 16 that voice also offers (launches, aircraft track, aircraft info, ship track, public cameras, plate-reader cameras, traffic flow, submarine cables, Nepal event pack, cyclones, fire perimeters, terrain height, military installations, map features, situation brief, military awareness); 8 area lookups that voice answers differently via the analyst query (aircraft in an area, vessels in an area, find aircraft, find vessel, earthquakes, active fires, infrastructure, satellites overhead); 4 voice-excluded items (next satellite pass, weather map image, recent satellite imagery, camera snapshot image); the "show in the globe" tool; and a panel-only relay tool that is flagged to hosts as hidden from the model (so the model-facing count is 29). Tools whose backing service is not supplied are silently left out of a given deployment.
- **Results:** a one-sentence summary plus structured data (as text and as structured content); images as image content; lists capped (default 25).
- **Error classes:** invalid arguments, unavailable, unsupported, malformed, retry later; anything else reported without details.
- **"Show it on the globe"** tool: takes another answer's view, or an area/camera, layers, style, basemap, annotations, and an entity to follow (aircraft or satellite; ships cannot yet be followed from a link). Returns a shareable link everywhere and, in clients that support MCP Apps, a live embedded globe panel in the conversation. Answers that have something to show carry a ready-made view (matching layers on, area framed from above, a single aircraft/satellite followed) and a link.
- **Embedded panel:** a special build of the app loaded by the client; it fetches app files/data only through a panel-only relay tool that needs a per-server key shown in the panel page (a routing guard, not access control) and refuses settings, credential, model, MCP and internal development routes. Panel handles an MCP-Apps handshake, loads the app in embed mode once and posts later views to it; first load takes 10-20 s. It works around host quirks (images as data URLs, timer-driven drawing when a host hides the panel, in-memory 2D canvases).
- **Panel relay details** [CENSUS]: the relay tool is hidden from model listings by an app-only visibility flag; every call must carry the key embedded in the panel page. Large responses are compressed and split into parts that the panel reassembles, and the continuation state lives in the one server process, so consecutive calls must reach the same process. Path refusals are normalised before matching (letter case, percent-encoding, trailing-dot tricks) and cover setup, credential, model, MCP and development-server internals. Processes from one install share one key stored in a cache file resolved from the checkout: creation is atomic, a malformed file triggers a repair that exactly one process wins, a busy repair is waited on for at most 2 s before each process falls back to its own key (and logs that panels may fail across processes), and an abandoned repair lock needs manual removal; HTTP servers keep per-server keys. Host quirks handled inside the panel only: images and stylesheets delivered as data URLs (some hosts refuse blob images), an https base address supplied where code needs one, timer-driven drawing when the host reports the panel hidden, and in-memory 2D canvases so an off-screen panel cannot blank overlays. Windows with Claude Desktop cross-process confirmation is outstanding upstream. Mechanism detail: 13f GAP-5.
- **View object and its limits** [CENSUS]: a "view" is camera (position, altitude, heading, pitch), layers, style, map imagery, annotations and one aircraft, military aircraft or satellite to follow; it is written and read in the share-link format. The link is null when no app address is configured. Ships cannot yet be followed from a link, and the cockpit flag cannot ride in a link.
- **Result hygiene** [CENSUS]: string lengths are counted in Unicode code points; timestamps outside the representable date range become null instead of failing the whole row.
- **Transport vs Kawn:** the HTTP route refuses all requests while launcher sharing is on, and is transport safety only. Kawn must add real authentication on any hosted MCP.
- **Embed mode:** a page can show only the globe and change its view by posting messages; the app acknowledges and reports readiness. Framing by other pages is off unless the operator lists allowed ancestors.
- DOC vs CODE: the project's MCP setup notes and README say "about 30"; the code gives exactly 30 listed (29 model-visible).
- **Keys needed:** none for most; Google key for place search; traffic key for traffic; fire key for fires etc. (tools report "unavailable" when a key is missing).
- **Privacy:** tools reuse the app's own proxies and public feeds; they may spend the same provider quotas as the app.

---

## 10. Cross-feature interactions worth preserving
- Voice reads and writes the same state as the manual UI (layers, styles, HUD, scenes, annotations) through shared operations, so the UI always reflects what voice did and share links capture it.
- Share links carry camera, style, layers and one tracked target; voice "show me" tools build links from the same view description.
- Annotation marks created by voice and by the manual draw tool are the same objects.
- Scenes can be played by voice; tracking is never re-established by scene playback.
- Contacts mode, cockpit, tracking and scene playback compete for the single camera owner; voice tools respect refusals.
- First-run missions in the welcome panel include a voice-oriented "Live Contacts" path.

---

## 11. Voice-routing benchmark (developer harness) [CENSUS]

A command-line harness measures how reliably a speech or language model picks the right tool for a spoken phrase, with no app page involved.
- **Inputs:** every provider receives the production instructions and the production tool schemas (for the realtime provider, the exact session body the token route would mint), a fresh context per phrase, stubbed tool results, and up to four continuation rounds until the model stops calling tools.
- **Providers:** the primary realtime provider; a second cloud realtime provider whose native-audio model rejects text output (the harness requests audio and bills the spoken confirmation at the audio-output rate); a local model runtime; and any chat endpoint speaking the common chat-completion format.
- **Four suites:** core (the routing phrase table with semantic grading plus typed-argument checks); coverage (probes for the newer layers, each row graded correct, honest-refusal, hallucinated-capability or wrong-tool, graded twice - once with an original rubric against the original tool schema and once with a second rubric for the current schema); dialogue (two-turn exchanges where turn two depends on turn one, such as a follow-up filter); capability (area handles, imagery, map data and point-and-ask phrases).
- **Controls:** reusable output folder to collect several models, suite and phrase filters, a total cloud-spend budget, temperature, context size, a reasoning toggle for local models, and a summary mode. Cost uses the app's own rate table; unknown model ids resolve to the priciest known table; second-provider rates carry a dated verification; unattributed tokens bill as audio.
- **Speech lint:** a pure checker flags hedge phrases and "approximately" placed before an exact tool number, and counts spoken words against answer words per turn; it feeds the per-turn span record and benchmark grading to keep replies short and exact.
- **Recorded-audio checks:** the same recorded clip is run through open-mic and real push-to-talk, verifying microphone acquire and release, tool outcome, usage, cleanup and reconnect (plus a sign-in variant).
- **Kawn use:** run the same harness on candidate Arabic and English models; add Arabic, Gulf-dialect and code-switched phrase rows; keep the honest-refusal grade because it measures exactly the failure the product forbids (claiming a capability that does not exist). Mechanism detail: 13f GAP-1 and GAP-8.

## 12. Retired, removed and planned items relevant to voice and tools [CENSUS]

**Retired or stale (do not specify):**
- The reference's current-state notes still list weather radar under "not currently in runtime" although observed rain radar shipped (September 2026); a source comment also says three HUD layouts although there are four. Treat code and change log as authoritative.
- The earlier stale tool counts (28 actions; the 17-then-20 history) remain superseded, as noted in section 4.
- The experimental AI video-edit look was removed; the seven deterministic looks remain (the voice style enumeration lists those seven).

**Planned or announced, not built:**
- A spatially aware summarise-and-prune context policy for voice (specified, not built); today the context window is only provider-truncated.
- A voice timeline for layers (no layer has one).
- Following ships from a share link, and carrying cockpit state in a link.
- A shared panel surface contract (theme tokens, rail input, fixed header with bounded scrolling body); matters only if Kawn builds its own panel system.

**Not verified:** voice behaviour on iOS Safari and Android Chrome (WebRTC, the 500 ms hold on touch); whether any telemetry exists beyond the local debug log (none found by reading, no network capture).

---

## Kawn notes (UAE city-intelligence: Dubai, Abu Dhabi, Sharjah; Arabic + English, RTL)

**Keep**
- Push-to-talk by hold plus open-mic, with strict arbitration so tapping keys keeps normal meaning.
- Code-composed results: very short spoken line + richer card; numbers verbatim; stale/degraded/unavailable words always preserved; counts always carry a named scope.
- Exactly-one-result-per-tool-call, supersede-on-new-input, and "claim success only on success".
- Point-and-ask with pointer snapshots and list-index references; never let the model copy coordinates.
- Progress narration (tick, bounded spoken lines), radio-style hand-off ducking if audio features exist.
- Spend/resource guard concept (soft warning, hard stop, no new actions after stop) and honest "incomplete estimate" display.
- Redacted, content-free-by-default diagnostics; recorded-audio acceptance tests.
- Layer manifest as the single source for spoken aliases and queryable fields.
- Analyst contract (scope words, completeness flags, follow-up memory).

**Adapt**
- Replace the cloud realtime service with a local or UAE-hosted speech stack (see below); keep the backend-pluggable seam.
- Preset cities -> Dubai, Abu Dhabi, Sharjah (plus Ajman, RAK, Fujairah) and key landmarks/districts/free zones; Arabic and English aliases for every layer name, panel, style and place.
- Geocoder chain must accept Arabic queries and return Arabic and transliterated labels; offline gazetteer of emirate districts/communities.
- Distances/units: metric with locale-aware numerals; Eastern and Western Arabic digits understood and spoken correctly.
- Entity vocabulary: replace aircraft/ship-centric analyst fields with city objects (permits, construction sites, transactions, transit vehicles, air-quality stations, incidents) while keeping the same filter/sort/scope model.
- Intel summary: a short phrase in the user's language (not exactly five words in Arabic - define a word/length budget per language); keep the "must carry feed-state token" rule, localised.
- Visual-grounding screenshot rules need privacy review: UAE rules on imagery of people/vehicles/critical facilities.
- MCP tool surface: expose city-intelligence queries (zoning, project status, transport, weather/dust, air quality) with licences checked per source.

**Census additions (Kawn)** [CENSUS]
- Voice presets: the original has eight preset cities for voice and a ninth tray-only pill. Kawn voice presets follow the owner decisions: default location Dubai (not Austin), with Abu Dhabi, Sharjah, Ajman, Umm Al Quwain, Ras Al Khaimah and Fujairah, plus airport pills as in the location tray (Dubai airports, and London as a secondary group); free-text search stays bilingual. Keep the "tray has more pills than voice presets" distinction explicit, and add Arabic aliases for every pill.
- Default view per owner decisions is Scope on with feather 100 percent; voice scope and detection commands must read and write that same state, and "turn scope off" must stay one utterance.
- Adopt the voice-routing benchmark (section 11) for Arabic/English model selection before choosing a speech stack.
- Keep the define-once catalogue, action-precedence namespace and single layer manifest so a new UAE layer is voice- and MCP-reachable the day it ships.
- Keep the two ordering rules (terminal answer for every call; cost tracker swap only when settled), adapted to a compute-budget meter.
- The deterministic summary needs a UAE point-of-interest catalogue and correct region logic (the upstream labels Abu Dhabi as Africa).
- Arabic signage reading is a visual-grounding upgrade; scene-context needs a UAE landmark table (original covers eight cities).

**Skip**
- ChatGPT-login/Codex-token reuse path (unsupported by vendor, account-terms risk).
- Dollar-based cost meter unless a metered cloud model is used; replace with time/compute budget, idle auto-stop, memory/GPU guards if fully local.
- Military/ship/ALPR/spy-style tool vocabulary and "classified-room" tone; keep neutral civic language.
- Sensor-style visual filters as voice commands (optional).

**Local-voice requirements to design for:** streaming speech-to-text with end-pointing and per-utterance Arabic/English detection incl. code-switching and Gulf, Levantine, Egyptian, Modern Standard dialects; right-to-left captions; function-calling model with strict schemas, multi-call turns and image input (or a separate vision model); streaming speech output with sentence chunking, instant cancel, truncation tracking and voices in both languages; echo cancellation or half-duplex fallback; progress hooks; short rolling context + live scene injection.

## Open questions / UNVERIFIED
- Exact eagerness and silence thresholds inside the cloud voice-activity detector are provider-side (only "semantic, low eagerness" is known).
- Whether transit/bike-share/analyst field lists are complete for non-US feeds - listed from the voice manifest only.
- Dwell/threshold numbers for pointer picking are from constants and tests; real-world feel (hit-testing tolerance) not observed live. UNVERIFIED.
- The 5-second browser abort for the HUD summary and the 15-second cadence are as documented; behaviour when the tab is hidden is UNVERIFIED.
- Exact wording of the model-facing instruction prompt is intentionally not reproduced; behaviour has been inferred from its rules (see sections 2, 4 and 5).
- Resolved this pass: voice query tools = 25, actions = 30, MCP = 30 (re-counted from source); these counts drift as tools are added.
- Whether the reticle's two looks (entity vs ground) differ in colour or only in shape - UNVERIFIED (only that they differ).
- No live voice session was run (static analysis only).

## Revision log
- Re-counted tools from source: 30 voice actions + 25 voice world-query tools = 55; MCP lists exactly 30 (one hidden from the model); recorded the stale "28" figure as a doc-versus-code conflict.
- Added the full MCP composition (16 shared with voice, 8 area lookups, 4 voice-excluded items, show-in-globe, panel relay).
- Point-and-ask: added freshness rules, edge margin, reticle lifecycle (camera-move hide, 1.5 s linger, 10 s maximum), THIS/HERE chips, and the exact radius formula (0.75 x camera height, 5-400 km).
- Voice cost: re-verified both rate cards, transcription rates, 2 USD warning and 5 USD cap; added price ratios and the cached-transcription pricing rule.
- Resolved the analyst list-length question (voice 12 when omitted, engine 10, max 50) and added the entity-context default of 5.
- Added public-camera catalogue pack rules (15 packs, per-pack caps, 4,000 ceiling, round-robin fill, partial-serving honesty) and voice card display limits; added confidence tag legend.
- Pass 3 (census merge): added tool-name namespace and action precedence, catalogue composition and lazy loading (4.6), single layer manifest (4.7), named-view shorthand, annotation steering hints, tracked-subject slot invariants, voice-card fade and adapter, queue/follow-up and deadlock ordering rules, cost-tracker swap rule (7.5), counting-number divergence and centre precedence, deterministic summary rules, MCP panel relay and key-sharing details, view-object limits, result hygiene, voice-routing benchmark with speech lint (11), retired/planned items (12); corrected the preset-city statement (eight voice presets, Tallinn tray-only); added Kawn notes tied to owner decisions. Confidence for census-derived text: source-level, no live run.

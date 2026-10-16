# Product and UX behavior study (clean-room, Wall A)

Subject: a browser-based "live Earth" explorer that fuses public real-time signals onto a photorealistic 3D globe. This document records observable product behavior only. Detail lives in reverse-engineering docs 01, 02, 05, 06, 07.

## 1. Who the users are

| Persona | What they want | How the product serves them |
|---|---|---|
| Curious explorer / enthusiast | Awe, a "forbidden control room" feeling, shareable moments | Cinematic looks, ride-along views, guided starter choices |
| OSINT hobbyist / analyst | Correlate aircraft, ships, satellites, fires, quakes, cameras in one place | Layers, click-to-track, rosters of nearby objects, source honesty labels |
| Content creator | Screen-record striking camera tours | Scripted camera tours, sensor "looks", a clean view, shareable links |
| Developer / tinkerer | Extend with new data sources | Modular layers, user-supplied keys, inspectable code |
| Educator | Explain space, weather, disasters | Launch replays, weather timeline, a narrated disaster replay scene |

No accounts, no teams, no roles. It is a single-operator, local-first tool; sharing is by link or file.

## 2. Product promise and tone

A "spy-satellite simulator" aesthetic (military HUD, thermal and night-vision looks) paired with a strict honesty message: every source is public, estimated items are explicitly labeled as estimates, incomplete community data is called incomplete, and stale or fallback feeds are admitted in answers. Trust is part of the brand.

## 3. Screen anatomy

One full-viewport 3D globe with floating chrome layered on top. No separate pages. Detail: doc 01 sections 1-6.

| Region | Contents | Behavior |
|---|---|---|
| Center | Globe / 3D city | Drag to orbit, scroll or pinch to zoom, click an object to select it |
| Top-left | Brand block (logo, name, tagline); optional frame-rate line | Words disappear on narrow widths, logo stays |
| Top-center | Row of five icon buttons: clear layers, copy share link, tilt, north-up, reset to full globe; loading and sync chips beneath | Reset flies to a whole-Earth view (about 2.8 s) and also exits tracking, orbit and cockpit; tilt flips between near-straight-down and an oblique 35 degree view |
| Left rail | Data Layers panel (visible but collapsed to a compact button) and Scenes panel (collapsed) | Panels collapse/expand; clicking a collapsed body expands it; headers stay visible while bodies scroll |
| Right rail | Display, camera feeds, Street Level, Weather, Recent Imagery, Context (selection telemetry, contacts, radio) | Display starts collapsed on a fresh profile. Camera feeds and Context are mutually exclusive; Weather and Imagery appear only while their layer is on. Overflowing panels scroll inside; lower-priority panels auto-collapse (never saved) |
| Bottom dock | Left wing: LOCATION tray (city pills, five landmark pills per city, search behind a magnifier); centre: voice capsule; right wing: VISUAL PRESETS tray (seven look tiles, map-source chips) | Both wings start compact. Mouse hover opens after about 140 ms and closes about 420 ms after leaving; a pin keeps up to two trays open; touch has no hover, so tap is the only way |
| Bottom-left | Mandatory credits button opening an attribution lightbox | Always present and keyboard reachable, including in clean view and recording |
| Bottom-right | "Power up" setup chip | Exists only on a local/dev server with a settings service; removed on hosted builds; retires once every key is set |
| Overlay | HUD text, detection boxes and cards, caption lines | HUD and detection are on by default; see clean view caveat in section 4 |
| Modal | First-run launcher | Appears after the loading screen fades (section 5) |

Start-up: a loading screen (at least one second), then an opening flight to a default city (Austin in the reference), or the shared-view restore when a link is present.

## 4. Modes

1. **Globe / free exploration** - default.
2. **Tracking mode** - clicking an aircraft, vessel, satellite, fire etc. locks the camera, draws a fading trail, opens a telemetry card. A click counts only if the pointer travelled at most 6 px; clicking empty space deselects only for a clean click (at most 6 px and 400 ms). Deselecting never flies the camera. A tracked fire or vessel offers a one-click jump to the nearest live public camera. Only flights and satellites can travel in a share link, as a "handoff", not a bookmark.
3. **Cockpit mode** - rides a tracked aircraft; the camera holds terrain beneath it. Entry is gated: Contacts mode must be active, both civil and military flight layers on, and an aircraft tracked; otherwise it is refused with a reason. Panels collapse on entry and are restored exactly on exit. Exit by Escape, C, the exit button, or reset. A briefing strip (live signals, regional headlines, local weather) and an optional weather-rendering toggle (off by default) are included. On narrow windows (about 760 px or less) the map rails and HUD text hide and the controls move to bottom pills. Contacts next/previous hop aircraft to aircraft.
4. **Contacts mode** - roster of everything within 250 km of the subject (tracked aircraft, selected vessel or mapped site); three nearest entries per cohort page; PREVIOUS / FOCUS / NEXT step through visited history, then the nearest unvisited contact, expanding the search radius when the window is exhausted. Class filters (for example helicopters only) arrive only by voice or tools, not buttons. A feed that is down shows "unknown", never zero. Entering forces detection on; leaving restores the prior layers and detection.
5. **Global Context mode** - one switch stages the whole situational picture (layers, framing). Leaving restores the exact prior view. Variants exist for live traffic and space missions.
6. **Sensor looks** - seven whole-scene looks (normal, retro CRT, night vision, thermal, anime-style, noir, snow), selectable by number keys 1-7. Applies everywhere including cockpit. Picking CRT, night vision or thermal also switches HUD to Tactical and detection to dense unless the user hand-edited detection, and opens a Parameters sub-panel.
7. **Detection overlay** - corner brackets and a single-line label (identifier plus one dim metric) on visible objects. On by default, dense, in every look. A Detect toggle (key D) switches it off or back to the last density; a density slider has five stops (sparse / balanced / dense), plus allocation, fade and outside-opacity controls. It does not "cycle".
8. **Clean view** - toggled by the Clean UI button or V; hides the rails, dock, top row, chips and setup chip, leaving a small exit pill at top centre. It does NOT hide the HUD, detection overlay, scope, annotations, the voice capsule or the credits. Escape does NOT leave it; use V or the pill. Not saved in links. Embedding mode forces it on and hides everything but globe and credits.
9. **Orbit mode** - slow circling (about 6 degrees per second, a minute per revolution) of the last landmark flown to. Key O only; no on-screen control; with no landmark it says "fly to a POI first". Stops when another city is chosen or on reset.
10. **Camera feeds mode** - public cameras projected into the 3D city; coverage cycles off / on / viewshed. Turning the layer on activates the nearest camera but flies only if nothing is being tracked or cockpit-ridden.
11. **Directions mode** - click A then B; a street-following route (drive, walk, cycle) drapes on terrain with a step list; Fly sends the camera along it.
12. **Scene director (tours)** - author, play, scrub, import, export multi-shot tours. During playback every control but Stop is disabled and Escape stops.
13. **Voice mode** - hands-free conversation with a realtime assistant (needs a key).
14. **Annotation / whiteboard (Draw)** - area, line, pin with label and colour; finish by double-click or Enter, Backspace removes the last point, Escape cancels then leaves; a 120-mark live cap applies.
15. **Recording presentation** - set by scene playback, not a button: hides chrome including the voice capsule, forces a minimal HUD and draws a 16:9 or 9:16 safe frame. There is no built-in capture.

## 5. First-run flow

1. The app starts keyless: satellite imagery and open terrain load, free feeds are available. All layers are off on a new profile.
2. After the loading screen fades a compact launcher card offers four paths: live aircraft contacts, space missions, an environmental (fires and quakes) view, or explore manually.
3. Choosing a path acts exactly like the user clicking the equivalent controls: it enables that path's layers (saved as real preferences), opens Context only for the two context paths, and leaves look, detection, scope and 3D untouched.
4. While launching, tiles go busy-but-focusable and a status line shows progress. On failure the card stays open with a retry-or-explore-manually message.
5. Return policy: the card returns every new browser session until the user ticks "do not show again" (the only durable dismissal). Choosing a path, choosing manual, or Escape dismiss it for the tab session only; blocked storage reverts the checkbox with an explanation.
6. It never appears for a share-link arrival or an embed; support switches force it on or off. It yields to cockpit, scene playback, recording and clean view, and waits if one is already active.
7. Focus is trapped inside while open and returns afterward.
8. Optional upgrade chip (local/dev only): lists each optional key, what it unlocks, where to get it; saving restarts the app. Externally managed keys are read-only. A URL switch reopens it when hidden in compact layouts.

## 6. Core interactions

| Intent | What the user does | What they see |
|---|---|---|
| Find a place | Type a name or coordinates, press Enter | Instant flight for coordinates and bundled place names; remote geocoders otherwise; toast on not-found or failure; late results from superseded searches are ignored |
| See live aircraft | Turn on Flights layer | Thousands of glyphs; swap to 3D models as the camera closes in (nearest only, or all, by setting) |
| Inspect an object | Click it | Camera locks, trail draws, telemetry card opens |
| Ride along | Cockpit control or C (when allowed) | Camera attaches; briefing strip appears |
| Hop between aircraft | Contacts roster | Next cockpit loads |
| Change the look | Press 1-7 or a look tile | Whole planet re-renders instantly (crossfade) |
| Check satellites | Enable Satellites, click one | Camera rides at orbital distance with orbit ring |
| Replay a launch | Choose a recent launch (past month) | Countdown through ascent; scrub at quarter to 4x; labeled as a reconstructed estimate |
| Weather | Open the weather panel | One shared history timeline with step back, play, latest; separate wind, cyclone layers |
| Radio | Enable radio, drag the analog tuner needle | Needle snaps to real stations; globe flies to the broadcaster |
| Measure | Ask by voice, or draw a line/area | Arrow with distance caption; endpoints stay pinned while orbiting |
| Reset | Reset button or say so | Back to full Earth |
| Share | Copy link (the address bar is already kept current, 500 ms debounce) | Camera, look, layers with options, panel states, and at most one tracked flight or satellite are encoded; clean view, orbit, cockpit, voice, radio and drawn marks are not |
| Open a shared link | Load the link | "Restoring shared view"; flight starts after about 1.5 s and takes about 3 s; any pointer press or wheel from the recipient cancels the camera move and follow; a tracked subject is retried for up to 90 s (civil flight), 45 s (military), 300 s (satellite) |

Layers are grouped as Movement, Cameras, Infrastructure, Events, Weather, Utilities; each row shows count, state button and a source-and-age line.

## 7. Keyboard and accessibility behavior

Detail: doc 02 Part A.

| Key | Effect |
|---|---|
| 1-7 | Select sensor look |
| H | Toggle HUD |
| O | Toggle orbit (needs a landmark) |
| V | Toggle clean view |
| F | Show/hide the whole Data Layers panel (visibility, not collapse; no on-screen button; not saved) |
| D | Toggle detection overlay off / back on (marks detection as hand-edited so looks 2-4 stop auto-enabling it) |
| C | Three meanings, in order: exit cockpit if in cockpit; else, with a civil or military aircraft tracked, try to enter cockpit (silently swallowed when the entry gate is closed, so nothing else happens); else toggle the camera-feed layer |
| Backtick | Frame-rate readout; the only page shortcut that ignores modifiers and key repeat |
| Q W E R T | Fly to landmark 1-5 while a city's landmark row is expanded |
| Space | Tap activates the focused control; hold 500 ms is push-to-talk |
| Escape | Back out one level (below) |

**Rules and defects.** Page shortcuts are ignored while typing in text boxes or selects (protection is by element type; a focused plain button does not protect). Escape alone still acts in the search field. Overlay surfaces (dialogs, cockpit, push-to-talk, credits lightbox) get first refusal. Page shortcuts do NOT ignore Ctrl/Cmd or key repeat (a test pins this): Cmd/Ctrl plus a digit changes the look as well as the browser tab, Cmd/Ctrl+C with an aircraft tracked toggles the camera-feed layer, and holding H, V, F, D or C flickers the toggle. Keys follow the typed character, so non-Latin layouts fail. Kawn should not copy any of this. There is no shortcut help screen; O, V, F, backtick and Space-hold are discoverable only through the README or trial.

**Escape, three tiers.** Tier 1, page-level capture surfaces act first and consume the key: first-run card, settings dialog, expanded Street Level viewer, cockpit (closes its pop-overs and rails stepwise, then exits), Draw (cancel shape, then leave), imagery box tool, cockpit/dock pop-overs. Tier 2, element-level, only when focus is inside: credits lightbox, inline edit fields, then the nearest expanded panel (collapses and returns focus to its toggle; this never also deselects a tracked object). Tier 3, page-level independent listeners, reached when focus is on the page or globe: clear search, stop scene playback, disarm Directions or clear its step, clear a Street Level sequence, deselect the tracked flight, satellite, vessel, transit vehicle or bike-share station. All tier-3 handlers can fire on one press. Escape never leaves clean view.

**Accessibility.** Native buttons with accessible names; collapse buttons expose expanded state; toggles expose pressed state; async buttons stay focusable while busy; truly unavailable controls are really disabled; visible focus rings everywhere; polite status regions for loading, toasts and errors; dialogs trap Tab and return focus; a visually hidden "visible map targets" list mirrors clickable canvas labels. Reduced motion is honored for the radio dial, wind animation, sonar sweep, panel transitions and chip animation. Weak spots: no description of the globe itself, icon-only top buttons rely on tooltips.

## 8. States

**Loading.** A global strip (top centre) appears only after 160 ms so quick loads never flash, lingers about 2.2 s on success and 5 s on failure, and after 30 s of loading switches to a long-wait message. Traffic and camera sync chips show a percentage or loaded/total count. Toasts are one line, shown 2 s. Known gap: a spinner, not true multi-phase progress.

**Layer states.** Each row's button reads OFF, ON, LOADING, DEGRADED, STALE, PARTIAL, FALLBACK or UNAVAILABLE. A keyless layer cannot be switched on and its tooltip names the missing key. Repeat clicks during a change are ignored.

**Empty.** Some layers report guidance ("zoom in", "nothing here", "idle") shown neutral, never as faults. Viewport-bound community data (military sites, license-plate cameras) is empty until zoomed to a city and is labeled incomplete.

**Error / degraded.** Statuses distinguish feed down, key required, degraded/fallback source, stale data. Failed mission launch stays open with retry. Server errors are sanitized. Map imagery falls back automatically if the preferred provider is unreachable (credits follow the source shown); disabled map-source chips carry the reason. Features needing keys say what is missing rather than hiding.

**Disclosure of uncertainty.** Estimated camera positions, reconstructed launches, simulated traffic along real roads, and incomplete mapped installations are labeled in context.

**Persistence.** Durable in browser storage: enabled layers and options, panel open state, floating window geometry, detection allocation, camera-calibration saves, scene library, voice tier and spend limits. Look, bloom, sharpen, HUD, detection mode, scope and map source live only in the address, so a bare URL returns defaults. Clean view, orbit, frame-rate readout and drawn marks are never stored. A share link overrides stored layers for that load without overwriting them.

## 9. Voice interaction

A dock-centre capsule with mic button, status, model tier and running cost estimate (soft warning near $2, hard cap near $5). Click the mic for open-mic; hold Space 500 ms for push-to-talk (a short tap keeps normal control activation). The assistant knows the current scene (coordinates, street names, active layers, zoom, selected entity). It can fly the camera, toggle layers, switch looks, track and enter cockpit, draw annotations, measure, play tours, answer analytic questions (counts, rankings, next satellite pass), and describe legible signage at street level without inventing labels. It reports success only for actions that succeeded. A terse AI summary of the view regenerates as it changes (HUD text). Detail: doc 08.

## 10. Director (tour) experience

Users capture camera shots into scenes (six built-in scenes ship), set duration and holds, attach data packs and clickable authored explanation cards, and play them with scrubbing. Import opens a preview first (scene list, attributions, missing layers, size) before confirm; cancel or Escape discards; invalid files never damage the current project. A bundled flood replay shows before/after imagery with non-commercial data notices.

## 11. Feedback patterns

Immediate re-rendering for looks; fading trails; caption lines for measurements; status lines inside cards; delayed spinners; state-tinted layer buttons; labeled estimates; confirmation only after real success; sanitized error strings; session versus durable preference distinction.

## 12. Gaps observed (mobile and inclusion)

Desktop-first. At 720 px and below the layout becomes two edge-to-edge scroll bands (layers above the midline, right-rail panels below) with the top row moved to the top-right; no bottom sheets, drawers, swipe gestures or long-press actions exist. Hover-open trays and header-drag of floating windows are mouse/pen only; touch uses taps and rail scrolling. The globe relies on stock one-finger drag and two-finger pinch/rotate/tilt. The viewport declaration likely zeroes safe-area insets on notched phones. Features reachable only by keyboard (touch parity gaps): F panel hide, O orbit, frame-rate readout, hold-Space push-to-talk (the mic gives open-mic), Escape (panels and scene Stop still have buttons). Tooltips have no touch equivalent. Also: no right-to-left layout, no localization, no onboarding beyond one launcher card, no persona-specific entry, no screen-reader description of globe contents beyond the target list, English Latin-script labels only. Real-device behavior is unverified.

## 13. Design language (observed)

Dark "mission control": near-black space, frosted glass panels with hairline borders, thin monospace uppercase micro-labels with wide letter-spacing, one dominant cyan accent, semantic colour tiers per object class (civil cyan, military amber, sea teal, space violet; traffic red / amber / green / grey), HUD colour shifted per look. Transitions about 150 ms (hover) and 300 ms (layout); camera flights 1.4-4.2 s eased; typed-in summary text; a global two-pixel focus outline. Fonts load from a public service (self-host in Kawn). Detail: doc 05 section 9.

---

## Ideas to do differently (Dubai / UAE city-intelligence)

1. **Persona-first launcher.** Replace four generic missions with three role entries: Investor (districts, off-plan projects, yields, transaction heat), Visitor (attractions, transit, events, comfort/heat), Government (service demand, permits, incident and traffic ops). Each lands on a pre-staged map with plain-language captions, and the choice is remembered per profile. Open on Dubai, not a foreign default city.
2. **Arabic-first, true RTL.** Mirror rails, dock, and timeline direction (Data Layers on the right, context on the left, chevrons and slider fills flip); Western versus Eastern numerals; bilingual place search that understands both transliterations of a district; bidirectional-safe text for mixed names and codes; font tuned for dense data. Location tray: Dubai, Abu Dhabi, Sharjah with five curated landmarks each (the reference's Dubai set is Burj Khalifa, Burj Al Arab, Palm Jumeirah, Dubai Frame, Museum of the Future).
3. **Mobile-first bottom sheets.** Replace the two scroll bands with a draggable bottom sheet (peek, half, full) and thumb-reachable action bar; trays become sheets, not hover popups; one gesture set (pinch, two-finger tilt, long-press to inspect); 44 px targets; offline-tolerant for tourists on roaming data; honor safe-area insets.
4. **Time-of-day and heat awareness.** A single "now / evening / summer afternoon" slider that changes shade, walkability, crowding, and prayer-time-aware opening hours.
5. **Explain-everything cards with provenance.** Each metric carries publisher, date, confidence, and official / modeled / crowdsourced tag (the reference's source-and-age meta line, extended); government users get a downloadable audit trail, investors a caveat banner for off-plan projections.
6. **Guided stories instead of raw layers.** Prebuilt narrated tours ("Palm to Creek in 5 minutes") in Arabic and English, shareable as a link that opens in the exact scene; every action reachable without a keyboard.
7. **Compare and shortlist.** Pin up to four places side by side with the same metrics; export a one-page brief (PDF/WhatsApp share).
8. **Accessible by default.** Screen-reader summaries of the visible map ("12 projects in view, 3 ready"), reduced-motion and high-contrast modes, a text-only list of anything on the map, colour-blind-safe status chips with icons.
9. **Regulatory-safe presentation.** Sensitive-site masking, a visible data-use notice, role-gated layers (public, licensed investor, government) with clear "why you cannot see this" messaging.
10. **Calm loading.** Skeleton district shapes and one combined progress line instead of per-layer spinners (keep the 160 ms delay rule); stale-data timestamps in both languages.
11. **Disciplined shortcuts.** One key, one meaning (split C); ignore Ctrl/Cmd chords and key repeat; physical-key codes for Arabic layouts; an in-app shortcut sheet; Escape as one ordered chain closing one thing per press; a touch control for every keyboard-only action.
12. **Persist preferences per profile.** Look, HUD and detection choices should survive a bare URL, unlike the reference where they live only in the address.

## Spec revision log

- Display panel starts collapsed on a fresh profile; Data Layers ships visible but collapsed; fixed rail contents (location and presets are dock trays, search lives in the location tray) per doc 01.
- Keyboard table rewritten per doc 02: F hides the whole Data Layers panel; D toggles detection (no density cycling); C has three meanings with a silent swallow case; page shortcuts ignore neither Ctrl/Cmd nor repeat (defect); Q-T landmark keys and Space-hold added.
- Escape rewritten as three tiers (docs 02, 06); Escape does not exit clean view; clean view keeps HUD, detection, voice capsule and credits (docs 01, 05).
- Modes corrected per docs 06, 07: cockpit entry gate, Contacts 250 km roster and stepping, orbit is key-only and landmark-bound, tracking click thresholds, detection defaults on and dense in every look, single-line labels.
- Added numeric loading, toast and tray timings, share-link restore timings and tracked-subject windows, persistence summary (docs 01, 02).
- Rewrote mobile and touch gaps per doc 01 section 11: 720 px two-band layout, mouse-only hover and drag, keyboard-only touch gaps, safe-area finding; added a short design-language section from doc 05.
- Added Kawn ideas 11 and 12 on shortcut discipline and per-profile persistence; dropped the "unpromised fifth idea" line (not in verified docs).

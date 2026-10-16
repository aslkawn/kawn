# 01 - UI controls and panels (behavioural reverse-engineering)

**Purpose.** This note records every visible control, panel, tray, dock, card, dialog and first-run flow of the reference product (a browser 3D-globe "live Earth" explorer), described only as behaviour: what the user sees, what each control does, defaults, ranges, timings, collapsed/expanded rules, responsive and embedded behaviour, and accessibility behaviour. It is written so a clean-room team can rebuild an original product without reading the source. Wording is paraphrased; numbers (defaults, ranges, delays, limits) are kept because they are behavioural facts. Anything not confirmed in the shipped behaviour is marked UNVERIFIED.

Census pointers: navigation, search and camera mechanics are in 13a; voice, HUD, first-run, settings and platform in 13f. Where this document says "Detail: 13a ..." or "13f ...", the census holds the full mechanism.

Conventions: "rail" = a vertical stack of panels along a screen edge; "tray" = a popup that rises out of the bottom dock; "chip" = a small toggle/selector button; "row" = one layer line in the Data Layers panel. Default states are the *first-run, no-share-link, no-saved-preferences* state unless stated.

---

## 1. Screen anatomy

The whole product is one full-viewport globe with floating chrome above it. There are no routes or pages. Stacking order from bottom to top: globe, world-anchored label canvas, HUD text, rails/panels (promotable, see 3.6), bottom dock, voice capsule, first-run card and settings dialog, toast, clean-view exit button, loading screen at start.

| Region | Contents | Default state |
|---|---|---|
| Top-left | Brand block: logo mark, product name, tagline "NO PLACE LEFT BEHIND"; an optional frame-rate line appears under it when toggled | Visible; at narrow widths the words disappear and only the logo remains |
| Top-centre | Row of five icon buttons (section 2.3) | Visible |
| Top-centre (under it) | Global loading status strip; traffic and camera sync chips | Hidden until work is in progress |
| Top-right-ish | "Active style" label (e.g. NORMAL) | Visible on desktop, hidden at 720 px and below |
| Left rail | Data Layers panel, Scenes panel (stacked, measured against HUD) | Both present; Data Layers visible-collapsed, Scenes collapsed |
| Right rail | Display, CCTV, Street Level, Weather, Recent Imagery, Context (in that order) | **Display starts collapsed** on a brand-new profile [OBSERVED-IN-SOURCE]: the start-up code deliberately forces it closed when nothing is stored, with a comment explaining that it used to open to advertise the HUD/Detect/3D toggles but that reason expired once they became on by default, and that it would compete with the first-run card. A stored choice wins in both directions. Older project notes that say "starts expanded" are out of date (see Open questions: resolved). CCTV/Street Level/Context collapsed; Weather and Recent Imagery hidden until their layers are on, and when first revealed they are not pre-collapsed [INFERRED from markup] |
| Bottom dock | Left wing: LOCATION tray launcher; centre: voice capsule; right wing: VISUAL PRESETS tray launcher | Both wings compact (collapsed) on every load |
| HUD overlay | Four-corner military-style telemetry text plus top/bottom bars (section 9) | ON, Tactical layout (documented default) |
| Bottom-left | Mandatory data/imagery attribution button (opens a credits lightbox) | Always present, keyboard reachable |
| Centre overlays | Safe-frame guide (recording only), orbit indicator, cockpit view | Hidden |
| Modal-like | First-run launcher card, provider-settings dialog (section 7) | Launcher appears after start; settings dialog closed |
| Loading screen | Logo, product name, status line that starts as "Initializing photorealistic world..." and is later overwritten by stage messages, including "Flying to Austin, TX..." when there is no share link and "Restoring shared view..." when there is one [OBSERVED-IN-SOURCE] | Shown until the initial restore finishes and at least 1 s has passed, then fades out |

Without a share link the camera performs an opening flight to Austin, Texas [OBSERVED-IN-SOURCE]; with a share link it instead performs the restore flight described in file 02. The opening flight is a two-step move: the camera is first placed 25 km straight above the default city, waits half a second, then flies for 4 s down to a 600 m oblique view (heading 15 degrees, pitch 30 degrees below the horizon) [OBSERVED-IN-SOURCE]. The pending flight is cancelled cleanly if the page is torn down mid-way. Mechanism detail: 13a section 2.4 (startup camera).

Startup sequence: loading screen -> (minimum 1 s and the initial share/stored-state restore settles) -> loading screen fades -> after the fade ends (or 900 ms fallback if the transition event never fires) the first-run launcher may reveal. The provider-settings chip is initialised in parallel and removes itself entirely if the settings service is unreachable. The loading line steps through: configuring the viewer, loading the photoreal 3D tiles (or the keyless globe, or an "unavailable" notice that names the reason), initialising systems, then flying to the default city or restoring the shared view. If every 3D-tile route fails, the plain globe is shown with satellite imagery and the loader line names the last error [OBSERVED-IN-SOURCE; confidence high]. Detail: 13a NAV-43, NAV-68.

**Bare viewer.** The globe engine is built with all of its stock widgets switched off (timeline, animation clock, base-layer picker, search box, home button, scene-mode picker, help, fullscreen, VR button, selection indicator and info box); every visible control in this document is the product's own. The engine's attribution output is redirected into the product's own credit container. Four-sample multisampling, a preserved drawing buffer (so screenshots and recording can read the canvas) and a 60 fps target are set, and the sky atmosphere is on from the first frame [OBSERVED-IN-SOURCE; confidence high]. Detail: 13a NAV-56.

**Idle behaviour.** The viewer renders every frame only while at least one animation holds a "keep drawing" claim (aircraft fleets, traffic, satellites, tracking, look crossfades, orbit, camera verbs, camera projection, cockpit); with no claims it renders only on camera input, tile loads or an explicit one-frame request from a discrete UI change, so a parked scene costs almost no GPU. Detection and Scope take no claim. Hidden tabs skip celestial and weather work, show a dash in the frame-rate readout and repaint once on return. Detail: 13a NAV-57 to NAV-59 [confidence high].

**Apple GPU workaround.** On iPhone, iPad and any GPU whose driver cannot link the per-vertex atmosphere shader (found by a throw-away probe context, with platform detection as backstop), that atmosphere stage is switched off before any tile draws while the fog density that scales tile detail stays on. Cost: no distance fog on 3D tiles and on the plain-globe basemaps for those devices. Relevant to Kawn only if iPad or mobile Safari is in scope. Detail: 13a NAV-60, 13f SYS-172 [confidence high].

**Kawn note (start-up).** Kawn opens on Dubai, top-down at city scale, with Scope ON and feather at its maximum (owner decisions of 2026-10-16), not on Austin with feather 11 %. Reuse the two-step opening flight idea (high, then glide down) with Dubai as the target.

---

## 2. Global chrome

### 2.1 Brand block and style indicator
- Brand block is decorative. The logo tracks the pointer ("gaze") but ignores touch input and is skipped under reduced-motion [OBSERVED-IN-SOURCE]; whether the loading-screen logo shares the effect is still UNVERIFIED (markup suggests yes).
- "Active style" label mirrors the current look name (NORMAL, CRT, NVG, FLIR, ANIME, NOIR, SNOW). Hidden at <= 720 px.

### 2.2 Frame-rate readout
Appears under the brand when the backtick key is pressed; shows "FPS n" refreshed once per second, counted from real rendered frames; shows a dash while the tab is hidden. Starts hidden every session; never persisted. Typing in fields, modifier keys, key-repeat and composition do not toggle it. On narrow screens it is pinned just below the brand.

### 2.3 Top-centre action row (five buttons)
| Button | Effect | Notes |
|---|---|---|
| Clear selected layers | Turns off every currently enabled data layer in one transaction | Stays focusable while working (announces busy); label changes to "Clearing..." while busy; also releases any dependency-held layers |
| Copy share link | Copies the current address (with a creation timestamp added to the copy only) to the clipboard | Toast on success/failure; the live address bar is already auto-updated (see file 02) |
| Tilt map | Toggles between a near-straight-down view (about 89 degrees below the horizon) and an oblique view looking 35 degrees below the horizon; any pitch shallower than 60 degrees below the horizon counts as already oblique, so the next press goes straight down [OBSERVED-IN-SOURCE] | Has a pressed state; vignette/scope remains |
| North up | Rotates camera heading to north | Shows a compass "N" with a needle that rotates with the heading |
| Reset to full globe | Cancels tracking/orbit/cockpit and flies out to a whole-Earth view (18,000 km altitude, looking straight down, 2.8 s flight) | Accessible name changes to "Resetting..." while the flight runs; completes or gives up after ~4.2 s; also stops orbit, exits cockpit and releases followed flights, vessels, satellites and launches; same action exists inside cockpit as RESET |

At <= 720 px the row moves to the top-right corner. In the Cyber HUD theme the provider-settings chip is relocated into this row (UNVERIFIED visual order).

**Behaviour details (census merge).**
- *Tilt.* The view orbits about a target chosen in this order: the tracked entity's displayed position, else the surface under screen centre, else the terrain under screen centre, else the ellipsoid. The move is a 650 ms animation to either 35 or 89 degrees below the horizon; quick repeated clicks continue toward the remembered destination. The pressed state is refreshed once at the end of each camera move (a depth read costs about 4 ms, occasionally 12 ms) rather than every frame. Accessible name flips between "Tilt map to oblique view" and "Return map to straight-down view" [OBSERVED-IN-SOURCE; confidence high].
- *North up.* Heading eases to zero about the same target, keeping pitch. The needle is redrawn every rendered frame (the engine's camera-changed event is too coarse); when pitch is within 1.5 degrees of straight down the heading is read from the camera itself because the target-frame heading is undefined there. The button exposes a north-up state and a label that names the current bearing [confidence high].
- *Reset.* Flies to 18,000 km straight down, north up, keeping the point under the camera; concurrent clicks share one pending result; the name reads "Resetting to full globe view" during the flight. The 18,000 km figure must stay inside the band that downstream policies treat as "global".
- *Zoom limits.* The product sets no global min or max zoom distance (the engine default applies). While something is tracked the minimum camera-to-target distance is raised to 150 m and zoom inertia is off; programmatic ranges are bounded 100 m to 20,000 km.
- *There is no keyboard zoom or pan.* Arrow keys and plus/minus act only inside the street-level photo viewer; see file 02. Voice offers relative zoom (13a NAV-05).
Detail: 13a NAV-32 to NAV-34, NAV-70 [confidence high].

### 2.4 Loading and status feedback
- **Global loading strip** (top-centre, polite live region): label "LOADING LIVE DATA" (or "REFRESHING LIVE DATA" for pure refreshes, "TURNING OFF LIVE DATA" while disabling) plus up to two layer names and "+N" for more. A strip does not appear until 160 ms after work starts (so quick loads never flash). After success it lingers 2.2 s; after failure 5 s. After 30 s of continuous loading it switches to a "long" visual state. Special messages exist for retrying mapped-site and ALPR-camera fetches. Persistent notices (for example "ACQUIRING" a shared aircraft/satellite) stay until resolved; finite error notices queue and each gets its full dwell once it reaches the screen.
- **Traffic sync chip**: "syncing road network" + percentage; confirmation lingers 1.5 s after completion; also shown during world jumps between cities.
- **Camera sync chip**: "loading frames" + loaded/total count (for example 612/3902), fills progressively as the camera catalogue streams in.
- **Toast** (single line, polite): fixed 2 s display, then fades; replaced immediately by the next one. Observed messages: "Fly to a POI first" (orbit with nothing to orbit), "Location not found", "Search failed", "Detailed outline unavailable", "CCTV layer unavailable", plus one-time layout hints.
- Chips hidden by clean view/recording, or not yet visible, swap instantly with no animation; reduced-motion also removes animation.
- **Verdict wording.** Finished loads read LOAD COMPLETE, LOAD CANCELLED or LOAD FAILED; other notices read LIVE DATA OFF and MAPPED SITES LOADED. Retry notices show a live countdown. The strip aggregates every layer's lifecycle without owning it; "complete" is a timing signal, not another stage. Finite notices start their dwell only when first shown, so a higher-priority failure cannot use up the time of one that is still queued; persistent notices stay until cleared. The chip label may animate character-by-character like a departure board, but a screen reader hears only the final text and an unchanged text is a no-op. The map-source readout and the style indicator are polite live regions [confidence high]. Detail: 13a NAV-69, 13f SYS-108, SYS-186.
- **Honest feed states.** Each layer's heterogeneous statistics are normalised into one of: nominal, loading, degraded, stale, partial, fallback, unavailable. "Guidance" statuses (zoom in, run a search, nothing here) are normal operation and never shown as faults, but a genuinely stale cache still reads STALE. A vessel source can report healthy empty coverage. A layer whose key is absent stays registered with its toggle disabled and a reason that uses the layer's display name; its share token stays stable [confidence high]. Detail: 13f SYS-187, SYS-188.

### 2.5 Clean view
A "Clean UI" toggle in Display and the V key hide all chrome (rails, dock, HUD, status). A small "EXIT CLEAN VIEW" button (highest layer) is the only control left on screen. Attribution stays visible. Clean view and recording presentation both conceal conforming panels *including outward popups*.

### 2.6 Recording presentation
Used by scene recording/voice, not by a button: hides panels, forces the HUD to Minimal (or off/auto/full per request), and draws a safe-frame rectangle (16:9 default, 9:16 alternative). The user's previous HUD mode and layout are snapshotted once and restored on exit. The first-run card yields while this is active.

### 2.7 Orbit indicator
When orbit is running a small "ORBIT" badge with a rotating arrow icon appears on screen; it is removed when orbit stops.

### 2.8 World-label accessibility list
A visually hidden list "Visible map targets" mirrors actionable labels drawn on the label canvas, with a polite status line. This is the only non-pointer way to reach canvas-drawn labels.

---

## 3. Panel system (how all rails behave)

### 3.1 Panel anatomy
Each panel: soft glow behind it, a header (title, hairline divider, optional status/count, collapse button), and a body. Left-rail panels show "+" when collapsed and "-" when expanded; right-rail panels show a left-pointing triangle when collapsed and a right-pointing one when expanded. Collapse button name and tooltip always read "Expand <panel name>" / "Collapse <panel name>" and expose expanded state.

### 3.2 Disclosure rules
- Clicking a collapsed panel's body area also expands it (not on the toggle itself).
- **Escape** collapses the nearest *expanded* panel that contains keyboard focus - it only acts when focus is actually inside that panel (a press with focus on the globe or page background never collapses a panel), and the press is then consumed so map-selection handlers do not also fire [FROM-TESTS]; it acts before any containing panel does; focus returns to that panel's own toggle (if Escape came from the toggle itself, focus is cleared so no ring lingers). Covers standard panels, the nested Parameters panel, cockpit Contact and Live Signals rails and cockpit Display/Radio pop-overs.
- Explicit user opens "claim" the panel lane: they outrank anything restored from a share link or storage.
- Programmatic opens happen only for explicit successful actions (Contacts / Space Missions / Cockpit actions reveal Context; turning Street Level on or opening a photo opens Street Level) and are not saved as a preference when they are automatic.
- Radio is a nested section inside Context; opening it forces Context open (in Cyber theme).

### 3.3 Right-rail policy (default theme)
- Display may stay open beside one contextual panel. CCTV and Context are mutually exclusive.
- In the Tactical HUD, expanding CCTV or Context hides the other contextual launcher; Display stays available.
- The most recently opened panel owns the constrained vertical lane; passive restore and automatic collapse never replace an explicit owner.
- If an active panel is taller than the corridor left by the HUD and other chrome, the rail scrolls that panel's body internally (header stays fixed).
- Compact buttons are 176 px wide, ~50 px tall; expanded **Display is 272 px wide, expanded Context 330 px**, and the right rail container itself is 330 px wide; the Data Layers panel expands to 280 px; the left rail uses the same 176 px compact width; both rails sit 52 px in from their screen edge; the right rail aligns to the left rail's settled top [OBSERVED-IN-SOURCE]. DOC vs CODE: older project notes say Display and Context are both 330 px, another says Display keeps the compact 176 px; a later note and the code say 272 px.
- The rail "auto-collapses" lower-priority panels when space runs out; this is presentation-only: it is never written to saved or shared state, and keyboard focus on the right rail is honoured (the focused panel is not collapsed out from under the user).

### 3.4 Cyber theme variant
Selecting the Cyber HUD layout skins all panels red/slate with chamfered frames and monospace type, collapses launchers to 200 px, and makes the **right rail one-at-a-time** (opening one explicit panel collapses its peers; floating windows are exempt). Entering Cyber the first time switches the look to FLIR with a 0.42 Ironbow palette setting and turns on the Sonar scanner; switching back restores the look used before Cyber. Popups (voice help/error, dock pins) escape the frames without clipping. A shared surface convention exists for adding new panels (fixed header, bounded scrolling body, popups as siblings of the body).

### 3.5 Floating ("portable") windows
CCTV and Street Level (and legacy Display placement - the notes say Display is no longer draggable; treat as not draggable) can be lifted out of the rail:
- Drag by the header with mouse or pen (touch does not drag - on a phone the header is a scroll swipe). Movement under 4 px is a click, not a drag.
- Resizable from every edge; minimums 300 x 160 (CCTV) and 320 x 280 (Street Level). Kept inside the viewport with a 6 px margin; the header is kept clear of the dock and voice capsule; re-clamped on window resize and when content height changes.
- Double-press the header (two presses within 400 ms and 6 px) snaps it back into the rail. A one-time toast teaches this ("double-click the header to snap back").
- Shrinking Street Level to its strip, collapsing it, or SHRINK after EXPAND docks it back automatically.
- Window geometry is remembered (persistent preference); a layout version bump can reset positions and shows a one-time notice.
- Z-order: panels promote within a 100-139 band on pointer-down (floating windows sit above docked rail panels); voice capsule 150; toast 200; clean-view exit 300.

### 3.6 Persistence of panel state
Collapsed/expanded per panel and floating geometry persist across sessions in browser storage; the *shared link* carries collapsed/pinned state for known panels (file 02). Automatic (layout-driven) collapse is excluded from both.

### 3.7 Dock trays (hover disclosure)
The two bottom wings behave identically: hover with a mouse/pen for **140 ms** to open; leaving closes after **420 ms** unless pinned or the pointer returns; scrolling the wheel over a closed wing cancels a pending open for 280 ms; touch has no hover. Keyboard focus *visible* inside the tray (ring showing) holds it open; a mouse-clicked button inside does not. Click/Enter on the launcher toggles immediately (Enter acts on press, Space on release) and moves focus into the tray once it becomes focusable (retries up to ~0.7 s past 240 ms because of a 180 ms fade). A **pin** icon on each tray keeps it open (pressed state exposed); pinning opens it, unpinning closes it unless the pointer is still over it; up to two trays can be pinned and they stack, newest pinned on top. Pinned state is shareable. Both wings always start compact on load.

---

## 4. Bottom dock

### 4.1 LOCATION tray
- **Collapsed face**: label "LOCATION" with a pin icon, plus two-line mini status: line 1 "location: <city or first segment of searched address>", line 2 "landmark: <POI name>" or the remaining address segments (ellipsised) or "Searched location"; empty reads "--".
- **Expanded**: row of landmark pills (up to five, with key hints Q W E R T), a divider, city pills, and a search field behind a magnifier toggle.
- **City pills** (nine): Austin, San Francisco, New York, Tokyo, London, Paris, Dubai, Washington DC, Tallinn. Clicking flies to the city's lead landmark and expands its five landmark pills; clicking the *active* city again just collapses the landmark row. Each city has exactly five curated landmarks. Dubai's five, in key order Q to T, are Burj Khalifa, Burj Al Arab, Palm Jumeirah, Dubai Frame, Museum of the Future [OBSERVED-IN-SOURCE]. City pill order is Austin, San Francisco, New York, Tokyo, London, Paris, Dubai, Washington DC, Tallinn.
- **Landmark pills** show their key letter and name; clicking or pressing the matching letter flies to that landmark (framing uses a per-landmark distance, pitch and heading). The active landmark is highlighted.
- Moving between cities triggers a "world jump": traffic work and detection pause until arrival or a 5.2 s watchdog. **Scope of the rule (corrected):** it applies only when the user clicks a city or landmark pill while a *different* preset city is already active. First use, a flight from a searched location, typed searches, voice flights and share-link restores do not use it (reset-to-globe begins its own jump), so a long typed-search flight does not suspend labels. Kawn can apply the rule on flight distance instead [confidence high]. Detail: 13a G9.
- **Search**: the magnifier expands the field and focuses it. Enter submits. Order: decimal coordinates and bundled place names resolve instantly with no network; then remote geocoders. Outcomes: found (flies, mini-status shows the address, city highlight cleared, landmark row collapsed), "Location not found" toast, "Search failed" toast, optional "Detailed outline unavailable" toast. Escape clears and collapses the field; Escape on the tray collapses the tray and clears unfinished search. The field gains a "searching" look while a query is in flight, and the box closes and clears when the flight starts or the search settles.
- A typed search while another is running supersedes it; late results from superseded searches are ignored. A stale answer is also dropped if a newer navigation (or a user gesture) happened since submit. A typed search deliberately does *not* release tracking or stop orbit at submit time, because the answer may never arrive; it releases and flies only on return, and only if it is still the newest navigation and Cockpit is off. Until a real arrival, the old "searched location" text stays on screen.
- **Landmark keys are conditional.** Q, W, E, R and T act only while that city's landmark row is open and focus is outside text fields and selectors (see file 02). A preset city always wins over a searched label in the collapsed readout; a one-segment address reads "Searched location".

### 4.1.1 Search and fly-to mechanics (census merge)

Four entry doors reach the camera: a pill in this tray (instant, no network), the typed search, a voice or tool request, and reset-to-globe. Every explicit destination first asks who owns the camera: if Cockpit is active the request is refused with a message to exit Cockpit; otherwise a new "navigation generation" is stamped, pending arrival corrections are cancelled, every follow owner is released (tracked aircraft, satellite, vessel, space-mission camera, Contacts), orbit and camera verbs stop, any flight in progress stops and any look-at lock clears [confidence high]. Detail: 13a section 1.

**Provider chain.** The first provider with a place wins; the whole chain has a 12 s budget:
1. Decimal coordinates (offline). Grammar: exactly two decimal components separated by a comma, semicolon or whitespace; each may carry an optional degree sign and at most one hemisphere letter (N, S, E, W) before or after; a leading sign together with a letter is refused; with no letters latitude comes first, with letters either order is accepted if the letters name different axes; degrees-minutes-seconds, grid references, plus codes, out-of-range values, extra components and trailing text all decline and fall through. The result is flagged "exact", labelled with four decimals and hemisphere letters, and gets a box about 3 km across.
2. Bundled names (offline): the nine cities and 45 landmarks, compared after case, accent and punctuation folding; only whole-name matches count so a bundled name never shadows a real place that merely resembles it.
3. The unqualified Alps (offline): returns the bundled range outline's centre and box; if the outline is unusable it raises an error instead of falling back to a same-named peak or shop.
4. Google geocoding, only if a browser Google key exists; the current view is passed as a bounding-box bias.
5. Photon (public OpenStreetMap geocoder), keyless: five candidates, 6 s timeout; pass one is biased to the view centre point and accepts only a name equal to or starting with the query; pass two is unbiased and prefers a name starting with the query's first comma-part, then one containing it. Place types are inferred from OpenStreetMap tags first (parks, stadiums, universities, airports, malls, zoos, cemeteries, forests, regions; lakes, rivers and peaks as natural features; roads as streets; town squares deliberately stay precise places) and only then from the coarse class. The tag-first rule exists because a lake framed at building range left the camera 26 m above water.
6. The app's own proxy to Nominatim, last resort, one request per second server-side, five-minute cache.
Definitive answers are cached 5 minutes (a clean "nothing found" 30 seconds, 64 entries per view bias); a timeout or network failure is never cached, so an outage is not remembered as "no such place". Detail: 13a NAV-09 to NAV-15 and G2, G3.

**Mis-read rescue.** If the answer is not "exact" and a Places service exists, the app compares it with the point under screen centre (or beneath the camera if only sky is shown). If the geocoder returned nothing or its answer is over 50 km away, a text search biased to a 6 km circle around the view centre is asked, and a hit is trusted only within 8 km of the view centre ("the Capitol" typed over Austin lands on the Texas building, not Washington). Typed coordinates and bundled names are never overridden. Without a Google key this step does nothing (UNVERIFIED whether any keyless substitute exists). Detail: 13a G4.

**Framing mode from place type.**

| Place type | Mode | Result |
|---|---|---|
| country, state/province, county | region overview | fit the whole box |
| city/town | city overview | fit the box, after the sanity gate |
| district, neighbourhood, postal code | neighbourhood close | close landmark framing at 250 m, no building outline lookup |
| street, intersection | street corridor | fit the box, else a 900 m range |
| park, natural feature, campus, university, airport, stadium, amusement park, zoo, cemetery, mall | area overview | fit the box (swath rule if huge), else 1,400 m |
| anything else (building, address, point of interest, coordinate) | precise place | close landmark framing, 250 m default |

An explicit distance (voice) skips box fitting; an explicit "close" forces landmark framing; an explicit "overview" forces box fitting and also disables the sanity gate.

**Flight recipes.**
- *Box fit.* Pad the box by 12 % of its height and width, at least 0.05 degrees each way, latitude clamped to 89.9; width is measured the short way round the planet so date-line boxes (Alaska, Fiji) are not read as 340 degrees; a padded width of 360 degrees or more becomes the full range. The engine picks the altitude so the rectangle fills the view, looking straight down, north up; 3 s for UI searches, 2.2 s for voice, cubic ease.
- *Sanity gate (cities and regions only; countries and explicit overviews exempt).* If the box diagonal exceeds 300 km and the geocoder's own anchor lies more than 15 % of the diagonal from the box centre, the box is replaced by a 40 km square around the anchor. Example: "Tokyo" is a prefecture including islands 1,000 km out to sea; without the gate the camera frames open Pacific. Measured ratios: Tokyo 0.40 and Hawaii 0.46 are gated; Nunavut 0.10, Alaska 0.09, Texas 0.05, California 0.04 are not. Countries with overseas territories (France 0.39, Portugal 0.38) show the same flaw and were left as is.
- *Natural-region swath.* A natural feature whose box diagonal exceeds 400 km would put the camera in space; instead frame a 280 km range at pitch 35 degrees below the horizon centred on the box, heading north for a tall feature and east for a wide one.
- *Landmark framing.* Aim height = sampled terrain height, else the cached coarse elevation lookup, else the city's curated elevation, else zero (the app waits a bounded time for the lookup so an uncurated hill town is not framed against sea level). For precise places, building outlines within 180 m are fetched from OpenStreetMap and scored (1,000 per shared query word longer than 2 letters, +500 if the point is inside, minus metres of distance; footprints under 2 m ignored; height from tag else 80 % of the longer side clamped 12 to 80 m). Range = the larger of the range that makes the building's bounding sphere (half the diagonal times 1.18) fill 57 % of the limiting view dimension times 1.05, and 1.35 times the radius. Pitch from shape: very tall and thin (height over footprint at least 2.5) 12 degrees below horizon, tall (1.2) 22, squat (0.35 or less) 45, otherwise 32, no outline 25; searched heading 30 degrees. Flight 3 s (voice 2.2 s) then a final look-at lock to centre exactly, then release. If outline data is unavailable the flight still happens and the toast says so.
- *Settle (ground guard).* After arrival, up to 8 checks 600 ms apart measure the rendered surface under the target and the camera; if the eye is within 120 m of the surface (deficits of 2 m or less ignored) it lifts straight up with a 0.9 s ease. Any key, pointer press or wheel on the canvas, tracking start or newer arrival cancels it.
Detail: 13a section 1.6, NAV-17 to NAV-23, G1, G5, G10 [confidence high; numbers from source].

**Kawn note (Location tray; owner decisions 2026-10-16).** Replace the nine US/world city pills with grouped, bilingual, RTL-ordered buttons with large touch targets: Emirates (Abu Dhabi, Dubai, Sharjah, Ajman, Umm Al Quwain, Ras Al Khaimah, Fujairah); Dubai airports (DXB, DWC, plus recommended AUH and SHJ); London airports (LHR, LGW, STN, LCY, LTN) as a secondary group. Each button flies the camera there; search must still work for any place, in Arabic or English, with Arabic aliases in the offline name index. Keep the sanity gate: emirate-level boxes (the Abu Dhabi emirate includes large desert and offshore islands) very likely trip the same flaw, so test "Abu Dhabi", "Dubai" and "Sharjah" (UNVERIFIED). Replace the public Photon and Nominatim services with licensed or self-hosted geocoders and add a Gulf-aware provider. The voice city list also needs UAE entries (the original lists eight cities, none in Abu Dhabi).

### 4.2 VISUAL PRESETS tray
- **Seven look tiles** in a row with icon, name and key hint: Normal (1), CRT (2), NVG (3), FLIR (4), Anime (5), Noir (6), Snow (7). Active tile is highlighted; tooltips describe the look. Selecting is instant.
- **Map source** section: heading, a small status readout (shows "3D" by default), and one chip per map source in this order: Google 3D, Bing Aerial, Bing Labels, Esri Satellite, OSM. Unavailable sources are dimmed, announce disabled, carry a tooltip with the reason, and show an "ION" badge when they need the free Cesium-ion token. Active chip is reflected from the real engine state (never optimistic), so a failed switch leaves the true source lit. Keyboard: Enter on the tray opener hands focus to the active source chip.
- **Map-source behaviour (census merge).** A single controller owns the imagery layer, terrain and 3D tiles. Each switch gets a new generation number and stale completions are ignored; "silent" switches (boot, scene hand-offs) emit no intermediate state; origin is recorded as manual or automatic; providers are created lazily so a hidden globe never downloads terrain; an unchanged provider keeps its live layer so re-applying the same stack never exposes the bare globe. A failed manual switch tries the recovery source (photoreal when available); an unavailable choice never switches and yields a tailored reason (for example which key is required). The Esri satellite source silently falls back to OSM if it cannot be built, and after two failed tile requests switches automatically, with a message such as "Esri Satellite tile requests failed; using OSM". Every switching, ready and error state is re-announced window-wide so layers re-evaluate their ground. Voice can switch the map with an alias table ("road map" goes to OSM; "satellites" always means the data layer, never a basemap) [confidence high]. Terrain follows the stack: world terrain via the ion token when present, otherwise keyless terrain, otherwise a flat ellipsoid; terrain tiles retry on 429/502/503/504 up to 3 times with exponential back-off from 750 ms to 15 s, honouring Retry-After. Detail: 13a NAV-40 to NAV-54.
- **Kawn note (map source).** Photoreal 3D over Dubai is the hero basemap but coverage is UNVERIFIED; Kawn should hold its own Google agreement instead of the ion community plan, use licensed or self-hosted tiles instead of the public OSM server in production, and check Esri terms for commercial use.
- **Collapsed face** shows "Style: <NAME>" mini status.
- Behaviour on selecting a military look (CRT/NVG/FLIR): HUD turns on in Tactical layout, detection turns on in the dense preset (unless the user already hand-edited detection this session), shader parameters jump to that look's curated start values (e.g. CRT pixelation 1.0, instability 0.42; NVG gain 0.18, bloom 0.22, scanlines 0.96, pixelation 1.0; FLIR sensitivity 0.85, bloom 0.2, mode 0.33, pixelation 1.0), and the Parameters panel opens beneath Display. Normal has no preset side-effects.

### 4.3 Voice capsule (centre of dock)
See section 10.

### 4.4 Dock at small widths
At <= 900 px the dock is clamped to the viewport; when either tray is open the other wing and the voice capsule shrink to icon width (mic visualiser, kicker and readout hidden). At <= 620 px titles truncate to ~2 characters. At <= 720 px trays open higher so they clear the capsule; pinned stacks add their height. Trays take almost the full viewport width on phones.

---

## 5. Left rail

### 5.1 Data Layers panel
- Header "DATA LAYERS" + collapse. The whole panel can also be hidden/shown by the F key (see file 02); it ships visible.
- Body: six headed groups in fixed order, rows in fixed order inside each:
  - **Movement**: Satellites, Live Flights, Military Flights, Local ADS-B, Live Vessels, Street Traffic, Transit, Bike Share
  - **Cameras**: Cameras (CCTV), Recent Imagery, Street Level
  - **Infrastructure**: Mapped ALPR Cameras, Installations, Data Centers, Submarine Cables, Dams
  - **Events**: Space Missions (launches), Quakes, Active Fires, Fire Perimeters, GNSS integrity
  - **Weather**: Wind, Rain radar, Satellite clouds, Lightning, Cyclones
  - **Utilities**: Directions, Radio
  - A layer not in any group would fall under "Other layers" (none ship that way).
- **Row anatomy**: icon, name, a count (loaded count, "1.2K" style from 1000 up, a dash when none, or a layer-specific label), an ON/OFF state button, and a meta line below. Rows for layers that own a rich card elsewhere (weather products, recent imagery) show no count and no chip area.
- **State button text**: OFF; or when on: ON, LOADING, DEGRADED, STALE, PARTIAL, FALLBACK, UNAVAILABLE; transitional ENABLING / DISABLING; UNCERTAIN when the last lifecycle result needs reconciliation. Each state also tints the button. A source that is unavailable (for example key missing) cannot be switched on and shows UNAVAILABLE; its tooltip and accessible name say which key is missing and, when another source of the same layer could work, point to it.
- **Interaction guard**: while enabling/disabling the button stays focusable but ignores clicks (announces busy) so repeat clicks cannot stack; ~one in-flight change per row.
- **Meta line** text patterns: "<source> · <age>" where age is just now / Ns ago / Nm ago / Nh ago / never; "LOADING" with a label; error text with "retry Ns" countdown; "PARTIAL ... n of m records accepted"; "FALLBACK ... <detail>"; "STALE ... retrying in Ns"; guidance statuses (zoom in, nothing here, idle) shown as plain hints, not faults; lifecycle text while enabling/disabling.
- **Row sub-controls** (shown only while the layer is on; hidden entirely otherwise): chips, colour legends with counts, an info line, and for Directions an ordered step list. Known ones:
  - Street Traffic: road-source chips TomTom / OSM / Hybrid (tooltips say TomTom and Hybrid need a key; without one OSM roads are drawn and the status line explains). Density (0.2-2.5) and speed (0.3-3.0) scales exist but only via voice/tools.
  - Satellites: DENSE chip (adds the whole Starlink shell; shows DENSE ... while loading, DENSE x on failure with click-to-retry) and a class legend with counts (stations, navigation, etc.).
  - Recent Imagery: two source chips - "MORE DETAIL · 30 m" and "DAILY OVERVIEW · 250 m"; everything else is in the Recent Imagery panel.
  - Mapped ALPR cameras: "SHOW NEAREST" chip (disabled while an object is being followed) and a legend swatch with count (coverage wedges are illustrative, not measured).
  - GNSS integrity: no chips; info line (regional sample, ~250 nautical-mile circle around view centre, cells from the last 30 min) and a three-level legend with counts; honest tooltip that anomalies are not a detection of jamming.
  - Fire Perimeters: containment legend (not contained/unknown, under 50 %, 50-99 %, fully contained) with counts.
  - Wind / weather / cyclones: rows only toggle; configuration lives in the Weather panel cards (5.4).
- Count text and meta text refresh on a periodic poll; refresh is skipped while the tab is hidden and runs once on return.
- All layers are **off** on a brand-new profile.

### 5.2 Scenes panel
- Header "SCENES", collapsed by default. Contents, top to bottom: scene dropdown + NEW + DEL; CAPTURE SHOT + UPDATE SHOT; shot list; START / STOP / NEXT; EXPORT PRESETS / IMPORT / RUN LOG; progress bar (0-100 % text inside the fill); status line (default "Ready").
- Six built-in scenes ship (Nepal flood incident, global flights radar, orbital watch, thermal threat board, city overload, omniscience pullback - five shots, about 28 s).
- Shot rows: title (double-click to rename inline; Enter saves, Escape cancels, blur saves), LOAD and DEL buttons, a meta line "<LOOK> · <detection mode> · <duration>s + <hold>s". Empty list text tells the user to capture a shot.
- NEW asks for a name in a native prompt; deleting a scene or shot uses a native confirm dialog.
- While a scene runs: every control except STOP is disabled, the panel is flagged running, a runtime caption appears at top-centre, and the body gets a playback class that hides the first-run card; Escape stops (status "Stopped (Esc)"). RUN LOG is disabled until a run exists.
- Import shows a preview (scene list, attributions, missing layers/sources, size) and requires confirmation; Escape or cancel discards; invalid files never damage the current project.
- Interaction buttons authored into a shot appear only after the shot finishes loading; selected via a feature click; Tab/Enter/Space operate them; they are inert during automatic playback and camera travel.
- Project persists in browser storage between sessions (see file 02).

---

## 6. Right rail

### 6.1 DISPLAY panel
Header "DISPLAY" (no drag handle in the current notes), body scrolls internally. Groups top to bottom (HUD and Detect were deliberately promoted to the top):

| Control | Type | Default | Range / options | Behaviour |
|---|---|---|---|---|
| HUD | toggle | ON | - | Shows/hides the four-corner HUD; the Layout row is shown only while the HUD is on [OBSERVED-IN-SOURCE] |
| HUD layout | select | Tactical | Tactical, Operator, Minimal, Cyber | See section 9; Cyber triggers theme change |
| Sonar (Cyber only) | toggle | ON | - | Contact-highlight scanner; status line states if unsupported |
| Sonar rings / range / power / opacity / sector | sliders | 8 / 100 % / 70 % / 84 % / 24 deg | 3-12 / 60-120 % / 0-100 / 35-100 / 8-60 | Session-only; reset on reload; reduced motion disables the sweep |
| Detect | toggle (label shows mode) | ON, dense (the static markup shows 50 % and unpressed, but the first-load baseline sets dense 75 %, Elastic, Fade 7, Outside 1 for every look including Normal) | - | Label shows SPARSE / BALANCED / DENSE by density; the button and the D key switch between off and the last non-off mode |
| Density | slider | 75 % | 0-100, step 25 (<=25 sparse, 50 balanced, >=75 dense) | |
| Allocation | 2-way radio | Elastic | Elastic, Weighted | Elastic: roughly equal label capacity per active layer with redistribution; Weighted: demand-driven with layer weights; persists |
| Fade | slider | 7 % | 0-40 | Width of the label fade band outside the keyhole, as % of keyhole radius |
| Outside | slider | 1 % | 0-100, step 1 | Opacity of labels/cards beyond the fade band |
| Parameters | collapsible sub-panel | follows look | per look | Appears inside Display; one slider per shader parameter of the active look (labels in 4.2 plus Anime saturation 0-2 / edge thickness; Noir contrast 0-2, grain, vignette; Snow density and wind; CRT pixelation up to 10; thermal WHOT/BHOT and Ironbow palette). Slider step is 0.01 when max <= 1 else 0.1; values printed to 2 or 1 decimals. Opens itself on look change, collapses via its own toggle |
| 3D | toggle | ON | - | 3D aircraft models for both civil and military layers; flat icons zoomed out |
| Models | 2-way radio | Proximity | Proximity, All | Proximity swaps in 3D only for nearest contacts when close; All for every in-view aircraft |
| Scope | toggle | ON | - | Circular "keyhole" mask: one fixed canvas under the detection surface, a radial gradient transparent inside and feathering to near-opaque page black outside; redrawn only on resize, tuning change, pixel-ratio change or a quantised alpha step (no animation loop). **Kawn: default ON, kept but with a visible one-tap off** |
| Feather | slider | 11 % (original); **Kawn default 100 %** | 0-100 | Softness of the mask edge as a fraction of the keyhole radius; 0 = hard crop; old links without the field read as 35. (Detection Fade is separate.) The outside fill is only 0.94 opaque above about 10,000 km altitude (stars faintly visible at full-globe altitude) and ramps to fully opaque by about 7,000 km |
| Draw | toggle | OFF | - | Whiteboard mode, see below. Its tooltip tells the user: click vertices, double-click or Enter to finish, Esc to cancel |
| Import | button | - | - | Local geographic files, see below. The picker accepts GeoJSON, plain JSON, KML, KMZ and GPX and allows several files at once |
| Celestial | toggle | OFF | - | Ring of **Sun and Moon** direction markers only (corrected: there are no planet markers); limited to the Normal look; the toggle first flies out to the full-globe composition and the ring turns itself off when the globe is no longer framed; visible when the Earth disc clears the keyhole by 24 px, hides below 12 px; 30 fps, 2-megapixel canvas cap. Bearings come from a published low-precision planetary series, recomputed each minute from wall-clock time because the app clock is frozen. There is no sun-driven day/night shading anywhere; photoreal tiles carry baked daylight. Detail: 13a NAV-64 to NAV-66 |
| Clean UI | button | OFF | - | See 2.5 |
| Bloom | toggle + slider | OFF, 0 % | 0-200 % | |
| Sharpen | toggle + slider | **ON at 49 %** [OBSERVED-IN-SOURCE]: the first-load baseline applies sharpening on at 49 before any link restore, overriding the unlit look of the static markup; selecting a CRT/NVG/FLIR look re-applies bloom off and sharpen on at 49 | 0-100 % | |

Sliders show a live value readout beside them; all support native arrow-key stepping with a visible focus ring.

**Detection and defaults, behind the controls (census merge).**
- The first-run baseline for detection (dense, 75 %) applies to every look including Normal and does not count as the user's own choice; military looks auto-enable the same preset until the user edits detection by hand in that session. Allocation Elastic shares label slots roughly equally between active layers and redistributes unused entitlement; Weighted uses visible demand with square-root scaling and semantic layer weights. The fade band is not the Scope feather.
- Labels are placed by a per-layer quota plus collision solver throttled to a private 125 ms cadence; drawing is batched, pixel-ratio crisp, tracked items keep a box but suppress their label (box 56 x 44 px near, 22 x 14 far).
- **Zero idle cost.** Detection repaints on change and asks for exactly one more frame only while a fade or unsolved work remains; every outstanding-work test must terminate; paint and demand read one monotonic frame timestamp; a skipped paint defers rather than cancels; a changed detectable set marks the solve dirty without itself requesting a frame. Measured: 0 renders per 5 s on a parked empty scene versus 301 with a continuous hold. Coupling to keep: aircraft bracket promptness silently depends on the aircraft layers holding the render loop.
- **Three-surface rule for defaults.** A first-run default lives in four places (engine constant, markup value and readout, UI baseline, share-link generator's starting state) which must move together, while the *parse* fallback for links that omit a field stays at the value of the era the link was written. Only the 3D-models default has durable storage; detection mode, density and Scope feather persist only in the link (500 ms debounce), so a new tab returns to defaults. Kawn changing Scope to feather 100 % must change all four, and keep old-link parsing at the legacy value. Detail: 13f GAP-6, GAP-11; 07 s3.
- The first explicit switch into the Cyber layout sets the FLIR look with the Ironbow palette at 0.42; switching back restores the previous look unless a restored link or scene is authoritative [confidence high].

**Draw (whiteboard).** Turn on, choose shape (Area, Line, Pin), optionally type a label (max 120 characters), pick colour (Primary, Amber, Cyan, Green, Red), click vertices directly on the world (max 512 points per shape; two clicks within 0.5 m count as one). Finish with double-click or Enter; Backspace removes the last point; Escape cancels the in-progress shape, a second Escape leaves draw mode. A live hint line shows measurement (area in ha/km2, line length) and refusals (too few points, shape encloses nothing). Clear removes every mark. A separate cap of 120 simultaneously live marks applies to all annotations (including voice-created ones); beyond it new marks are refused with an "annotation limit reached" result [OBSERVED-IN-SOURCE]. Draw takes exclusive pointer ownership so no layer selects objects under a vertex; if another tool holds the pointer the hint names it. Enter only finishes when focus is on the canvas, page body or the label field - a focused button keeps its own Enter.

**Import.** Accepts GeoJSON, KML, KMZ, GPX via picker or by dropping files anywhere on the app. Each imported file gets a row with show/hide, fly-to and remove. Limits: 12 files per session, 50 MiB per file, 120 MiB inflated KMZ, 20,000 entities, 250,000 positions. Refusals state a reason and the rest of the batch continues. Bytes never leave the device; overlays can appear in screenshots shared with voice. Removing a row moves focus to the next row, else previous, else Import.

### 6.2 CCTV panel
Header "CCTV". Contents: preview frame (image or video canvas), a source badge (source name; "UNKNOWN" until known), a meta line (city, heading, field of view, range, projection state, calibration badge, provider and credit; or "Enable CCTV to load camera intersections"), then control rows:
1. **CCTV OFF/ON** and **NEAREST** (focus nearest camera to the view).
2. **PREV**, camera dropdown (entries "City · Name"), **NEXT**.
3. **FOCUS** (disabled until a camera is active), **COVERAGE** three-state (OFF -> COVERAGE ON wireframes -> VIEWSHED ON colour volumes; **starts at ON** when the layer is enabled), **AUTO HOP OFF/ON** (cycles cameras; **default interval 18 s**, adjustable 8-90 s via tools only) [OBSERVED-IN-SOURCE].
4. **PROJECTION ON/OFF** (camera image draped in the 3D scene as a monitor; default ON) and a calibration chip "CAL · CALIBRATED / CURATED / RAW PRIOR / EDITED (UNSAVED)".
5. **Calibration block**: ADJUST toggle (drag in the world: rings rotate, arrows translate, handles set range and field of view), readout chips HDG, PITCH, FOV, RANGE, HGT (mount height), delta-N, delta-E; clicking a chip turns it into a number field (Enter commits, Escape cancels, leaving the field commits; keystrokes do not leak to global shortcuts). SAVE CAL persists the pose; RESET CAL clears it.
6. **Scene summary** text, revealed with a typewriter effect.
All controls beyond the enable button are disabled while the layer is off or no camera is active. Floating window with 300 x 160 minimum. The active camera's preview frame is re-fetched about every 10 s [OBSERVED-IN-SOURCE].

**Calibration reachability (status).** The census found the calibration block above documented in the panel, but the changelog says the earlier shift-drag calibration interaction was removed and the paired-point calibration is "not in runtime", while a stored calibration record and an edit component still exist. Whether a user can still reach ADJUST, SAVE CAL and RESET CAL in the shipped build is UNVERIFIED (confidence low). Treat the ADJUST gizmo (rings, arrows, handles; each drag becomes one of seven offsets over the original pose estimate; saves go to browser storage with provenance) as live in this document, and the older shift-drag and paired-point routes as removed; do not specify them for Kawn. Detail: 13f section 2.5 and UNVERIFIED list; 13d GND-92, GND-94.

### 6.3 Street Level panel
Header: "STREET LEVEL", an on/off pill (OFF/ON, pressed state, the pill *is* the layer switch), collapse. Collapsed by default; opens itself only on explicit enabling or when a photo opens (not on restore).
- **Providers** chips, one per provider in its own colour. Lighting a chip darkens the others (one at a time); clicking the lit chip turns the layer off. Providers: Mapillary (free key, lit by default), Google Street View (off by default, billed per panorama; appears only on the Google 3D map), KartaView (keyless, off by default). A chip without its key explains the key instead of switching and other chips still work.
- **Viewer area** (appears when an image is open): EXPAND (Esc shrinks; expanded behaves as a dialog with Tab trapped), FIT/FILL radio (whole photo with bars vs cropped to fill; persisted), FOLLOW (the globe camera follows where the viewer looks; only on Google 3D; stops when the map source changes or another feature takes the camera), close (x, also deselects on the map). Caption shows author, capture date and a link to the provider's page.
- **Settings**: IMAGERY radio ALL / 360 / FLAT; SINCE stepped slider 0-8 (0 = any date) whose readout names the cut-off date; both hidden when no lit provider honours them (Street View is all-360 and ignores them; KartaView filters by upload date and a note says so). A key problem disables the filter block but keeps provider chips focusable.
- **Legend** of coverage colours, **coverage line** (counts or "zoom in" hints), and an **OPEN STREET VIEW** button (Street View only; enabled at street zoom, tells the user to aim at a street if pointed at sky).
- Error strip (alert) for rejected keys or failed loads. Status words on the panel: KEY REQUIRED, KEY REJECTED. At phone width the viewer height is derived from the available band so the whole photo fits.
- Keys inside a KartaView panorama: left/right turn (or step in a flat photo), up/down pitch, plus/minus zoom; arrow chevrons step along the drive.

### 6.4 Weather panel (appears only while a weather layer is on)
Title "WEATHER" + count of active products. Body: one card per active product, each with a settings section (closed by default; disclosure memory lasts the page session): wind (model GFS or ECMWF, field None/Speed/Pressure/Temperature, units km/h, m/s, mph, Pause/Resume, "Read wind at map centre" which captures a reading with coordinates, speed, direction, scalar value, valid time and freshness and drops a map marker; unit chips reformat without resampling; model/field change clears it), observed products (coverage, image treatment, opacity light/strong), cyclones (storm list that selects and flies to an advisory, official advisory link). **Observed history** block (only if rain radar, satellite clouds or lightning is on): one shared timeline slider with earlier/later steppers, Latest, Play/Pause, a readout, and oldest/newest labels; dragging previews instantly and commits after a 150 ms pause; each product shows "no frame within N" when none is near the chosen time. Honest labels: wind is forecast, animation does not advance time.

### 6.5 Recent Imagery panel (appears while that layer is on)
Fixed-height blocks so nothing moves: box actions (SELECT BOX drag on map; USE VIEW; CLEAR), notice line with ZOOM IN (box must fit 1,000 km), a horizontal listbox strip of daily cards (newest first, last 30 days; each shows date, sensor and resolution, cloud percentage or "no imagery"; a "START HERE" badge marks the recommended day), a one-line hint that changes per state, mode radio IMAGE / VS BASEMAP / A / B, two slot rows with unpin x, opacity slider (default 100 %), SWAP, EXPORT (per side in A/B), and a collapsed DETAILS card (notes and hedges; optional "hide empty days"). Comparison shows a draggable divider (arrow keys, Home, End). The Google 3D map is replaced by Esri imagery while a comparison is showing and returns when cleared.

### 6.6 CONTEXT panel
Header "CONTEXT" with a small radio launcher in the header and the collapse toggle. Collapsed by default.
- **Mode tabs** CONTACTS and SPACE MISSIONS (tablist; Left/Right/Home/End move; both reachable by Tab in every state). Until one is chosen a standby note explains both.
- **Contacts view**: COCKPIT button (shown only when eligible), SEARCH NEARBY SITES, and the context card: subject line with the 250 km window; PREVIOUS / FOCUS / NEXT; cohort rows (civil flights, military flights, vessels, mapped sites), each with a count (exact, or a lower bound with an unknown marker), the data source and a one-line reason, and up to three nearest entries per page. Pages auto-rotate every 10 s; up to 10 examples. Entries are buttons that focus that object; unavailable entries are plain text. Focus is preserved by identity during live refresh; if the focused entry leaves, focus falls to the explanatory note at the end. A footer note states that missing broadcasts, unloaded areas or unmapped sites are not evidence of absence. Refresh cadence 750 ms parked, 175 ms while the camera moves (settles after 250 ms still).
- **Space Missions view**: roster headed "AVAILABLE MISSIONS" with a count; loading state "loading 30-day mission index"; items preview on hover or keyboard focus (temporary globe rotation and marker highlight) and select with Enter/Space; hint line says Tab previews, Enter/Space selects.
- **Radio section** (nested, collapsed): ENABLE; station-tag select (All, news, talk, weather, public safety, aviation/marine, traffic/transit, music, other, plus generated genres); station card (name, meta, tags); tuner dial (hidden until stations exist) with needle that snaps to real stations - drag, or keys Left/Right (+/-1), PageUp/PageDown (about a tenth of the list), Home/End; Escape cancels a drag; release commits and flies to the broadcaster; PREV / PLAY / NEXT / STOP; volume 0-100 (default 80); station site link; Radio Browser directory link; privacy line (audio connects directly to the broadcaster, which sees the listener's IP). A **compact radio** in the Context header (and a third copy inside cockpit) offers enable, prev/play/next and volume, plus "open full" and close.
- **Local SDR card** (Chrome/Edge WebUSB only): connect, locate (use own position), change device, mode FM or ADS-B 1090, gain select (auto plus about 29 fixed steps from 0.0 to 49.6 dB), FM frequency field 87.5-108 MHz step 0.1 (default 98.5), TUNE, seek back/forward, volume (80), stats strip in ADS-B mode (messages/s, heard, positioned, signal level), status lines. One tuner is shared by FM and the Local ADS-B layer. Selected device and gain persist.

---

## 7. Modals, cards and settings flows

### 7.1 First-run launcher ("choose your first view")
- Card with scanline accent, kicker "MISSION CONTROL · FIRST LAUNCH", heading, one persuasive line (forbidden-cockpit feeling, sources public, data real), four tiles, a "do not show again" checkbox with an "ESC to dismiss" hint, and a status line that normally holds a voice tip.
- Tiles: LIVE CONTACTS (opens Context in Contacts mode with civil and military flights), SPACE MISSIONS (opens Context in Space Missions), ENVIRONMENTAL (flies out to the globe, turns on earthquakes and active fires), EXPLORE MANUALLY (nothing changes).
- Choosing a tile acts exactly as the user clicking the equivalent controls: layer enables are real saved preferences; the context panel opens only for the two context tiles; camera change is session-only; detection, scope, 3D and look are untouched.
- Busy: all tiles go aria-disabled (not disabled, to keep keyboard focus), the status line shows a mission-specific message. Success closes it. Failure keeps it open with "could not open that mission (<layers>), retry or explore manually" until the next attempt.
- Show policy: appears every new browser session; never for a visitor arriving by a share link or when embedded; a support switch can force it on or off (forcing on outranks every suppression). Only the checkbox makes dismissal durable; mission choice, Explore and Escape dismiss only for the tab session. If storage is blocked the checkbox reverts itself and the line says storage is blocked.
- Show-policy detail (census merge): the launcher is *not* one-shot; it returns each new browser session. A page option of 0 suppresses it and 1 replays it (outranking both flags). Every close except the checkbox writes a session-scoped flag only. A mission enables its own layers at user origin (durable, as if clicked), reveals Context for the two context missions, moves the camera (session only), and deliberately does **not** touch detection, allocation, 3D models or Scope. The reason: setting the detection override flag would silently disable the military-look auto-preset for the whole session; missions therefore run through the same context-mode and layer-enable paths as the visible controls. Environmental turns on quakes and fires and is honest about a missing fires key [confidence high]. Detail: 13f SYS-180, GAP-11.
- Kawn note (first-run): replace the four tiles with UAE persona/mission entries (Skyline, Ports, Weather and dust, Explore; or Investor / Visitor / Government as in the Adapt list below), keep the session-versus-durable dismissal rule, and keep the "never touches Scope" principle so the Scope-on, feather-100 default view is stable.
- Keyboard: focus lands on the first tile, Tab cycles inside, Escape dismisses, focus returns to where it was. It never fights for Escape: it yields (dismisses for the session, without stealing focus) the moment cockpit, scene playback, recording or clean view takes the screen; it waits (does not appear) if such a mode is already active; and it ignores a hit-test where something is painted over it (for example the credits lightbox). It re-measures its scroll fade on rotation (fade only if tiles overflow; very short screens <= 620 px high shrink it).

### 7.2 Provider settings ("POWER UP")
- Bottom-right chip (bolt icon) labelled with how many keys are waiting ("POWER UP - n KEYS WAITING"); correction: the chip *retires (hides)* when every visible key is set, so "POWERED UP" appears only when the dialog is reopened (the project's readme line describing a persistent "POWERED UP" chip does not match the shipped behaviour); the dialog stays reachable this session and via a URL switch. It only exists on a local/dev server with a settings service; on hosted builds or LAN visitors the chip and dialog are removed.
- Dialog: "Ground station · provider settings", close button, explanatory paragraph (app flies keyless; each key switches on another real feed; saved to local configuration, then the server restarts and the page reloads itself; server-side keys stay on the machine; browser-exposed keys must be restricted at the provider; externally managed keys are shown but never touched), one row per provider, SAVE KEYS, "ESC to close", status line (default text praises the Google key as the biggest unlock; shows Saving..., "Paste at least one key first", errors, success then restart notice).
- Rows (**nine visible; ten in the registry**, corrected): Google Maps (photoreal planet, place search, Street View; metered), OpenAI (voice; API key or ChatGPT sign-in, with a toggle between the two and a polling sign-in flow with 2-minute limit), AISStream (ships), NASA FIRMS (fires), TomTom (real traffic), Cesium ion (Bing imagery, terrain), OpenSky (more flight credits), Mapillary (street imagery), Launch Library (missions). A separate server Google key is hidden. Each row shows a status LED, a tier dot (metered vs free), what it unlocks, a link to get the key, an input, and a remove button for saved keys. Values are bounded (max 512 characters, at most 16 changes per save).
- First Google key: the stored address is cleaned of an auto-picked keyless basemap so the reload boots the photoreal default.
- Mechanics (census merge): the dialog is rendered entirely from a server status listing (one row per key: what it unlocks, where to get it, a paste field). SAVE writes the project's local configuration file, made owner-only *before* the secret is written, then restarts the dev server, after which the page reloads. Values supplied by the shell or the system keychain show "configured externally" and are read-only. The whole surface removes itself in a production build or for a non-loopback visitor, and is disabled while the server is LAN-shared. The ten registry rows are Google browser key, Google server key (hidden), OpenAI, AISStream, FIRMS, TomTom, Cesium ion, OpenSky, Mapillary and Launch Library. A command-line "setup doctor" gives the same readiness report without printing values (Node 24.14+ or 26 required) [confidence high]. Detail: 13f SYS-181, SYS-182.
- Kawn note: replace with server-managed secrets and an operator-only admin screen; no file editing, no user keys, no browser-visible keys beyond provider-restricted map keys.
- Keyboard: same capture-phase Tab trap and Escape as the launcher; focus returns to the chip.

### 7.3 Credits lightbox
Attribution button (named, Tab-reachable, Enter opens immediately, Space on release), lightbox focuses Close, Escape/Close/backdrop dismiss and focus returns to the button. Cockpit stays active behind it. Per-layer licences (open-database, non-commercial share-alike for cables, fire, satellite, seismic and city open-data sources) are registered once as static credits, hidden from the on-screen line and shown in this expandable popover; imagery credits follow the source actually displayed, including after a fallback. The credit container sits outside the clean-view hide rules, and the provider's required logo or text must stay [confidence high]. Detail: 13f SYS-183; 13a NAV-54. Kawn: a legal must, with bilingual wording.

### 7.4 Native dialogs
Scenes use the browser's prompt and confirm for naming and deletion. File pickers use the native chooser.

---

## 8. Cockpit (first-person aircraft view)

Entry: COCKPIT action in Contacts view, or the C key; allowed only when Contacts context is on, both live and military flight layers are enabled, and a civil or military aircraft is tracked. Exit: EXIT COCKPIT button (also shows keyboard hints), Escape, C, RESET (exits and flies to full globe), or selection loss.
- On entry the map panels collapse (their prior open/closed snapshot is restored exactly on exit); opening Data Layers inside cockpit temporarily collapses the Contact rail, restoring it only if that action collapsed it.
- Overlays: visor frame lines, target geometry, roll scale (+-30), pitch rails (+-20), horizon "LEVEL" guide, status words "OPTICAL PLANE" and "VISOR LOCK". Top line: "FIRST PERSON", callsign, meta. Vision carousel PREV / CURRENT / NEXT over the seven looks (wraps; clicking the current one advances; starts on the active map look). Lower-centre instrument cluster: ground speed (kt), heading (deg), altitude (ft), compass tape; clock (UTC) and position readout. Curved altitude and speed tapes hug the visor rim (suppressed on narrow screens). Estimated-destination chevron + bearing and a FROM -> TO strip appear only if a plausible route exists.
- **Contact rail** (left): 250 km subject window, four cohort counts, nearest observed/mapped example with relative bearing and distance, freshness, uncertainty line, Previous (prior visited) / Next (nearest unvisited), collapse, and a "WX" weather-effects toggle (default OFF; the opt-in persists; clouds are a low-resolution transparent pass, fail clear when weather data is missing).
- **Briefing rail** (right): three pages - live signals, regional news (headlines link out to publishers, never treated as verified), local info (temperature, wind, sky, precipitation, attributed weather source). Previous / Next, "n / 3" position, direct SIG/NEWS/LOCAL tabs, CYCLE OFF/ON (nine-second auto-advance; paused while hovered or focused; off by default); collapse. Missing observations are shown as unknown, never as zero.
- **Utility strip**: DISPLAY and RADIO launchers open pop-overs (the Display controls are re-homed into slots: HUD, detection, parameters, 3D; radio compact player). Escape closes an open pop-over before anything else.
- Left/right map panels, CCTV and Context are hidden in cockpit; empty-space clicks are ignored; narrow screens use stacked top/bottom fallbacks (<= 760 px).
- Not a VR session.

---

## 9. HUD overlay and layouts

Four layouts selectable in Display: **Tactical** (default; four-corner brackets, classification-style banner, summary, REC clock, MGRS, lat/lon, GSD/NIIRS, altitude/sun, edge metadata, bottom bar), **Operator** (centre banner and framed panels; exposes the Context panel more prominently and hides Scenes per field notes), **Minimal** (reduced), **Cyber** (red accents, theme change, Sonar). The banner text is cosmetic. Common readouts: mode label, one-line summary (look, altitude band, "NEAR <landmark>" within 150 km else sector coordinates, region, altitude, window size, sun elevation, off-nadir angle, UTC offset, provenance tag), UTC/REC clock ticking each second, telemetry every 250 ms. Summary text regenerates every 15 s and on camera settle; an AI summary replaces the deterministic line when available and it types in two characters per 24 ms. The summary can lag the camera (cosmetic). The "NEAR" tag uses a catalogue of only eight cities and a coarse region rule that mislabels the Gulf (Abu Dhabi reads as Africa upstream); Kawn needs its own UAE point-of-interest catalogue and region logic, and a local grid or Makani-style reference in place of MGRS. The AI line is exactly five words, produced from basemap labels and enabled layers only, with a 5 s abort then the deterministic fallback; when no key is configured the route answers gracefully rather than failing (13f SYS-103, SYS-104). Both altitude readouts share one datum: camera height is ellipsoidal and the displayed mean-sea-level height subtracts a cached geoid undulation per coarse cell (SYS-102). Detection mode banner text is developer-only (hidden unless a debug switch is used). Hidden at <= 720 px: edge metadata strips.

---

## 10. Voice control capsule

- Centre of the dock. Contents: kicker "AI AGENT", a model-tier button (STD / MINI; applies to the next session; persists), estimated session cost readout coloured by level (starts "~$0.00"), the round mic button (label ON/OFF, accessible name explains toggling and hold-Space), an audio-level visualiser, a status readout, and a help tray ("hold Space to speak, tap Space to activate focused controls").
- **Interaction**: click the mic = open-mic conversation until clicked again. Hold Space for 500 ms (on the map, page background, or even over a focused control) = push-to-talk; release to send. A short Space tap keeps its normal effect on a focused control. Holding Space while the assistant speaks interrupts it. Radio is paused while voice listens. Text fields are protected.
- **States**: OFF, CONNECTING, LISTENING (sub-lines "Hold Space to talk" / "Release Space to send"), EXECUTING, ERROR (error tray with alert role, a hint to check microphone permission and network, and DISMISS).
- **Cost control**: soft warning at about $2 (one visual cue), hard cap at about $5 closes the session; both persisted and editable; a disabled threshold round-trips correctly.
- **Voice card** (above the capsule, dismissible): YOU caption, optional "THIS" pointer line (what the pointer was over when Space went down), assistant caption, a numbered plan with per-step status, a result block (title, chips, lines, referenced items, collapsible notes), and a polite live region for screen readers.
- Pointer context: on Space-down the app snapshots what is under the pointer so "this" can be resolved; a reticle can be shown.
- Requires a key; capsule shows an error state if unavailable.
- **Spend readout and cap (census merge).** The estimate folds per-response usage into a session total with cached tokens subtracted per modality; unrecognised model ids and unattributed residue are billed at the most expensive known rate (uncertainty resolves upward); a response cut off by teardown never reports usage, so the readout shows a trailing asterisk (incomplete) rather than guessing. The model tier chosen applies to the *next* session; the meter is bound to the model the server echoed, not the tier requested, and may be replaced only when the session is fully settled (inactive, no data channel, no peer connection) so a late completion cannot be lost or dodge the cap. At the cap a latch stops *new* tool dispatch but tools already running finish. Thresholds are one persisted pair; zero or negative disables a threshold (stored as an "off" sentinel); corrupt storage falls back to defaults rather than disarming. Detail: 13f SYS-005 to SYS-008, GAP-4.
- **Typed commands** supersede an interrupted turn; a refused late tool call is still answered once with a terminal "superseded" result, because an unanswered call would deadlock the model (SYS-010, SYS-011). Progress narration is a code-authored timing policy (soft two-note tick after 0.7 s of silence only when the mic is muted; first progress line near 1.5 s; at most two step lines at least 3 s apart; one "still working" line near 8 s) (SYS-013).
- Kawn note: Arabic labels and RTL card; show the cost in AED; replace the cloud realtime provider behind the same two-operation backend seam with a UAE-hosted or on-device speech stack.

---

## 11. Responsive and mobile behaviour

Evidence legend for this section: [OBSERVED-IN-SOURCE] means the behaviour is spelled out in code or styling; [FROM-TESTS] means a test pins it; [INFERRED] means reasoned from the above, not seen running.

| Width / height | Change |
|---|---|
| <= 980 px | Dock proportions tighten |
| <= 900 px | Dock clamps to viewport; opening a tray shrinks its sibling and the capsule; attribution moves above the dock |
| <= 760 px | Cockpit rails use separated top/bottom fallbacks; Cyber short-screen variants |
| <= 720 px | **Mobile layout.** Both rails go edge-to-edge (16 px side margins) and become vertical scroll containers: the left rail runs from 70 px below the top to just above the screen midline, the right rail from just below the midline down to a floor that tracks the mandatory credit line (so the credit stays readable at every height); panel positions are released to the flow; panel glow is pinned inside the panel so it creates no scrollbars; Display compact controls are 176 px wide; the style indicator and the HUD side-edge strips hide; the top-centre buttons move to the top-right (offset by the device's top safe-area inset); HUD bars become full width; the voice capsule sits bottom-right at a 196 px minimum; CCTV preview height is capped (120 px); trays are full-width; Street Level viewer height derives from its band; popups must stay inside the rail (outward popups clip) |
| <= 620 px | Dock titles truncate; first-run and settings dialogs use compact sizes |
| <= 575 px | The data-credits lightbox becomes a full-screen sheet [OBSERVED-IN-SOURCE] |
| <= 520 px | Brand text and tagline hide; logo only |
| short heights (<= 620/640 px; Cyber cockpit decor also trims at <= 700 px) | First-run card and Street Level compact; scroll fade appears only if overflowing |

The layout engines that normally measure obstacles and re-flow the rails skip themselves entirely at <= 720 px and mark the rails as mobile [OBSERVED-IN-SOURCE]. In the mobile layout the Display panel is not counted when deciding whether any right-rail panel is expanded.

**Touch and pointer evidence in code**
- Dock trays and panels open on hover only for mouse and pen pointers; touch pointers are ignored for hover, so a tap (click) is the only way to open a tray on a phone [OBSERVED-IN-SOURCE].
- Floating-window dragging by the header is for mouse and pen only; a finger on a header is a rail scroll [OBSERVED-IN-SOURCE]. Window resize handles do declare themselves touch-exempt (the browser will not scroll under them), but because dragging is mouse/pen only it is UNVERIFIED whether a finger can actually resize.
- Elements that explicitly take over touch gestures from the browser: the radio tuner dial, the Recent Imagery comparison divider, a draggable handle in the Nepal flood demo, and the KartaView panorama (one finger drags the view, two fingers pinch the field of view) [OBSERVED-IN-SOURCE].
- The globe itself uses the 3D engine's stock touch handling (one-finger drag, two-finger pinch/rotate/tilt) - [INFERRED], not remapped by the app. Cyclone tracks listen to touch-end as well as mouse-up when deciding what was clicked.
- The logo gaze effect ignores touch [OBSERVED-IN-SOURCE].
- The page declares a plain device-width viewport without the edge-to-edge fit option, so on notched phones the safe-area insets that the styling reads (top-centre buttons at <= 720 px, and the cockpit bottom controls) most likely resolve to zero [INFERRED]. No other chrome reads safe-area insets.
- No bottom sheets, drawers, swipe gestures or long-press actions exist: mobile is the desktop layout squeezed into two scroll bands. No test suite for the mobile layout was found beyond style-rule assertions.
- **Touch parity gaps** (features reachable only by keyboard): the F key (hide/show Data Layers panel), the O key (orbit a landmark - no on-screen control found, though a voice tool may reach it), the backtick frame-rate readout, hold-Space push-to-talk (the mic button gives open-mic instead), landmark letter keys (the pills still work by tap), Escape (every panel still has a collapse button; scene stop has a button). Look tiles, HUD, Detect, Clean UI and CCTV all have tappable controls.
- The settings chip may be hidden in compact layouts (a URL switch reopens it).

---

## 12. Embed mode

An embed switch (or an inline host flag) shows the globe alone: body flagged as embedded, clean view forced on, HUD, first-run card, settings chip, dock, both rails, label-access list and view switcher hidden; attribution stays. An embedding page controls it by cross-window messages: it can send a request to show a *view* (look, map source, exactly-these layers, annotations, camera, optionally a followed aircraft/satellite and cockpit) and receives an acknowledgement listing each step with ok/failed, plus a ready announcement once the app can accept views. Views are applied one at a time in arrival order; replies go only to the asking origin (or any origin for an inline panel or sandboxed/null origin). A followed entity is retried up to 20 times at 1 s intervals; otherwise the camera flies in 2 s. Inline panel hosting keeps drawing from a 33 ms timer if the host stops animation frames. Annotations carried in an opening link (max 24 marks, 6,000 characters) are drawn after restore whether embedded or not.

Order of application for an incoming view (census merge): look, map source, exactly the view's layers, annotations, then the followed entity (retried up to 20 times at 1 s until its layer holds it), and only then the camera, because a camera flight would end a follow; cockpit last if asked. The acknowledgement carries ok plus the per-step results and goes to the sender's origin; the app announces ready; only the parent frame may send. **Framing policy:** every document refuses framing by default (frame-denial header and a frame-ancestors "none" policy); an operator allow-list (or "any") applies only to embed-mode documents; the MCP panel needs no framing. Detail: 13f SYS-184, SYS-185. Kawn: needed for partner embeds (developer portals, government dashboards); add an explicit allow-list per partner.

---

## 13. Accessibility behaviour

- Native buttons/selects/sliders/checkbox with accessible names everywhere; collapse buttons expose expanded state; toggles expose pressed state; radio groups use roving tab stops with arrow/Home/End.
- Busy/disabled convention: lifecycle buttons use "aria-disabled + busy" (stay focusable) rather than native disabled so focus is never lost; real disabled is used for truly unavailable controls.
- Visible focus rings are global, shown on selected/active items too, inset where frames would clip them (dock, cockpit launchers, location pills, layer buttons, sliders, camera dropdown).
- Live regions: loading strip, toast, tray mini-status, map-stack status, radio playback state, cockpit briefing position and signal list, voice card, first-run and settings status lines (polite); error trays and the Street Level error strip use alert.
- Dialog focus: launcher and settings trap Tab, Escape closes, focus returns; credits lightbox likewise; expanded Street Level viewer behaves as a dialog.
- Escape model: one key, closes the nearest thing (see file 02).
- Reduced motion: honoured for the radio dial, wind animation (static flow), sonar sweep, cockpit/clean-view chip animation, panel transitions and cyber effects; no glow animation is forced.
- Space is overloaded (activate vs push-to-talk) but resolved by the 500 ms rule without losing native activation.
- Weak spots: no screen-reader description of the globe itself beyond the target list; canvas labels are not clickable by assistive tech other than via that list; icons-only top buttons rely on tooltips/names; colour-coded state chips lean on colour plus text (text is present).

---

## 14. State catalogue (empty, loading, error, degraded)

| Situation | What the user sees |
|---|---|
| Layer loading | Row meta says loading; state button LOADING; after 160 ms the global strip appears |
| Layer needs a key | Row state UNAVAILABLE (cannot be turned on) or "KEY REQUIRED" on panels; tooltip names the key |
| Feed down | Row ERROR text with retry countdown; button DEGRADED or STALE; strip shows failure for 5 s |
| Fallback source in use | Button FALLBACK; meta names the source and coverage (for example a community aggregator at a 250 nm window) |
| Partial data | Button PARTIAL; meta "n of m records accepted" |
| Guidance (zoom in / nothing here / idle) | Neutral hint, never red |
| Viewport-bound data empty | Shown as guidance until zoomed to a city; flagged incomplete |
| Voice unavailable | Capsule error state with dismiss |
| Provider settings unavailable | Chip silently removed |
| Search not found / failed | Toast |
| Mission launch failed | Launcher stays open with retry message |
| Map source unavailable | Chip dimmed with reason; automatic fallback to another imagery if the preferred provider is unreachable |
| Storage blocked | Preferences silently skipped; launcher checkbox reverts with explanation |

---

## 15. First-run default state summary

Map: Google 3D when its key exists, otherwise Esri satellite; camera flies to Austin (25 km high, then 4 s down to 600 m); HUD on, Tactical; Detect on, dense 75 %, Elastic, Fade 7 %, Outside 1 %; Scope on, Feather 11 % (**Kawn: Dubai, Scope on, Feather 100 %**); 3D on, Proximity; Bloom off (0 %); Sharpen on at 49 %; Celestial off; no layers on; all dock trays compact; left Data Layers visible but collapsed to its compact button; Display, Scenes, CCTV, Street Level, Context, Radio collapsed; launcher shown once per session; FPS readout hidden; voice off.

---

## Kawn notes

**Keep**
- One-surface layout (map plus floating panels), collapsible panels with explicit-versus-automatic collapse distinction, Escape-closes-nearest model, hover-open/pin/unpin trays, delayed loading chip (160 ms) with long-wait escalation, honest layer states (ON/LOADING/STALE/PARTIAL/FALLBACK/UNAVAILABLE) with source and age on every row, guidance-not-error states, "key/permission required" tooltips naming what is missing, toast and first-run patterns, credits button that is keyboard-operable, aria-disabled-not-disabled for async buttons.
- Share-link-aware first-run (never shows to a link recipient); session-versus-durable dismissal rule.

**Adapt (UAE city-intelligence, Arabic + English RTL)**
- Replace the five generic missions with persona entries (Investor / Visitor / Government) and save the choice as a profile preference rather than a boolean.
- Rails mirror in RTL: Data Layers on the right, context on the left, chevron glyphs flip, dock wings swap, slider fill direction flips, numerals toggle Western/Eastern; compass and timeline directions need explicit decisions.
- Mobile should not be "two scroll bands": use a draggable bottom sheet (peek/half/full) with a thumb-reachable action bar and 44 px targets; keep trays as sheets, not hover popups.
- Layer group names: replace Movement/Cameras/Infrastructure/Events/Weather/Utilities with city groups (Mobility, Real estate, Services, Environment/Weather incl. heat and dust, Events, Utilities).
- Look tiles: Day, Night city, Heat, Blueprint, Noir (rename military looks); keep numeric hotkeys but document them in-app.
- Data Layers meta line pattern (source - age - state) becomes the provenance chip with official/modelled/crowdsourced tag and bilingual timestamp.
- Location tray: superseded by the owner decision of 2026-10-16: grouped buttons for all seven emirates, Dubai airports (DXB, DWC, AUH, SHJ) and London airports (LHR, LGW, STN, LCY, LTN), Arabic labels in RTL order, large touch targets; Dubai is the default start location; bilingual search understanding transliterations (see 4.1.1). Curated landmarks per emirate can still be added behind each button.
- Default view: Scope ON with feather 100 % on first load, with a visible one-tap way to turn Scope off and to change feather; define "centre" as the free map area (not the full window) when side panels are open and on phones, and put the most important live data in the centre zone.
- Keyboard-only features (F, O, backtick, hold-Space, Q to T) need on-screen equivalents; add accessible keyboard pan and zoom since the original has none (file 02).
- Keep a persona-neutral "Power up" concept only as an operator-side admin screen; no user keys in the public product.
- Voice capsule: keep push-to-talk idea (but Space-hold conflicts with Arabic IME usage and focused controls; consider a visible hold button plus key), add bilingual captions.

**Skip**
- Classification-banner HUD styling and military/ISR vocabulary (TOP SECRET banners, MGRS/NIIRS/ONA, FLIR/NVG names), Cyber sonar, TR-3B easter egg, SDR/Local ADS-B card, CCTV/ALPR panels, military aircraft and installations rows, Pinokio/local-key flows, cockpit WebXR-like visor (retain only as a later "ride the flight" tourist feature), Celestial ring.

---

## Open questions / UNVERIFIED

**Resolved in this revision (moved into the body above)**
- Display first-run state: collapsed (code and the newest project notes agree; older notes are stale).
- Sharpen default: on at 49 %.
- Whether the HUD layout row hides when HUD is off: yes.
- Dubai's five landmarks: now listed.
- Safe-area handling: only the top-centre buttons at narrow widths and the cockpit bottom controls read it; the viewport declaration likely zeroes it.
- Logo gaze on touch: ignored on touch.
- CCTV preview refresh (about 10 s) and default auto-hop interval (18 s).
- Double-click-to-track: the stock engine action is left in place and is only suppressed while the draw tool or the imagery box tool is active.

**Still open**
- Whether the Weather and Recent Imagery panels really appear expanded the first time their layer is switched on (inferred from markup; stored/link preference would override).
- Whether the settings chip ends up before or after the other buttons in the top row under the Cyber theme.
- Whether a finger can resize a floating window (handles are touch-exempt but dragging is mouse/pen only).
- Exact visual treatment of the loading screen logo gaze.
- Real-device behaviour of the two-scroll-band mobile layout (no device evidence; only styling).
- Whether the CCTV calibration controls (ADJUST, SAVE CAL, RESET CAL) are still reachable in the shipped build after the shift-drag interaction was removed (13f).
- Whether a keyless substitute exists for the Places-based search rescue; whether the Nominatim proxy uses the bias it is sent; Arabic and bilingual query quality on the public geocoders; photoreal 3D coverage over Dubai and Abu Dhabi (13a).
- Real-device voice behaviour on iOS Safari and Android Chrome (WebRTC, 500 ms hold on touch) (13f).

## Revision log

- Re-verified numeric claims against source: loading strip delays, toast length, reset flight time, panel widths, hover delays, CCTV and Context cadences, voice cost thresholds, import limits, draw limits all confirmed; Display expanded width corrected from 330 to 272 px and Data Layers expanded width added (280 px).
- Resolved the Display default-state conflict (collapsed on first run), Sharpen default (on at 49 %), HUD layout row visibility, Dubai landmark list, CCTV defaults and cadences, tilt angles and globe-reset flight time.
- Added evidence-tagged mobile and touch section with a touch-parity gap list (keyboard-only features), pointer-type rules for hover and dragging, safe-area findings and touch-owned widgets.
- Added start-up behaviour (opening flight to Austin without a link, loading-screen stage messages), 120-mark live annotation cap, and Escape-needs-focus rule for panels.
- Added confidence tags to the sections that changed; unresolved items trimmed to those that need a running device.

### Pass 3 (census merge)

Sources: 13a (globe, camera, navigation) and 13f (voice, HUD, looks, platform); master list in 13-feature-catalog-master.
- Corrections applied: Celestial row is Sun and Moon only (no planets); world-jump rule narrowed to pill clicks with a different preset city active; provider settings has nine visible rows and ten in the registry; the settings chip retires when all keys are set and "POWERED UP" is not a persistent label; first-run launcher is per session, not one-shot; tray landmark keys are conditional on an open landmark row; no keyboard zoom or pan; no sun-driven day/night shading.
- Added: opening-flight numbers and loader stage list; bare-viewer note; idle render governor; Apple GPU workaround; tilt, north-up, reset and zoom-limit mechanics; status-strip verdict wording and honest feed states; new subsection 4.1.1 (search and fly-to: owner check, provider chain, coordinate grammar, rescue, framing-mode table, box fit, sanity gate, swath, landmark framing, ground guard); map-source controller behaviour; Scope and feather mechanics; detection behind-the-controls rules and the three-surface default rule; CCTV calibration reachability status; first-run show-policy and mission side-effect reason; settings dialog mechanics; credits detail; voice spend/cap ordering rules; HUD summary region caveat; embed order and framing policy.
- Kawn notes added: Dubai default, Scope on with feather 100 %, emirate and airport location tray, Arabic search, geocoder replacement, sanity-gate testing for emirate boxes, admin-only key management.
- Confidence tags updated on all touched text; remaining uncertainty moved to Open questions.

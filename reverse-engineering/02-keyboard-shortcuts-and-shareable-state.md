# 02 - Keyboard shortcuts, gestures and shareable state (behavioural reverse-engineering)

**Purpose.** This note documents (a) every keyboard shortcut and widget-level key in the reference product with its context, effect and conflicts, (b) the mouse, trackpad and touch gestures, and (c) exactly which application state is written into the shareable link, what a recipient gets back, what is never shared, and which preferences are session-only versus persistent in the browser. It describes behaviour only; parameter names, storage keys and code identifiers are intentionally omitted. Items not confirmed are marked UNVERIFIED.

---

## Part A - Keyboard

### A.1 Global rules (apply to every shortcut)

1. **Typing protection.** Single-key shortcuts are ignored while focus is in a text field, select list or text area (this includes range sliders and checkboxes, which are input elements), and in the location search field. **Escape is the only page-wide key that still acts while typing** (it clears/collapses the search field - from any target, not only the search field itself). Protection is by element type only: a rich-text (editable) region and a focused plain button are *not* protected, so shortcuts still fire with focus on a button [FROM-TESTS]. The landmark letter keys use the same protection.
2. **Capture-phase surfaces get first refusal.** Open dialogs (first-run card, provider settings), the cockpit view, the push-to-talk Space handler and the credits lightbox listen before the page-wide shortcuts and may consume the key.
3. **No modifier or repeat discipline in the page-wide set** [FROM-TESTS]. The page-wide letter and digit shortcuts check neither modifier keys nor key-repeat: a test explicitly presses a held-down, Ctrl-modified key and expects the action to fire. So a browser chord such as copy, find, bookmark, new-tab-switch (Ctrl/Cmd plus a digit changes the look *and* switches the browser tab) also fires the shortcut, and holding a key (H, V, F, D, C) flickers the toggle on every repeat. Cockpit's C, the frame-rate key and push-to-talk Space *do* ignore modified presses (push-to-talk also ignores Shift). This is a defect worth not repeating.
4. **Case-insensitive** for letters. Matching is on the typed character, not the physical key, so it follows the keyboard layout (a layout where digits need Shift, or a non-Latin layout, will not trigger them) [INFERRED from the typed-character comparison]; the frame-rate key alone also accepts the physical backquote position.
5. **Native keys on controls are never hijacked**: Enter on buttons, Space on buttons (release-activated), arrow keys on sliders/radios/selects stay native.
6. **Key repeat** is ignored for toggles that would flicker (cockpit C, frame-rate readout, disclosure Enter).

### A.2 Page-wide shortcut table

| Key | Context | Effect | Conflicts / notes |
|---|---|---|---|
| 1 | Anywhere (not typing) | Look: Normal | Also changes saved look in the link; does not pass through the "user hand-edited detection" flag |
| 2 | same | Look: CRT | Auto-turns on HUD (Tactical) and dense detection unless detection was hand-edited this session |
| 3 | same | Look: NVG | same as 2 |
| 4 | same | Look: FLIR | same as 2 |
| 5 | same | Look: Anime | |
| 6 | same | Look: Noir | |
| 7 | same | Look: Snow | |
| H | same | Toggle HUD on/off | Shows/hides the HUD layout row in Display; claims the "visual" restore lane so a pending share restore will not overwrite it |
| O | same | Toggle slow orbit of the current landmark | Needs a landmark/city flown to first, otherwise a toast "fly to a POI first". Radius = the landmark's framing altitude (500 m if none), pitch = its pitch (-30 deg default), 6 deg per second (about 60 s per revolution) [OBSERVED-IN-SOURCE]. No on-screen control for orbit was found (keyboard, and possibly voice). Auto-stops when a different city or landmark is chosen, on reset-to-globe, and on dispose. An "ORBIT" badge shows while active |
| V | same | Toggle clean view | Escape does **not** exit clean view; use V or the on-screen exit button |
| F | same | Show/hide the whole Data Layers panel | This is visibility, not collapse (the panel ships visible, in its compact collapsed form). Not saved anywhere, not in the link, and no on-screen button exists for it |
| D | same | Toggle detection overlay off/on | Turning back on restores the last non-off density/profile. Marks detection as hand-edited for the session, which stops looks 2-4 from auto-enabling it. (Project readme calls it "detection" key; an older note called it "cycle density" - actual behaviour is on/off) |
| C | Anywhere (not typing) | **Three meanings, resolved in this order:** (1) while in cockpit: exit cockpit; (2) while a civil or military aircraft is tracked: try to enter cockpit; (3) otherwise toggle the CCTV layer on/off (when switched on with no active camera it focuses the nearest camera) | **Primary conflict.** Mechanism [OBSERVED-IN-SOURCE]: the cockpit listener runs first (capture phase); in cases 1-2 it stops the event outright, so the CCTV listener never sees it. Case 2 swallows the key even when cockpit is not allowed (Contacts context not active, or either flight layer off) - nothing happens and CCTV is *not* toggled. A tracked satellite or vessel does not trigger case 2. Cockpit's handler ignores Ctrl/Cmd/Alt chords and so passes them on; the CCTV handler has no such check, so Cmd/Ctrl+C with a tracked aircraft *does* toggle CCTV |
| Backtick | Anywhere (not in a text/select/editable field; Ctrl, Alt, Meta and Shift all ignored; key-repeat and IME composition ignored; already-handled presses ignored) | Toggle the FPS readout | Hidden at start of every session. The only page-wide shortcut with full modifier/repeat hygiene |
| Q W E R T | **Only while a city's landmark row is open** and focus is outside text fields and selectors | Fly to landmark 1-5 of that city | Conditional, not global: with the tray closed or no city expanded they do nothing; ignored when typing; no modifier check (Cmd+W / Ctrl+T / Cmd+Q chords, which close tabs or quit the browser, also fly the camera if the browser lets the page see them). No other page-wide letter uses these keys today, so there is no live collision; any future letter shortcut would collide |
| Space (tap) | Focused control | Native activation on release | Tap = activate; hold changes meaning |
| Space (hold >= 500 ms) | Anywhere (map, background or over a focused control; not in text entry - the protected set is wider than for the letter keys: text, search, email, url, phone, password, number, date/time inputs, multiline fields, editable regions and ARIA textbox/searchbox/spinbutton roles; Shift/Ctrl/Alt/Meta+Space ignored) | Push-to-talk: voice session starts or the mic un-mutes; release sends | Held press on a focused control blurs it first so the release cannot activate it. Does not claim a click-started (open-mic) session. A hold while the assistant speaks interrupts it. Radio is paused while voice listens. Cancelled if focus moves, the window blurs or the tab hides. Background Space is always reserved immediately to stop page scroll |
| Escape | See A.3 | Closes/cancels the nearest thing | Overloaded; see ordering |
| Tab / Shift+Tab | Open first-run card, provider settings, expanded Street Level viewer, cockpit-free pages | Cycle focus inside the dialog | These surfaces trap Tab; the rest of the page does not become inert (not true modals) |
| Enter | Disclosure buttons (dock launchers) | Opens/closes immediately and moves focus into the tray | Space on the same button acts on release (native) |
| Enter | Draw mode (canvas/body/label field focus) | Finish shape | A focused button keeps its own Enter |
| Enter | Credits button / its Close | Activate immediately | Space activates on release; a long Space hold hands over to push-to-talk and cancels the pending activation |

### A.3 Escape - complete ordered behaviour

Escape means "back out one level". There is no single dispatcher: many independent listeners exist, and the result depends on *where* each listens and whether it stops the event. Re-read from the listener wiring, the real priority is three tiers [OBSERVED-IN-SOURCE unless tagged]:

**Tier 1 - page-level capture listeners (run first, before anything below; each one that acts consumes the key so lower tiers never see it).** Among themselves, ties are broken by the order the surfaces were wired at start-up, which is UNVERIFIED; in practice these surfaces are rarely active together.
1. First-run card: dismisses for the session. Active only while the card is shown and is the topmost thing at its own position (something painted over it, such as the credits lightbox, makes it ignore the key); it also steps aside for cockpit, scene playback, recording and clean view.
2. Provider settings dialog: closes it and returns focus to its chip. Same shared dialog helper as the first-run card.
3. Expanded Street Level viewer: shrinks it and docks it. Only when the key comes from inside the viewer, from the page background, or from the globe; a field elsewhere (such as search) keeps its Escape. Stops the key before the map's sequence-clearing handler.
4. Cockpit: in order - if the key came from the credits lightbox, do nothing (the lightbox owns it); if the compact-radio disclosure or a cockpit utility pop-over is open, do nothing (their own capture listeners close them); else collapse the Contact rail if focus is inside it; else collapse the Live Signals rail if focus is inside it; else exit cockpit and restore the tracked entity and follow camera. Ignores key-repeat.
5. Draw mode (while on): cancels the shape in progress; with nothing in progress, leaves draw mode. Ignored if focus is in any text, number or slider-type input *other than* its own label field, or in an editable region - in which case the key falls to lower tiers (for example, with a Display slider focused, Escape collapses Display instead of cancelling the shape).
6. Imagery box-selection tool (while armed): cancels it; if the Recent Imagery panel claims the key first (it exits a live comparison), the tool stays armed.
7. Compact radio disclosure in Context, and the two cockpit pop-overs (Display, Radio): close the open one, return focus unless Escape came from its own toggle, and stop all later listeners so the same press does not also dismiss the first-run card.

**Tier 2 - listeners on a specific element (bubble phase; only when focus is inside that element; each consumes the key).**
8. Credits lightbox: closes it and returns focus to the attribution button (cockpit stays active).
9. Inline edit fields: CCTV calibration number field (cancels), scene shot rename field (cancels), radio tuner drag (cancels, snaps back), Recent Imagery strip (cancels preview/box tool; the panel's own Escape then sees the key as already handled and does not collapse).
10. Panels: the nearest expanded panel containing the focused element collapses and focus returns to its toggle (cleared if Escape came from the toggle). Nested Parameters panel first, then its parent. Dock trays also clear an unfinished location search. Because this is element-level and consumes the key, **a panel-focused Escape never also deselects a tracked object** [FROM-TESTS].

**Tier 3 - page-level bubble listeners (every one runs independently when the key reaches the page; none checks whether another has acted).** Reached only when focus is on the page background, the globe, or a non-panel control and tiers 1-2 did not consume the key:
11. Location search field: collapses and clears (only if expanded).
12. Scene playback: stops it ("Stopped (Esc)").
13. Directions layer: disarms placement mode, else clears the selected step.
14. Street Level: clears the selected image sequence, skipping when the target is inside a panel, a dialog or a text field, or the key was already handled.
15. Tracking/selection: deselects the tracked civil flight, military flight, satellite, vessel, transit vehicle or bike-share station - each layer has its own listener (the vessel, transit and bike-share ones are missing from older notes).

**Practical consequences.**
- On the map (focus on page body), one Escape press can *simultaneously* deselect a tracked flight, clear a Directions step, clear a Street Level sequence, stop a running scene and dismiss the search field - all of tier 3 fire together. Pairs that co-fire are by construction (no layer stops propagation); real-browser confirmation is still UNVERIFIED.
- Escape never exits clean view or recording presentation (use V or the on-screen exit button).
- Escape does not close the voice help/error trays [OBSERVED: no handler found].
- Priority summary for a rebuild: modal surfaces and active tools first, then the focused widget, then ambient map selections - and a rebuild should make tier 3 one ordered chain that stops at the first thing it closes.

### A.4 Widget-level keys (only when that widget has focus)

| Widget | Keys | Effect |
|---|---|---|
| Context mode tabs (Contacts / Space Missions) | Left, Right, Home, End | Move between tabs |
| Space Missions roster items | Focus (Tab) previews; Enter or Space selects | Preview = temporary globe rotation and marker highlight, same as hover; keyboard and pointer previews are independent |
| Contacts list buttons | Enter / Space | Focus that object; list refresh preserves focus by identity |
| Radio tuner dial | Left/Right (+-1 station), PageUp/PageDown (~10 % of list), Home/End; Escape cancels | Key release commits the tune and flies to the broadcaster |
| Radio, SDR and compact radio | Enter on the frequency field | Tune |
| Street Level segmented controls | Arrow keys (wrap), Home, End | Roving selection |
| Street Level KartaView viewer | Left/Right turn or step, Up/Down pitch, +/- zoom | 360 photo: look around; flat: step |
| Recent Imagery strip (day cards) | Left/Right, Home, End move focus and preview; Enter previews; **S** (single mode) or **A** pins focused day to slot A, **B** to slot B; Escape cancels | Key repeat ignored for pinning |
| Recent Imagery mode radio | Arrows (wrap) | Image / vs Basemap / A-B |
| Comparison divider | Left/Right, Home, End | Move the swipe line |
| Sliders (Display, Parameters, timeline, volume) | Arrow keys native | Value changes apply live |
| Weather timeline | Slider arrows + earlier/later/Latest/Play buttons | Preview then commit after 150 ms |
| Location search field | Enter submits; Escape clears | |
| Draw tool | Enter finish, Backspace remove last point (not inside the label field), Escape cancel/leave | Live only while draw is on and never when typing in another field |
| CCTV calibration chip field | Enter commit, Escape cancel, blur commits | Key events are stopped so global shortcuts cannot fire |
| Scene shot title | Double-click to edit; Enter save, Escape cancel | |
| Scene action buttons | Tab, Enter/Space | Authored buttons only |
| Cockpit briefing | Tab continues from briefing footer to Display and Radio launchers | Focus is not stolen on refresh; if the focused signal disappears, focus moves once to the page tab |
| Map source chips | Enter on tray opener focuses the active chip (Space opens on release) | Retries briefly during tray fade |

### A.4b There is no keyboard zoom or pan

The complete page-wide shortcut set is: 1 to 7 (looks), H, O, V, F, D, C, backtick, Q W E R T (conditional), Space-hold and Escape. **No key zooms, pans, tilts or rotates the globe.** Arrow keys and plus/minus act only inside the street-level photo viewer (see A.4). Camera movement by keyboard-free means is limited to mouse, trackpad and touch (Part B), the on-screen tilt, north-up and reset buttons, landmark flights, and voice (relative zoom: little 25 %, medium 55 %, a lot 100 % of the distance to the view target, with minimum steps of 20 m in and 50 m out and never closer than 25 m to the target; camera verbs for pan, tilt, rotate and orbit). Consequence: the globe is not operable by keyboard alone, an accessibility gap. [OBSERVED-IN-SOURCE; confidence high.] Detail: 13a NAV-04, NAV-05, NAV-29, G6, G8.

**Kawn note (LATER, accessibility).** Add keyboard camera control: arrow keys pan, plus/minus (and the physical equivalent keys) zoom, with modifier-free handling that respects typing protection, key repeat for smooth movement, and an on-screen help sheet. Use physical-key codes so Arabic layouts work.

### A.5 Shortcut documentation gap
Key hints are shown on look tiles (1-7), landmark pills (Q-T), in tooltips for HUD (H) and Detect (D), and in the cockpit help line ("ESC exit, C toggle"). There is no shortcuts overlay or help page; O, V, F, backtick and Space-hold are discoverable only through the README, the voice help tray (Space) or trial.

---

## Part B - Mouse, trackpad and touch gestures

Engine defaults are used for camera navigation; the app does not remap them except for trackpad pinch. Where a mapping comes from the 3D engine's defaults rather than app code it is marked UNVERIFIED.

| Gesture | Context | Effect |
|---|---|---|
| Left-button drag | Globe | Orbit/pan the globe (rotate about the earth or look) - engine default, UNVERIFIED |
| Right-button drag / wheel | Globe | Zoom - engine default, UNVERIFIED |
| Middle-button drag (or modifier + drag) | Globe | Tilt - engine default, UNVERIFIED |
| Mouse wheel | Globe | Smooth zoom toward the cursor. The app sets no global minimum or maximum zoom distance (engine default; about 100 m near, astronomical far). At street level tile loading can lag (project notes advise stopping around 500 m). Zooming far out has no clamp with synthetic wheel events. While something is tracked, the minimum camera-to-target distance is 150 m and zoom inertia is off, and a per-frame clamp keeps the camera on the same side of the target |
| Trackpad pinch | Globe | Browsers report pinch as a Ctrl-modified wheel; the app adds a Ctrl+wheel zoom binding (unless the engine already binds one), intercepts those events in the capture phase, multiplies the pixel delta by 8, clamps it to 120 px per event and re-sends a synthetic wheel event on the canvas marked so it is not relayed twice. Events not in pixel mode, with zero delta, or already relayed are ignored. The relay is removed cleanly on teardown. A real Ctrl+mouse-wheel in pixel mode is amplified the same way (INFERRED); ordinary wheels and native touch pinch are unchanged [OBSERVED-IN-SOURCE; confidence high] |
| Any automatic camera move in progress | Globe | A drag, wheel or pinch cancels it (flights, ground-guard lift, orbit and camera verbs also stop on any new navigation) |
| Touch: one-finger drag, two-finger pinch/twist/drag | Globe | Pan, zoom, rotate/tilt - engine defaults, UNVERIFIED |
| Click on a trackable object | Globe | Selects it (travel up to 6 px counts as a click even for a long press); camera locks on, trail draws, card/tooltip opens |
| Click on empty globe | Globe | Deselects the tracked entity only if it was a clean click: travel at most 6 px **and** press at most 400 ms. Ignored in cockpit, during draw, during other pointer-owning tools |
| Click on a coverage line / cone | Street Level | Opens nearest photo / selects cone; Street View opens nearest panorama on click of the map below ~3 km |
| Hover | CCTV markers, launches, street-level lines, dock trays, Space Missions roster | Throttled hover highlight; dock trays open after 140 ms and close 420 ms after leaving (mouse/pen only) |
| Wheel over a collapsed dock tray | Dock | Cancels a pending hover-open for 280 ms (so scrolling past does not pop it open) |
| Click on a collapsed panel body | Rails | Expands it |
| Drag panel header | CCTV, Street Level | Lifts the panel into a floating window (mouse/pen only; touch scrolls the rail instead); 4 px threshold |
| Drag window edge | Floating panels | Resize (min sizes 300x160 CCTV, 320x280 Street Level) |
| Double-press header (400 ms, 6 px) | Floating panels | Snap back into rail |
| Double-click | Draw mode | Finish shape. While draw mode or the imagery box tool is active the engine's stock double-click action is removed; otherwise the stock action stays in place (resolved: not remapped) |
| Double-click | Scene shot title | Rename |
| Drag on globe | Imagery box tool, CCTV ADJUST gizmo (rings rotate, arrows move, handles resize range/field of view), radio tuner needle, comparison divider | Tool-specific; each claims exclusive pointer ownership while active so aircraft, vessels, satellites, cameras and other layers do not select underneath |
| Drag-and-drop files | Anywhere on the app | Imports GeoJSON/KML/KMZ/GPX |
| Press and hold mic / Space | Voice | See push-to-talk; mic click = open-mic toggle |
| Window pointer-down or wheel | During initial share restore | Cancels the delayed shared-camera flight and shared "follow" so a recipient is never yanked from where they are steering |

Pointer ownership rule: only one tool at a time holds the pointer (draw, box select, calibration, etc.); a second claim is refused and the hint names the holder.

---

## Part C - Shareable state

### C.1 How the link works (behaviour)
- The address bar is the source of truth. Whenever the camera, look, panels or layers change, the address fragment is rewritten (replace, not push - so Back does not step through states) after a **500 ms debounce**. Writes are suppressed until the initial restore has completely settled.
- **Copy share link** copies the full current address and adds the copy time (so a recipient's app can classify the link's age); the live address itself is not aged.
- Opening a link with a valid camera position triggers a restore. The camera flight starts **1.5 s after load**, takes **3 s** (ease in-out) and ends by snapping exactly to the stored pose. If the recipient drags or scrolls the globe before it starts, the camera and follow portions are cancelled while layers and visuals still restore. A top-centre "restoring shared view" message holds the startup screen until every part settles.
- **Editing the fragment after the page has loaded does not re-apply it**; no listener for fragment or history changes exists, so a full reload is needed [OBSERVED-IN-SOURCE].
- Only the camera latitude/longitude make a link "valid". The layer list, panel states and shader values are read only when the link carries the current version marker; older links restore camera, look, bloom, sharpen, HUD, detection, scope and map only. A minimal link with just a position reads as: HUD off, detection off, sharpen off, bloom off, scope on, Google 3D map, altitude 800 m, pitch -35 [OBSERVED-IN-SOURCE].
- Any valid link also makes the recipient's stored panel open/closed choices be ignored for that load (the link's panel field, or defaults, applies) and a link without a layer field leaves layers at defaults rather than the recipient's saved set.
- Gesture cancel is triggered only by a pointer press or wheel on the globe; a key press does not cancel the restore flight.
- A link that carries a camera **overrides the recipient's saved layer preferences for this load and does not overwrite them**; a link without the newer layer field uses the old default-layers behaviour and ignores local saved layers. An explicitly empty layer field means "no layers".
- Layer restoration is independent per layer: one failing source cannot block the others. A newer explicit user action supersedes only the lane it owns (layers, options, camera, look, map, each panel).
- The first-run launcher never shows when a share link is present.

### C.1b Write suppression, lanes and the restore flight (census merge)

- **Hash writes are suppressed while the initial restore is pending.** Without this, a recipient's first moves (or the app's own start-up camera) would overwrite the incoming link before it had been applied. The suppression lifts only when every part of the restore has settled, including the map source (which is awaited before the final share-state write). [OBSERVED-IN-SOURCE; confidence high.] Detail: 13a G11, NAV-36, NAV-37.
- **Validity.** Only finite latitude and longitude make a link valid; otherwise it is ignored entirely. Missing altitude reads 800 m, heading 0, pitch -35 degrees and roll 0.
- **Lanes.** Restoration runs in separate lanes (visual, map, layers, each panel, camera, tracked subject). A later explicit user action claims its lane and wins over a delayed restore; this is why H, the look keys and clicking a panel open all outrank a pending link. A recipient's own gesture beats the link's camera.
- **Restore flight.** After the 1.5 s delay the camera flies 3 s, then re-applies the exact stored pose *only if the restore still owns the camera*; a user gesture before that cancels the camera portion (not the visual or layer portions). If the manager is destroyed during its own flight, the owned navigation is cancelled only if it is still the current one.
- **Map source in the link.** Stored as a short identifier, restored after the camera, and a newer explicit map choice claims the map lane; when absent the default is the photoreal source (falls back automatically if unavailable, for example no Google key).
- **Copy link.** The snapshot is built without touching browser history, the copy-time stamp is added to the copy only, and the clipboard write reports success or failure by toast. The stamp lets a recipient classify staleness (C.4).
- **Version marker.** A version marker on the link gates restoring layers, panels and shader values; links without it restore the camera, look, bloom, sharpen, HUD, detection, scope and map only.

**Kawn note.** Keep the write-suppression rule exactly; the Kawn default (Dubai, Scope on, feather 100 %) is written explicitly into every generated link, while links that omit the field keep reading as their era's value (C.3 last paragraph).

### C.2 Facets serialised into the link

**Camera**
1. Position (latitude, longitude to 4 decimals) and altitude (rounded metres)
2. Heading, pitch and roll (rounded degrees)

**Visual**
3. Active look (normal, CRT, NVG, FLIR, anime, noir, snow)
4. Look-specific shader values: per look, an allow-listed set (retro: pixelation, distortion, instability; NVG: gain, bloom, scanlines, pixelation; FLIR: sensitivity, bloom, white/black-hot mode, pixelation, Ironbow palette; anime: saturation, edge thickness; noir: contrast, grain, vignette; snow: density, wind). Values are clamped to the look's range and stored at 0.01 resolution. Only the active look's values travel.
5. Bloom on/off and intensity (with a scale-version marker so old links still mean the same brightness)
6. Sharpen on/off and intensity
7. HUD visibility and layout
8. Detection mode, density, allocation strategy, label fade distance, outside-opacity
9. Celestial ring on/off
10. Scope on/off, feather percentage, and an optional pinned outside-fill opacity (written only when pinned; absence means the altitude-adaptive default)
11. Map source (Google 3D, Bing aerial, Bing labels, Esri satellite, OSM)

**Layers (enabled set + selected options)**
12. Which layers are on, in a fixed canonical order (explicit empty = none). The set covers every layer in the panel except Local ADS-B (not serialisable) and the event-specific demo layers. A layer tag is a permanent public identifier that is never reused, so old links keep working; tags are one character first, then two-character base-36.
13. Per-layer options (allow-listed, a short assignment each; defaults omitted):
   - Civil/military flights: 3D models on/off and mode (proximity/all); selected civil flight identity; selected military flight identity (the 3D default was flipped to ON later, so "ON" is now written explicitly while an omitted value is read as OFF for old links)
   - Satellites: catalogue (core/dense) and selected satellite identity
   - CCTV: coverage mode (off/on/viewshed), projection on/off, auto-hop on/off
   - Street traffic: road source (TomTom/OSM/hybrid)
   - Wind: model (GFS/ECMWF), field overlay (none/speed/temperature/pressure), units (km/h, m/s, mph), paused
   - Rain radar and lightning: opacity (light/strong)
   - Satellite clouds: infrared filtered/full, opacity, product (global/regional)
   - Recent Imagery: the box (four edges, integer 1e-5 degrees, rejected rather than clamped outside bounds), the A and B pinned days (product + calendar date, validated both ways), mode (image / vs basemap / A-B), the swipe-divider position (**link-only**), VIIRS source toggle
   - Street Level: Mapillary on/off, other providers' switches by stable id, panorama filter (all/360/flat), "since" days (0 up to 36,500)
   - Radio: station-tag filter and volume (never auto-selects or plays a station)
   Enabled-only (no options): vessels, ALPR cameras, bike share, directions, earthquakes, fire perimeters, GNSS integrity, dams, data centres, active fires, awareness (Contacts) layer, installations, launches, cables, transit, cyclones, event demo layers.
14. **Tracked subject**: at most one of {civil flight, military flight, satellite}; stored as the object's transponder/catalogue identity (bounded grammar - 16 chars alphanumerics plus a few separators; anything outside the grammar is rejected, never truncated). It is a *hand-off*, not a bookmark: see C.4.

**Panels**
15. Collapsed/expanded state for each known panel: preset tray, location tray, data layers, CCTV, recent imagery, street level, radio, scenes, global context, display, parameters. Plus **pinned** state for the two dock trays. Automatic (layout) collapse is never recorded; a recipient's malformed or partial panel field cannot overwrite their own layout.

**Annotations** (read-only in the live link): a link may carry up to 24 marks in at most 6,000 characters and they are drawn after restore (also when embedded). Kinds: pin, highlight, area, arrow, route, label; colours primary, amber, cyan, green, red. The live address is rebuilt from scratch on every write, so the marks vanish from the address after the first write and Copy share link never includes them [OBSERVED-IN-SOURCE]; they come only from view-builder tools.

### C.3 What is restored vs not

| Restored from a link | Not restored / not shared |
|---|---|
| Camera pose (after 1.5 s delay) | Clean view, recording mode, orbit |
| Look and its allow-listed shader values | Cockpit mode (only the tracked subject, and views from tools may re-enter it) |
| Bloom, sharpen, HUD on/layout, detection settings, scope, celestial | Voice session, radio playback or station selection |
| Map source (falls back if unavailable, e.g. Google 3D without a key) | Scene playback, scene library, run logs |
| Enabled layers and their allow-listed options | CCTV active camera, saved calibration, calibration mode |
| Panel open/closed and pin state | Street Level selected image or sequence, viewer expansion, FOLLOW |
| One tracked subject (with expiry, below) | Floating panel position/size, panel scroll |
| Recent imagery box, pinned days, comparison mode | Weather history time, wind inspection reading, cyclone selection |
| Radio filter and volume | Drawn marks, imported files, annotations created in-session |
| | Sonar settings, Parameters panel open state beyond collapse flag |
| | Voice tier, spend limits, cloud-auth mode |
| | Live data itself, feed health |
| | Local ADS-B layer and tuner state |

Share links are **authored state**: when a default later moves (for example 3D models, scope feather, detection outside opacity) the *reading* of an absent field stays frozen at what it meant when links were authored, and the generator always writes new defaults explicitly. Older links that omit a field therefore reopen as their author saw them.

**Three-surface rule for defaults (census merge).** A first-run default lives in four places that must move together: the engine constant, the markup value and readout, the UI baseline, and the link generator's starting state. The *parse* fallback for a link that omits a field stays at the value of the era in which that link was written (for example Scope feather legacy 35, bloom legacy scale inverted and migrated on every ingestion path: links, scenes, voice). Only the 3D-models default has durable browser storage; detection mode, density and Scope feather persist only in the link, written on the 500 ms debounce, so a new tab or bare address returns to defaults. Detail: 13f GAP-11, SYS-121 to SYS-123, SYS-141, SYS-167.

### C.4 Tracked-subject hand-off and expiry
- The recipient sees a persistent top-centre "ACQUIRING" notice naming the shared subject until the feed row arrives. The app keeps retrying once per second for a window of **90 s (civil flight), 45 s (military flight), 300 s (satellite)** counted from restore start [OBSERVED-IN-SOURCE].
- Only after that window does it give up and clear the subject. The wording depends on the link's age: "follow expired" only when the link carries a copy-time stamp older than the window; otherwise "is unavailable"; "could not be restored - feed unavailable" when the feed itself failed. A link read from the live address bar (no stamp) therefore never says "expired". Failure notices wait until the startup cover has faded, and a share-specific failure arriving while another failure shows is queued for its own full dwell.
- Only a share-link load runs this watch; the camera follow also requires the shared camera flight to have been applied (if the recipient steered first, the follow is cancelled and cleared without touching their saved preferences).
- Only one family can be tracked at once; an incoming link with more than one family fails closed (no tracking). A tracked identity is also stored in the recipient-side persistent layer state when they choose it themselves, but a stored identity is not auto-followed by this watch (INFERRED: the watch is gated to share loads).
- An explicit navigation, a manual gesture, disabling the owner layer or a newer selection cancels the pending hand-off.

### C.5 Link format limits and validation
- Layer field capped at 4,096 characters, option field at 512; oversized, repeated, empty-member, duplicate or unknown layer tokens reject the **whole layer payload** (never salvage a prefix); unknown option tokens are ignored individually; bad panel assignments ignored individually.
- Camera latitude/longitude must be finite numbers or the link is treated as absent. Other numeric fields fall back to their legacy defaults when missing (heading 0, pitch -35, altitude 800 m, scope feather legacy 35, outside opacity legacy 5, bloom legacy intensity 50, detection fade legacy 16).
- Copy-time stamp must be a positive integer second count not in the future to be honoured.

### C.6 Session versus persistent preferences

| Preference | Scope | Notes |
|---|---|---|
| Enabled layers + allow-listed options + tracked subject | **Persistent** (browser storage) | Written only on explicit user/voice/tool choices; passive restores, dependency-held layers and context-owned changes are not saved. Link-only values (swipe position) never stored. On a normal visit this is restored; on a link visit the link wins and storage is untouched |
| Panel collapsed/expanded | Persistent | Excludes automatic layout collapse; dock trays always start compact regardless |
| Floating window geometry (CCTV, Street Level) | Persistent | Versioned; a version bump clears old positions and shows a one-time notice |
| One-time hints (snap-back hint, layout-reset notice) | Persistent flags | |
| First-run "do not show again" | Persistent | The only durable dismissal |
| First-run dismissed (mission chosen, Explore, Escape) | **Session** | Returns next browser session |
| Detection allocation (Elastic/Weighted) | Persistent | Detection mode/density/fade have **no** storage - only the link/address carries them |
| Look, bloom, sharpen, HUD, scope, feather, celestial, map source | **Neither** persistent nor session beyond the address (no browser-storage write found for them) | Survive a same-URL reload only if the 500 ms address write has already happened; a bare URL or new tab returns to defaults |
| Street Level fit/fill | Persistent | |
| CCTV saved calibrations | Persistent in browser storage (explicit SAVE; restored when the layer starts) | Unsaved edits are session-only and shown as "edited" |
| Cockpit weather-effects opt-in | Persistent | Default off |
| SDR device and gain | Persistent | |
| Voice model tier, spend limits, cloud-auth mode, recent voice errors | Persistent | |
| Scene library (director project) + a checkpoint | Persistent | Imported bundle bytes are session-only and must be re-imported after reload |
| Sonar settings | **Session** (reset on reload) | |
| FPS readout, clean view, orbit, drawn marks, imported files | Session (not even stored) | |
| Provider keys | Server-side configuration, never in the browser or link | Browser-exposed keys must be provider-restricted |
| Tracked-subject notices, restore status | Session | |

---

## Part D - Query-level switches (behaviour only, names withheld)

| Switch | Effect |
|---|---|
| Embed switch | Globe only, clean view, no launcher/settings/dock/rails; attribution stays; accepts view requests from a framing page, applied in a fixed order with an acknowledgement to the sender's origin (file 01 section 12). Also available inline when a host page sets a global flag first. Framing by other pages is refused by default; the operator allow-list applies only to embed-mode documents |
| Welcome on / off | Value 1 forces the first-run card on (outranks every suppression, including the "do not show again" flag and a share link); value 0 suppresses it. The card otherwise returns every new browser session and never shows to a share-link recipient or when embedded |
| Setup | Opens the provider-settings dialog even after the chip retired or in a compact layout |
| Debug readouts | Developer-only banners for traffic and detection solver; hidden otherwise |
| Road-source override (traffic) | Forces a road source over restored state until the user chooses explicitly |

---

## Kawn notes

**Keep**
- Hash-based state with debounced replace-state writes and a visible "restoring shared view" state; link wins over local storage for that load without overwriting it; "authored state" rule (absent field means what it meant when authored; defaults written explicitly); permanent never-reused layer tags; fail-closed whole-payload rejection; bounded identity grammar; hand-off (not bookmark) semantics with per-subject expiry; recipient gesture cancels the camera flight; panel state in links excluding auto-collapse.
- Escape = back out one level, with focus-return; typing protection; push-to-talk hold threshold that preserves short taps.

**Adapt**
- Stop using single letters as app-wide shortcuts that do not check modifiers; in an Arabic-first RTL product use physical-key codes (not typed characters) and a visible shortcut sheet; replace C's triple meaning with distinct gestures.
- Make shared state human-meaningful: persona/profile, selected district/project, comparison period, time-of-day slider, language/numeral choice, chosen basemap, enabled thematic layers, selected shortlist (max 4 places). Add a short-lived signed link for sensitive government views and an optional "frozen snapshot vs live" choice.
- Add timestamp-based expiry only for live subjects (a vehicle, a flight to DXB/DWC/AUH/SHJ), not for static places; keep the 90 s style windows configurable per feed.
- Persist look/preferences per profile (not only in the URL) so a bare URL returns the user's choices; keep recipient preferences untouched by links.
- Provide a gesture set for touch (two-finger tilt, long-press to inspect) and make the tap-to-select threshold and deselect rule explicit (6 px / 400 ms is a good start).
- Defaults for Kawn (owner decisions 2026-10-16): Dubai top-down at city scale, Scope ON, feather 100 %. Write these explicitly in every generated link; location-tray flights (emirates, DXB, DWC, AUH, SHJ, LHR and the other London airports) are plain camera states, so a shared link to "Sharjah" is just a camera pose, no special token.
- Keyboard: add camera pan and zoom keys (A.4b), keep typing protection, and add modifier and repeat hygiene to every shortcut; consider applying the world-jump pause by flight distance rather than by pill click.
- URL length: keep the layer set compact (the reference uses a dot-joined token list plus short assignments) and add short-link storage on the server for large states.

**Skip**
- Orbit-as-hotkey for landmarks (keep as UI action), frame-rate hotkey (developer menu only), military/cockpit specifics, SDR keys, TR-style easter eggs, voice Space-hold unless Arabic input method testing passes.

---

## Open questions / UNVERIFIED

- Exact engine-default mouse and touch mappings (rotate/zoom/tilt buttons) - taken as library defaults, not tested.
- Whether modified chords actually reach the page in each browser (the page-level code has no modifier check and a test pins that; browsers may swallow some chords first).
- Co-firing of multiple Escape handlers in map mode with a tracked object and an open panel.
- Whether numpad digits and non-Latin keyboard layouts trigger shortcuts (typed-character based).
- Exact local-storage lifetime/cleanup rules and quota behaviour.
- Which map-stack fallbacks are applied on restore when a linked source is unavailable (documented in general terms only).
- Mapillary photo viewer's own keyboard bindings (third-party viewer defaults).
- Tracking identities for vessels, fires and installations are not shareable (only flights and satellites) - confirmed from the facet list but not live-tested.
- Whether the engine's default mouse mapping is exactly as assumed in Part B (13a section 5); whether any code uses the sun position for shading (none found).
- Real-device touch behaviour of push-to-talk and the 500 ms hold on iOS Safari and Android Chrome (13f).

**Resolved in this revision:** annotation marks never copied; fragment edits never re-applied; stock double-click left in place; Display/Escape priority tiers; C-key mechanism; modifier/repeat policy confirmed by tests.

## Revision log

- Rewrote the Escape section as three tiers (capture-phase surfaces, element-level handlers, page-level independent handlers) and added the missing vessel, transit and bike-share selection handlers; documented that a panel-focused Escape never co-fires with map deselection while map-focused Escape fires all tier-3 handlers together.
- Corrected the shortcut rules: no modifier and no key-repeat protection on page-wide keys (test-pinned), typing protection is by element type only, landmark keys inherit the same gaps, C has three meanings with the capture-phase mechanism explained, push-to-talk ignores modifiers.
- Corrected tracked-subject expiry (retry window versus link-age wording), added link validity rules, minimal-link reading, stored-panel override, and gesture-cancel scope.
- Resolved annotation copy question, fragment re-apply question, double-click question; added pinch amplification numbers.
- Added tags and a list of still-open items.

### Pass 3 (census merge)

Sources: 13a (navigation, zoom, share-link restore) and 13f (first-run, keys, embed, status).
- Corrections applied: Q W E R T stated as conditional on an open landmark row; explicit statement that no key zooms or pans the globe; welcome switch values 0 and 1 spelled out (1 outranks every suppression); embed switch details.
- Added: new A.4b (no keyboard zoom or pan, voice zoom fractions as the only non-pointer zoom, Kawn LATER proposal); trackpad pinch relay edge cases and wheel zoom limits including the 150 m tracked-target minimum; new C.1b (write suppression during restore, validity, lanes, restore flight ownership, map source in links, copy-link behaviour, version marker); three-surface rule for defaults and the persistence rule for detection and feather; Kawn notes for Dubai default, Scope feather 100 %, explicit defaults in generated links, location tray flights as plain camera states.
- Confidence tags set on touched text; pointer to the census: mechanism detail in 13a section 2.4 and G8, G11, and 13f GAP-11.

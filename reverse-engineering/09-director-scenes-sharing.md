# 09 - Director, Scenes and Sharing (behavioural reverse-engineering)

**Purpose.** The reference product contains a "Director": a scene-authoring and playback system that turns the live globe into repeatable cinematic tours. A *scene* is an ordered list of *shots*; each shot says where the camera goes, how long the flight and the pause last, which visual style and display settings apply, and which data layers are on. Newer document versions add geographic anchors, explicit start-to-end camera moves with easing, bounded data packs (map shapes, georeferenced images, media markers), declarative click interactions, import preview, and shareable scene files or asset bundles. This document records the full observable behaviour - authoring controls, playback phases, seek/scrub, document model, validation limits, sharing flows, bundled scenes and failure behaviour - so a clean-room team can rebuild an equivalent. Nothing here reproduces code, file names or identifiers; field concepts are described in words and numeric limits are given exactly.

Confidence tags: [OBSERVED-IN-SOURCE] read in the reference source; [FROM-TESTS] stated by its automated tests; [FROM-DOCS-ONLY]; [INFERRED]. Numeric limits and bundled-scene data in this document were re-checked against source in the 2026 revision pass. Pass 3 merged the independent feature census (13e, scene playback section and gaps G2, G3, G4, G6, G7) and tags census-derived mechanism text [CENSUS]; it is source-level, no live run. Mechanism overview of playback: 13e, "How scene (Director) playback works".

---

## 1. User-facing surfaces

### 1.1 The Scenes panel
A collapsible panel (hidden in the Operator HUD layout; visible in the Tactical/Cyber layouts; also openable by voice). Contents, top to bottom:
1. **Scene selector** (drop-down of scenes in the current project), **NEW** (asks for a name, default "Scene N"), **DEL** (confirms "Delete scene ... and all shots?").
2. **CAPTURE SHOT** and **UPDATE SHOT**.
3. **Shot list**: one row per shot with its title (double-click to rename inline; Enter or leaving the field saves a non-empty name; Escape cancels), a **LOAD** and a **DEL** (confirms) button, and a meta line of the form style - detection mode - flight seconds + hold seconds (e.g. "NORMAL - OFF - 4.0s + 0.9s"). Clicking a row selects it; the highlight updates without rebuilding the row so native rename still works.
4. **START**, **STOP**, **NEXT**. [CENSUS] START does not play only the selected scene: it queues the selected scene's shots, then every other scene in project order, wrapping round to the earlier ones; the queue is fixed at the start. In a fresh project, START on the first scene therefore plays all six built-ins back to back. Only voice "play" runs a single scene. NEXT loads the next shot of that same queue (wrapping) without running. Kawn should expose "play this scene" and "play all" as separate controls (see Kawn notes).
5. **EXPORT PRESETS** (whole project), **IMPORT**, **RUN LOG**.
6. Authoring/sharing row (after a scene/shot is selected): **EDIT DETAILS**, **SHARE SCENE**.
7. **Progress bar** with percentage, and a **status line** (e.g. "Ready", "Captured: Scene / Shot", "Updated: ...", "Stopped (Esc)", errors).
8. A separate **scene runtime** readout (also surfaced in the Context area) showing current scene/shot and timing during playback.

While a run is active, all editing buttons (START, NEXT, selector, NEW, DEL, CAPTURE, UPDATE, EXPORT, IMPORT) are disabled and STOP is enabled. RUN LOG is disabled until at least one run has completed.

### 1.2 Playback presentation ("recording preview")
Starting a run puts the app in a clean presentation mode: panels are hidden, the HUD is forced to the full (tactical) layout, a **safe-frame overlay** shows the intended crop, and the page marks itself as playing a scene. Exiting restores the HUD mode and layout the user had. [CENSUS] **Correction: the frame is always 16:9.** A preview run always requests the 16:9 frame with the full HUD; the 9:16 option exists in the overlay but nothing in the reference ever asks for it, and a scene recipe's own panel-hiding, HUD-mode and frame fields are not used for this chrome (only the recipe's HUD mode is converted into each shot's HUD visual setting, which then overrides the "full" request shot by shot). Only scene preview runs turn the recording state on - no voice or automation tool does - and the share link does not carry recording state (the flag appears only in the control-state readback). The HUD mode before the run is captured once and restored on exit; the recording chrome keeps the scene-runtime readout and the credits line visible.

**Safe-frame geometry** [CENSUS]: a full-screen, non-interactive layer above the map and below popovers; a 1 px outline at about 28 percent white with a 1 px dark halo; thirds guide lines at about 20 percent white. The 16:9 box width is the smaller of 90 percent of the viewport width and 90 percent of the viewport height times 16/9; the 9:16 box width is the smaller of 58 percent of the viewport height and 76 percent of the viewport width. The HUD button state follows the recording state. Look on non-16:9 screens is geometry only, not observed. This is a presentation aid for screen capture - the product does **not** itself record video. A separate user-controlled "Clean UI" mode hides chrome while keeping the required map/data attribution visible.

Keyboard: while a run is active, **Escape** stops it (status "Stopped (Esc)"). There are no other global scene shortcuts. Voice can list, play (single scene), stop, advance and report status.

### 1.3 Capture and edit
- **CAPTURE SHOT**: appends a new shot to the selected scene with the live camera (latitude, longitude, altitude above the ellipsoid, heading, pitch, roll), the live visual state, and the state of **every registered layer** (on/off plus layer parameters). Defaults: title "Shot N", flight 4 s, hold 0.9 s. Fails with "camera not ready" if the camera is not available.
- **UPDATE SHOT**: overwrites the selected shot's camera, visual state and layer states with the current viewport (if the shot has an authored move, the new camera becomes the destination and keeps the ellipsoid height reference; the move's start and easing are preserved; a shared anchor is never changed).
- **LOAD** (per shot): applies that shot alone - visual state, layer on/off reconciliation, camera flight - without starting a run. Newest LOAD wins if several are clicked quickly; blocked while a run is active.
- **NEXT**: loads the next shot in the queue (wraps around) without running.
- **NEW / DEL** scene and shot; deletion selects the first remaining shot.
- Layer states stored in a captured shot include selection parameters for tracked contacts; **playback never re-establishes tracking** (those keys are dropped on the way to the layer) because the follow camera would fight the shot's flight. A selected CCTV camera is kept.

### 1.4 Run log
RUN LOG downloads a small JSON file describing the most recent completed run: start time, estimated duration, number of scenes/shots run and a timestamped list of events (run start, each shot start/complete, errors). It stays on the user's machine.

---

## 2. What a shot contains (document concepts)

| Concept | Meaning | Notes |
|---|---|---|
| Identity | Stable id and title | Missing ids are generated once and persist on next save/export. |
| Camera destination | Either an inline pose (latitude, longitude, height above WGS84 ellipsoid in metres, heading, pitch, roll in degrees) or a reference to a scene-local named anchor plus orientation | Omitted orientation defaults to heading 0, pitch -35, roll 0. A reference cannot also carry inline coordinates and references do not chain. |
| Authored move (optional) | A start pose (inline or anchor) plus an easing choice (linear or cubic ease-in-out) | Requires flight duration and hold. Without a move the shot uses an ordinary camera flight even in a newer-format file. |
| Flight duration | Seconds for the camera travel | Minimum 0.2 s; a zero falls back to 4 s; maximum 86,400 s. |
| Hold | Seconds paused after arrival | 0 allowed; maximum 86,400 s. Bundled content with media can extend holds at runtime (without rewriting the saved value). |
| Visual state | Style (normal, CRT/retro, night-vision/surveillance, thermal, anime, noir, snow), per-style parameters, basemap choice, bloom (on/off, intensity, scale version), sharpen (on/off, intensity), HUD (visible, layout), detection (mode, density, allocation, fade percent, outside-opacity percent), scope keyhole (on/off, feather percent) | Scope and detection edits were once lost on save and now survive import/capture. |
| Layer states | Per layer: on/off and optional parameters (a plain boolean is accepted as shorthand) | Interpreted only by already-registered layers; importing a file never fetches URLs, registers modules or evaluates code. |
| Data pack selection | List of pack ids active for this shot | A shot with none clears the previous shot's packs. |
| Interactions | Declarative click actions (section 6) | Up to 64 per shot. |
| Pack provenance | Source pack id and version on shots copied from a bundled pack | Lets bundled-pack updates and renames work. |

Scene-level fields: id, title, ordered shots, list of layer ids to release when the scene ends, applied-pack markers (pack id, optional version, mapping of original shot title to saved shot id), anchors, data packs. Project-level: version, created/updated timestamps, ids of installed built-in scenes, scenes list (may be empty).

### 2.1 Anchors
A scene-local geographic position with id, optional title, latitude, longitude, height, and an explicit statement that the height is above the ellipsoid. At most 1,024 anchors per scene; ids unique within a scene; a reference to a missing anchor fails validation before anything is replaced. One anchor can serve several shots with different headings/pitches.

### 2.2 Heights
Only ellipsoid heights are supported. Terrain-relative and mean-sea-level inputs are **rejected**. There is no terrain follow and no collision guarantee - authors must leave clearance themselves.

---

## 3. Camera moves, timing and playback

### 3.1 Ordinary shot flight
Smooth eased flight to the destination, using the app's normal camera flight. Cubic easing is used for pose-to-pose sampling in seek. [CENSUS] The live flight carries a safety timeout 0.6 s longer than the flight, and its cancellation signal is handed to the in-flight flight so Stop is immediate. **Live versus seek differ for ordinary shots:** the engine's own flight can arc upward on long hops, whereas seek and scrub use a straight eased interpolation of position, so the in-between views differ while the endpoints agree. Only authored moves share one sampler between live playback and seek (see 3.2 and 3.6).

### 3.2 Explicit move
During an explicit move, latitude and height interpolate linearly between endpoints with the chosen easing; longitude, heading and roll take the **shortest signed arc** (an exact 180-degree tie takes the negative arc; longitude wraps cleanly across the dateline); pitch interpolates directly. Endpoints are exact, including zero pitch. It is plain coordinate interpolation - not a geodesic and not a flight with an automatically raised arc. Curved paths and look-at targets are future extensions.
- Load and Replay of a shot with a move start at its authored start pose regardless of the current viewport; so does Play.
- Seeking inside the hold uses the exact destination; backward seeks do not depend on any previous camera position.
- Each active move owns a single animation callback that is revoked on Stop, replacement, import, teardown, a navigation hand-off or manual pointer/wheel input. Navigation also cancels a run in an authored hold so the next shot cannot retake the camera. Camera claims still respect the cockpit's refusal.

### 3.3 Run phases (per shot, in order)
1. Select shot. 2. Apply visual state. 3. Reconcile layers (refused layers are reported). 4. Load the shot's data packs [CENSUS]. 5. Travel (camera). 6. Settle. 7. Hold, then complete the shot. Cancellation is checked after every phase and passed into in-flight work (a camera flight or network request can be interrupted). Final cleanup runs on completion, Stop or failure (the last scene is retained for ordinary playback only when the run asked not to release on finish). [CENSUS] Cleanup revokes timers and animation callbacks, clears data packs and click actions, cancels the camera flight, switches off the scenes' release-list layers and restores the user's HUD mode and layout from before the run.

**No restore pass.** [CENSUS] Nothing restores the layers the user had before the run: layers a shot names stay as the last shot set them, layers a shot never names were never touched, and only the scene's own release list is switched off at the end. The earlier map source and visual look are not described as restored. Captured shots name every registered layer, so a captured scene fully re-dresses the globe; recipe scenes name only four (flights, satellites, earthquakes, traffic). Kawn should offer an explicit "restore my view after the tour" option.

### 3.4 Layer reconciliation rules
- A shot's layer map only governs layers it **names**; undeclared layers are left alone (bundled shots only name a few layers; captured shots name all).
- Layers no longer registered (retired) are skipped.
- If an isolating context mode (the space-missions mode) is active, playback leaves it first, because that mode refuses other layers.
- A scene's release list is applied when playback ends/switches scene.

### 3.5 Camera ownership
Playback claims the camera for the entire run (so the follow camera of a tracked contact stops writing); runs refuse to start if the camera is unavailable (cockpit) with a reason, and refuse to start while another run is active. A voice or navigation action during playback stops it.

### 3.6 Progress, scrubbing and replay
- The progress bar totals each shot's flight plus effective hold. Direct load, forward/backward seek and replay all use the same camera sampler as live playback **for authored moves only**; ordinary shots are flown live by the engine but scrubbed by straight eased interpolation (see 3.1) [CENSUS].
- Bundled event scenes expose scrubbing; seeking reconstructs layer and evidence state for that moment (backward replay trims downstream history).
- **Scene clock** [CENSUS]: a separate clock publishes elapsed time, scene duration and a running/seeking/stopped state to subscribers (the event panel's scrubber uses it), while a different progress timer drives the bar. The total duration counts every shot's flight plus its effective hold, where effective hold is at least that shot's media and path floors. Scrub and LOAD compute which shot, and which flight or hold phase, a given scene time falls in and place camera and layers accordingly; the newest request wins.
- **Media-bounded hold** [CENSUS]: when a layer owns media, the hold waits for the media to finish with a hard cap of 20 s; an overrun stops the scene with an error rather than hanging (fail-closed).
- **PLAY SCENE in the event panel** [CENSUS]: it continues from the current shot to the end of that one scene, does not wrap, and does not turn on the recording presentation (panels stay visible, which keeps the event panel usable). Like any run it claims the camera and refuses to start while another run is active or cockpit owns the camera.
- Stop immediately settles any pending hold and revokes timers; stale clock callbacks cannot affect a replacement run.
- Media waits (bundled evidence clips) are bounded: playback of an embedded clip is waited for up to about 5 seconds to start and has its own stall timeout; if provider playback is suppressed (for example in some embedded shells) a fixed authored dwell (about 7.65 s for the one clip that has it) is used instead; unavailable media can never strand a scene. Autoplay of media requires the user having pressed Play Scene or Play Shot; LOAD, restoration and seek are passive.

### 3.7 Persistence
- The project is saved to the browser's storage after every edit and checkpointed during runs; storage-write failures show an "unsaved" toast.
- If the saved project is unreadable the app uses temporary defaults with a visible warning and **blocks saving over the original bytes** until the user explicitly imports a valid file; export can still preserve temporary edits.
- Built-in scenes are installed into a new project by default and tracked by id so deleted built-ins are not resurrected; additional bundled packs can be appended to a scene and later updated without disturbing user renames.

---

## 4. Data packs

A scene may declare up to **8 packs**; a shot selects which are active. Declarations are validated at import without loading anything. Loading, seeking or playing a shot acquires only its selected packs; leaving releases them.

| Aspect | Rule |
|---|---|
| Manifest | Version 1; id, format, source, attribution, placement; optional exact byte length and lowercase SHA-256 digest (checked before rendering). |
| Source | Names a *trusted application registration* (e.g. an asset directory or the in-memory bundle), never a script or arbitrary URL. A document cannot supply headers, keys, cookies, endpoints or handlers. Paths are relative, with no traversal, query, fragment, URL syntax, percent escapes or empty segments. |
| Attribution | Always required: non-empty text and licence; optional HTTPS link opened with no opener or referrer. Sharing never changes an asset's licence. Attribution cards stay visible with the assets. |
| Map shapes | UTF-8 feature collection (Point, LineString, Polygon incl. holes); each feature needs a unique string id; coordinates are longitude, latitude, ellipsoid height (missing height = 0, may be underground); geometry properties are ignored (no external icons, HTML, styling or scripts). Max 2,000 features and 50,000 positions; feature ids up to 256 characters; a polygon may have at most 128 rings; heights between -12,000 m and 1,000,000,000 m. |
| Image | PNG only, at most 4096 x 4096 pixels (checked before decoding); placed as a horizontal geographic rectangle (west/south/east/north bounds that must increase and may not cross the dateline) at a stated ellipsoid height. |
| Media | MP4/WebM video or MPEG/Ogg/WAV/WebM audio; located by an anchor with ellipsoid height; shown as a labelled marker with an attribution card holding native, user-operated controls. Never autoplays; no time sync with the scene; no third-party embeds; codec support depends on the browser. |
| Size limits | 8 MiB per asset; 32 MiB of selected assets per shot. |
| Loading | Requests carry no credentials or referrer, refuse redirects, bypass caches; a 15-second deadline; late results are discarded even if the adapter ignores abort; any missing, invalid, oversized or hash-mismatched asset fails that shot's pack load, removes partial results and reports a stable error that does not expose a URL. No offline fallback or silent substitute. Cross-origin directories need CORS. |
| Lifecycle | Stop, replacement, next-shot, scene release and teardown cancel pending work, remove entities/cards, stop media and revoke object URLs. Direct LOAD and seek retain packs until the next operation or Stop. |
| Not supported | Transforms, terrain-relative placement, terrain collision avoidance, arbitrary styling, archive extraction. |

---

## 5. Import, preview and sharing

### 5.1 Import (IMPORT button)
Accepts a scene JSON file (any supported older version) or a scene **asset bundle** file. The user sees a **preview** before anything changes:
- lists scenes and shots, data-pack attribution, which sources are configured or unavailable, **missing layer ids**, and bundled bytes;
- makes no asset requests; configured sources are not proof the files exist (ordinary assets are checked on LOAD); bundle preview verifies manifest and byte integrity, while format decoding and geometry limits are still enforced at render;
- **Apply** replaces the whole current project after validation and playback cleanup (export first to keep both); **Cancel** or Escape discards the staged import;
- if the project changes while a preview is open, the draft is invalidated rather than overwritten;
- invalid files leave the project and saved copy intact and report the failing field path;
- if several files are read concurrently only the newest request may replace the project; disposal prevents a pending import from publishing.

### 5.2 Validation (applies to import and storage writes)
| Limit | Value |
|---|---|
| File size | 5 MiB (UTF-8 size checked before JSON parsing, and again by the file importer) |
| Scenes | 256 |
| Total shots | 10,000 |
| Entries per collection | 10,000 |
| Nesting depth | 24 levels |
| Visited values | 200,000 |
| JSON string length | 65,536 characters |
| Field / id length | 256 characters |
| Title length | 4,096 characters |
| Numbers | must be finite |
| Latitude / longitude | +/-90 / +/-180 |
| Height | -12,000 m to 1,000,000,000 m |
| Heading / roll | +/-360 degrees |
| Pitch | +/-90 degrees |
| Durations and holds | at most 86,400 s |
| Prototype-style keys | refused at every depth |
| Unknown versions or unsupported fields | rejected with the field path, before any state changes |
Legacy numeric strings are accepted for numeric fields. Post-processing values are clamped to the UI slider ranges.

### 5.3 Versions and migration
Unversioned files and versions 1-4 migrate on import without turning ordinary shots into authored moves; ids, pack bindings, durations, visuals and attribution survive. Version 4 added anchors and moves; 5 added data packs; 6 added interactions. **Version 6 is the current and export format** [OBSERVED-IN-SOURCE][FROM-TESTS]: the validator accepts versions 1-6 (interactions are only legal from 6, packs from 5, anchors and moves from 4), and the tests assert that an imported older project comes out as version 6. DOC vs CODE: the scene-document reference page still says version 5 is the write format and shows a version-5 sample; the director overview and the status notes say 6. The code and tests win; the reference page is stale. Bloom intensity has its own scale version (older files used an inverted scale and are migrated once; the document version and bloom scale version are independent). Finite heights below 100 m and zero pitch are preserved (they used to fall back to defaults). The storage key keeps an older name for compatibility; its suffix is not the document version.

### 5.4 Edit scene details (basic authoring)
A dialog with validated text fields for: scene anchors and data-pack manifests; the selected shot's camera, optional move, flight, hold, pack ids and interactions. Helpers: **capture camera as anchor** (adds a named position from the live camera), **set move start to current camera** (creates a pose-to-pose move ending at the shot's existing camera), **use ordinary flight** (removes the move). **Apply details** validates the entire project, preserves ids, layer declarations and content notices and keeps the shot selected; **Cancel** changes nothing. It is deliberately a draft editor - no node graph, no path editor.

### 5.5 Share a scene
- **SHARE SCENE** exports only the selected scene as JSON: camera, timing, layers, anchors, manifests, interactions and attribution, but **no asset bytes** (the recipient needs the same source registrations and files). **EXPORT PRESETS** still exports the entire project.
- **Asset bundle**: the user selects the declared pack files (or a folder); folder paths match a manifest path directly or after stripping the chosen root; single filenames must be unambiguous across all packs. Ambiguous or missing matches abort without downloading a partial bundle; previously imported bundle files can be reused. Export never fetches remote files and never copies registered bundled media. Declared integrity values must match the supplied files.
- **Bundle format**: an uncompressed JSON container with a format tag, bundle version 1, the project document (rewritten so pack sources point at the reserved in-memory bundle source, with exact byte length and SHA-256 added) and an array of assets (relative path, MIME type, base64 payload, SHA-256). Rejected: URLs, traversal, duplicate or unreferenced assets, unknown fields, missing packs, hash mismatches.
- **Bundle limits**: 50 MiB total JSON, 64 files, 8 MiB per file, 32 MiB of asset bytes; allowed types JSON/GeoJSON, PNG, MP4/WebM video, MPEG/Ogg/WAV/WebM audio; no executables, headers or credentials.
- Only declared pack files are bundled - not registered recipe media, linked media, map tiles or live data. Source notices and licences stay in the document; bundling a file does not grant redistribution rights.
- **Lifetime**: imported bundle bytes live in memory for the project/session only (Stop keeps them for replay; project replacement or teardown releases them). Only the scene JSON is saved to browser storage, so **a bundle must be re-imported after a page reload**; missing bytes fail explicitly with no network fallback.
- This is file sharing, not a storage service or offline basemap.

---

## 6. Declarative interactions (version 6)

- Attached to a shot, each interaction has an id, a label (up to 256 characters), a **target** (a feature id inside a map-shape pack selected by that shot) and one **action**.
- Actions:

| Action | Fields | Effect |
|---|---|---|
| Card | text (<= 4,096 chars), optional HTTPS source link (no credentials/query/fragment) | Shows plain text (never HTML) and the link. |
| Focus | anchor id | Sets a top-down camera pose at that anchor through normal camera arbitration (respects cockpit refusal). Immediate pose, not a timeline segment. |
| Shot | shot id in the same scene | Loads that shot at its authored start. |
| Layer | layer id, on/off | Requests that state through the normal layer admission path. The layer must have an explicit baseline in the current shot; a destination shot must declare a baseline for every layer the departing shot could change. |

- Targets are only map-shape features (images and media are not targets). Assets load before feature existence is checked; a missing feature refuses the whole action set.
- Actions appear after LOAD or seek finishes; they are **inactive during automatic playback and camera travel**. Clicking a feature selects its actions and moves focus to the first button; the user explicitly activates one (Tab traverses, Enter/Space activates, visible outlines, status announced to assistive technology). No global shortcuts. Drawing mode blocks both picking and execution.
- A chain of user-triggered shot changes is limited to **64 transitions**; an explicit LOAD starts a new chain.
- No automatic branching or loops, no per-frame interaction loop, no scripts/modules/fetch endpoints/headers. Import/Stop/replacement/teardown remove selection, cards and picking handlers and abort pending layer requests. Stop cancels pending transitions, but completed layer changes remain visible. Executing an action never modifies the scene file; LOAD and seek reapply baselines and clear selection.

---

## 7. Bundled scenes

### 7.1 Five generic showcase scenes (about 26-36 s each; 4-5 shots)
[CENSUS] Their recipe fields for hiding panels, HUD mode and frame shape are not used for the recording chrome (see 1.2): playback always uses the 16:9 frame, so the 9:16 entry below is authored intent only and is never applied. Each declares which of four layers (flights, satellites, earthquakes, traffic) are on/off, a style, bloom and sharpen settings, a detection mode, and per-style tuning.

| Scene | Length | Style / HUD / frame | Layers | Camera story |
|---|---|---|---|---|
| Global Flights Radar | ~30 s | CRT, minimal HUD, 16:9, bloom 62, sharpen on, detection off | Flights on | Mid-Atlantic at ~19,000 km, then France ~8,000 km, Japan ~4,200 km, San Francisco ~1,700 km, back out to ~12,000 km. |
| Orbital Watch | ~32 s | Night-vision, full HUD, 16:9, bloom 58, detection sparse | Satellites on | Start ~22,000 km over the Atlantic, Africa ~12,000 km, Tokyo ~6,500 km, Paris ~3,000 km, pull out ~15,000 km. |
| Thermal Threat Board | ~26 s | Thermal, full HUD, 16:9, bloom 72, sharpen on, detection off | Earthquakes on | California ~5,200 km, Japan ~3,100 km, New Zealand ~2,600 km, North Atlantic ~9,000 km. |
| City Overload | ~30 s | Night-vision, minimal HUD, 9:16 vertical in the recipe but **never applied (16:9 frame always)**, bloom 65, detection dense | Flights, satellites, traffic on | New York descending 1,500 -> 550 -> 260 km with heading swings, then Los Angeles ~1,200 km, London ~1,200 km. |
| Omniscience Pullback | ~36 s | CRT with heavy distortion, full HUD, 16:9, bloom 68, detection sparse | Flights, satellites, quakes, traffic on | Tokyo 280 km, 900 km, 3,800 km, then 9,500 km over Asia and finally ~18,000 km over Africa. |

(Camera moves use flight durations of 4-7 s and holds of 0-1 s; the last shot of each scene has hold 0.) Shot counts: Global Flights Radar 5 (flights 6/5/5/5/4 s), Orbital Watch 5 (7/5/5/5/4 s), Thermal Threat Board 4 (5/5/5/4 s), City Overload 5 (5/4/4/5/5 s), Omniscience Pullback 5 (5/4/5/5/6 s). Orbital Watch actually starts over the Gulf of Mexico/Florida side of the Americas (about 28 N, 82 W), not the mid-Atlantic. Camera pitches run from about -26 (low Tokyo start) to -82 (near top-down at 22,000 km). In the build the "dense" detection of City Overload is stored under a legacy name that maps to dense. Fresh projects install these five scenes plus the Nepal scene (six built-ins); the Nepal scene is listed first.

### 7.2 Nepal Flood Incident (event scene)
- [OBSERVED-IN-SOURCE] 25 shots documenting the 26 August 2026 Bhote Koshi outburst flood. A fresh project seeds the scene with only the first three legacy shots (titled generically), and the startup upgrade step is idempotent: it renames and extends them to the full 25 and also upgrades older saved copies, preserving user edits; the scene sits next to (after) a previous flood scene id if present, else after the flights-radar scene. Pack version in the reference: 18.
- All shots use the photoreal 3D basemap when it is genuinely available, else fall back to Esri imagery at playback time (a credential alone is not trusted). Style is normal, bloom 0, sharpen on, detection off, HUD full, panels not hidden, 16:9 safe frame. Scene release list: the event layer and the locator layer.
- Shots 1-8 are the original "context to valley" sweep (flight 4 s each, pitch -90 top-down except the last two); shots 9-25 are the evidence beats (flight 4.2 s except the last at 4 s). Altitudes are metres above the ellipsoid; "min hold" is the runtime floor that media/path timing can force above the authored hold.

| # | Shot | Lat, Lon | Alt (m) | Heading / Pitch | Flight / Hold (s) | Notes |
|---|---|---|---|---|---|---|
| 1 | Global Incident Context | 27.5962, 78.5718 | 15,628,002 | 0 / -90 | 4 / 0.9 | locator: Nepal context |
| 2 | Nepal-Focused Globe Rotation | 28.7248, 81.3226 | 2,068,966 | 0 / -90 | 4 / 0.9 | Nepal context |
| 3 | Regional Approach | 27.0354, 84.6287 | 809,543 | 0 / -90 | 4 / 0.9 | regional locator |
| 4 | Nearby Cities | 27.891, 85.428 | 225,734 | 0 / -90 | 4 / 0.9 | city context |
| 5 | Incident Corridor | 28.0529, 85.2189 | 126,931 | 0 / -90 | 4 / 11 | incident places labelled (minimum 11 s overview hold) |
| 6 | Flood Path | 28.1691, 85.352 | 60,328 | 0 / -90 | 4 / 0.9 | schematic path |
| 7 | Corridor Overview | 27.8986, 84.8723 | 53,247 | 60 / -48 | 4 / 0.9 | path overview |
| 8 | Upper Valley | 28.4187, 85.4341 | 7,977 | 155 / -28 | 4 / 8 | trigger record |
| 9 | Upper Valley Collapse | 28.417, 85.3937 | 9,850 | 146 / -34 | 4.2 / 1.4 (min 10) | media autoplay |
| 10 | Debris-Dammed Lake | 28.3929, 85.5709 | 10,744 | 232 / -37 | 4.2 / 1.4 | media autoplay, path during media (4.8 s) |
| 11 | Second landslide | 28.3992, 85.5358 | 10,145 | 224 / -38 | 4.2 / 11 | path 6.5 s |
| 12 | Nepal-China Border Gate | 28.3168, 85.3997 | 5,900 | 208 / -42 | 4.2 / 2.5 (min 8) | 2021-vs-2026 comparison, evidence revealed after camera settles |
| 13 | Timure Evidence Cluster | 28.2860, 85.3791 | 5,032 | 198 / -40 | 4.2 / 4 (min 12) | comparison, path 2.1 s |
| 14 | Syabru Besi Passage | 28.2103, 85.3615 | 5,893 | 205 / -38 | 4.2 / 4 (min 11) | comparison, path 2.8 s |
| 15 | Dhunche | 28.1614, 85.3243 | 6,212 | 208 / -39 | 4.2 / 16 | path 2.5 s |
| 16 | Mailung / Upper Trishuli bridge | 28.1406, 85.2660 | 6,353 | 211 / -40 | 4.2 / 13 | path 2.9 s |
| 17 | Mailung Bazzar, Dandaguan | 28.1265, 85.2480 | 6,331 | 212 / -40 | 4.2 / 15 authored, runtime hold 7.65 | the one clip with a fixed 7.65 s dwell (7 s clip + 0.65 s exit) |
| 18 | Dandagaun, Rasuwa | 28.1244, 85.2452 | 6,311 | 214 / -40 | 4.2 / 15 | path 1.2 s |
| 19 | Dandagaun viewpoint restaurant | 28.0867, 85.2299 | 6,740 | 215 / -41 | 4.2 / 13 | path 2.5 s |
| 20 | Betrawati Bazaar | 28.0298, 85.2277 | 7,034 | 216 / -41 | 4.2 / 12 | path 2.9 s |
| 21 | Bhainse | 28.0231, 85.2294 | 8,051 | 219 / -41 | 4.2 / 13 | path 1.5 s |
| 22 | Bidur / Trishuli Consequence | 27.9817, 85.1963 | 7,422 | 215 / -40 | 4.2 / 4 | path 2.6 s, last sourced path reach |
| 23 | Devighat / Taadi Khola Bridge | 27.9211, 85.1771 | 9,056 | 224 / -42 | 4.2 / 13 | shows path history only |
| 24 | Charaudi | 27.8651, 84.8384 | 10,347 | 232 / -42 | 4.2 / 11 | path history only |
| 25 | Final view | 28.0834, 85.206 | 113,520 | 64 / -83, roll 174 | 4 / 11 | no media; locator returns to the incident-places view |

- Order note: the evidence order interleaves by geography (collapse, lake, second landslide, border gate, Timure, Syabru Besi, Dhunche, Mailung x2, Dandagaun x2, Betrawati, Bhainse, Bidur, Devighat, Charaudi, final). Evidence-clip shots (all except the final view) autoplay their media only when the user pressed Play Scene/Shot, with a media hold of 6 s (7 s for Mailung Bazzar) plus a 0.65 s exit, and evidence is revealed over 0.9 s after the camera settles. Every shot's hold is at least that media-plus-path time. A few opening beats carry an evidence hold card with 8-12 s minimum holds (rows 9, 12, 13, 14 above).
- Source-path reaches (those with a real river centreline) are camera-led for Timure onward; the lake and second-landslide reaches are media-led; the three last shots only keep the history of the path.
- Uses an event panel, geographic labels, scrubbing, evidence cards that link to (or, where permitted, embed) original witness posts, comparison of historical (2021) and post-event (2026) satellite crops over an Esri basemap, an animated schematic flood path that follows the active surface and reveals reach by reach after arrival, and a locator that orbits the upper valley.
- Basemap behaviour: Google 3D for authored 3D shots when available, else Esri; comparison shots always Esri; keyless terrain without a Cesium token. Saved shots keep their authored map preferences across keyed and keyless runs.
- Honesty rules: the flood path is schematic (not a hazard model, not arrival timing); the 2021 image is historical context, not a pre-event baseline; evidence counts from one source map are not independent confirmations; no witness clips are bundled (cards link out; embeds are disabled in embedded shells that would redirect them).
- Licence: the imagery crops and derived river centreline are CC BY-NC 4.0 (non-commercial); it is exposed to assistants through a dedicated query tool and its layers are hidden from the layer menu (scene-driven only).
- Data-pack style content authored by contributors is credited; contributor credit is preserved.

---

## 8. Cross-feature interactions
- Scene playback uses the same layer enable/disable path as manual toggles (including context-mode exclusivity and key-required states).
- Starting playback stops tracking, cockpit refuses scene camera claims, and manual navigation cancels playback.
- Voice can list/play/stop/advance/report scenes; "Play Orbital Watch" is the canonical voice example.
- Scenes can set the HUD layout, detection, scope keyhole, bloom, sharpen and basemap per shot, so they are the main route to repeatable "hero shot" sequences.
- Imported own-file overlays (GeoJSON/KML/KMZ/GPX, up to 12 files, 50 MiB each, session-only) are separate from scene data packs and are not part of scene files.
- Attribution for packs, event imagery and basemaps is registered in the credits popover and stays visible in presentation mode.

---

## Kawn notes (UAE city-intelligence: Dubai, Abu Dhabi, Sharjah; Arabic + English, RTL)

**Keep**
- The whole shot/scene model, ellipsoid-only heights, anchors, authored moves with shortest-arc interpolation and seek-equals-playback sampler: it is a clean, testable core for "guided tours" (city briefings, site visits, council presentations).
- Import preview before apply, bounded validation, field-path errors, protection of unreadable saved data, and "invalid import changes nothing".
- Data packs with mandatory attribution + licence, integrity hashes, size caps and no executable content - ideal for emirate GIS layers (project footprints, zones, flood areas).
- Declarative interactions (card/focus/shot/layer) with accessibility rules and the 64-transition cap.
- Bundles with hash verification and "re-import after reload" honesty.
- Presentation mode with safe-frame overlay (add a 4:5 or A4/landscape option for briefings, and make the frame shape a real per-scene choice that playback honours - the reference never applies 9:16).
- [CENSUS] Separate controls for "play this scene" and "play all", an explicit "restore my view after the tour" option (the reference has no restore pass), and a distinct "play from here to end of this scene" for event-style stories.
- [CENSUS] Scrub and playback should share one sampler for ordinary shots too (the reference scrubs by straight interpolation but flies live by arc).
- [CENSUS] Owner decisions: Kawn opens on Dubai with Scope on and feather 100 percent; scene shots that capture visual state must store scope keyhole and feather so a tour can return to that default, and tour presets should start from the Dubai view and the location tray groups (emirates, Dubai airports, London airports).

**Adapt**
- Replace the five generic showcase scenes with city tours: Dubai Marina/Palm/Downtown, Abu Dhabi Corniche/Saadiyat, Sharjah heritage/Corniche, airport approach corridors, port/free-zone views, construction-progress then-and-now.
- Replace the Nepal event pack with an incident/event template (e.g. flood or dust-storm, large fire, major road closure) using the same evidence-card pattern and honesty rules; keep "schematic, not a model" labelling.
- Arabic and English scene titles/labels/cards per scene (bilingual fields), RTL card layout, Arabic-aware text (no HTML in cards stays), localised numerals and dates.
- Add per-scene audience/permission flags (internal, partner, public) and an approval stamp on shared bundles.
- Add narration track support (pre-recorded or synthesised Arabic/English voice-over per shot) rather than relying on media markers.
- Add terrain-relative heights for tall-building/urban work only if a trustworthy city height model exists; otherwise keep ellipsoid-only and document it.
- Consider server-side scene storage and permissions instead of browser-only storage.

**Skip**
- Spy-sensor visual styles in scene presets (keep optionally for demos), the conspicuous classified-style HUD frames, and the Nepal content/licence entirely.
- Telemetry run-log download unless useful for QA; keep local-only if retained.

## Open questions / UNVERIFIED
- Exact visual look of the Scenes panel on narrow screens and the safe-frame overlay proportions were not observed live; only the controls and states are documented.
- Resolved: the 7.65 s dwell belongs to the Mailung Bazzar shot (7 s clip + 0.65 s exit); other shots use 6 s + 0.65 s of media time as a floor.
- Resolved this pass: version 6 is current (code and tests); the scene-document reference page is stale.
- The exact effective hold of each Nepal shot depends on runtime media and path timing floors; the table gives authored holds plus the floors found in the pack rules, not measured playback times.
- Behaviour of scene import when two scene ids collide with built-in ids (replace vs rename) - the doc says "apply replaces the whole project"; per-scene merge is not described. UNVERIFIED.
- Keyboard accessibility of the draft editor dialog (focus trapping) - not documented in the sources read.
- No live playback was run (static analysis only).

## Revision log
- Resolved the scene-format version conflict: 6 is current/export (validator accepts 1-6); documented the stale reference page as a doc-versus-code item.
- Replaced the Nepal shot list with a full 25-row table (position, ellipsoid altitude, heading/pitch, flight/hold, runtime floors, path timings) and explained the fresh-project seed-then-upgrade behaviour.
- Corrected Orbital Watch's start location and added per-scene shot counts and flight timings for all five generic scenes, plus the built-in install order (six scenes, Nepal first).
- Added missing validation details: polygon ring limit, feature-id length, interaction label length.
- Resolved the 7.65 s media dwell question; re-verified all numeric limits (5 MiB, 256 scenes, 10,000 shots, bundle 50 MiB/64 files, packs 8, interactions 64, anchors 1,024).
- Added confidence-tag legend.
- Pass 3 (census merge): corrected the 9:16 claim (playback always uses the 16:9 frame; 9:16 is never requested) and the City Overload row; added START scope (selected scene then all remaining, wrapping; only voice plays a single scene) and event-panel PLAY SCENE scope; added safe-frame geometry; stated that only scene preview runs set recording state and the share link does not carry it; corrected the shared-sampler claim (authored moves only; ordinary shots live arc vs seek straight interpolation); added the no-restore-pass rule, cleanup contents, scene clock and effective-hold floors, the 20 s media hold cap, the data-pack phase in the run sequence, and new Kawn notes tied to owner decisions. Confidence for census-derived text: source-level, no live run.

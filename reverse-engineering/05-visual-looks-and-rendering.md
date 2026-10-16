# 05 - Visual Looks, Basemaps and Rendering (behavioural study)

**Purpose.** This document records, as observable behaviour, how the reference product draws the planet: which imagery and terrain sources it can show and in what fallback order, how photoreal 3D tiles and terrain are handled, every whole-scene "look" (sensor/stylised preset) with its parameters and side effects, the post-processing controls (bloom, sharpen), the celestial ring, the circular scope treatment, Clean UI, the 3D aircraft models and their proximity swap, performance and quality governors, and the visual design language of the interface (palette roles, type, shape, motion, glass surfaces). It is written for a clean-room team: nothing here describes how the code is organised, only what a user sees and what numeric thresholds govern it. Items not confirmed from the source or the tests are marked UNVERIFIED.

Confidence tags used below: [OBSERVED-IN-SOURCE] confirmed by reading the reference implementation, [FROM-TESTS] confirmed by an automated test that states the intended behaviour, [FROM-DOCS-ONLY] stated only in the project's prose documentation or the live walkthrough, [INFERRED] derived by reasoning. Untagged numeric values were re-checked against the implementation in the revision pass unless a section says otherwise.

Mechanism detail: 13a sections 2.5 and 2.6 (basemap controller, token renewal, viewer set-up, render governor, Apple Metal workaround, celestial ephemeris, no sun shading) and 13f sections 1G to 1I and gap 6, 7 and 11 (looks, bloom, sharpen, idle cost of detection, defaults living in four places).

The product is a single full-viewport 3D globe with floating glass chrome on top. All looks are post-processing filters applied to the finished 3D scene image (globe, tiles, data glyphs and 3D models); the HUD, detection overlay, scope mask, celestial ring and panels are separate screen layers composited above it, and are not filtered. [OBSERVED-IN-SOURCE]

---

## 1. Basemap sources ("map stacks")

### 1.1 The five selectable sources

| Source label (short) | What it is | Needs | Terrain it pairs with |
|---|---|---|---|
| Photoreal 3D (3D) | Google Photorealistic 3D Tiles - textured city/terrain mesh. When active the plain globe is hidden entirely; the tiles supply their own terrain | A Google Maps key, OR short-lived tokens offered by the app's own server, OR a Cesium ion token (ion hosts the same Google asset) | None separate (the mesh is the surface) |
| Bing Aerial (Aerial) | Microsoft Bing aerial imagery delivered through Cesium ion | A Cesium ion token | World terrain (Cesium) with vertex normals, no water mask |
| Bing Labels (Labels) | Bing aerial-with-road-labels through Cesium ion | A Cesium ion token | Same as above |
| Esri Satellite (SAT) | Esri World Imagery tiles (Maxar/Earthstar etc.), keyless | Nothing | Keyless open terrain (see 1.3) unless an ion token exists, then world terrain |
| OSM | OpenStreetMap raster tiles, keyless | Nothing | Same as Esri |

- Esri Satellite is the keyless default; OSM is the last-resort safety net.
- Picture/pick-feature info is disabled on the Esri layer (no feature popups).
- Bing imagery is fetched through the ion endpoint and forced onto HTTPS even when the page runs on plain local HTTP.
- The tray shows one chip per source in a fixed order: 3D, Aerial, Labels, SAT, OSM, headed "MAP SOURCE" with a small live status word (the active source's short label). The active chip is lit; unavailable chips are greyed, carry a hover reason, and an ion-dependent chip shows a small "ION" badge. Clicking an unavailable chip does nothing (it is announced to assistive tech as unavailable with the reason).
- The lit chip always follows the source actually on screen, never the one just requested, so a failed or superseded switch leaves the true source lit. The chip list is a fixed allow-list of five. Pressing Enter on the tray opener hands keyboard focus to the active chip. [OBSERVED-IN-SOURCE, 13a]
- **Bing via ion, in detail:** the app asks ion for the asset's endpoint, checks that the answer really describes Bing imagery, builds the imagery provider from the returned key (forced onto HTTPS because Bing otherwise follows the page's plain-HTTP scheme on a local address), and merges ion's attribution text into each tile's credit line. Not run in the observed session. [OBSERVED-IN-SOURCE, 13a; appearance UNVERIFIED]
- **Voice map switching:** spoken aliases resolve to the five ids; "satellite(s)" always means the satellites data layer and never a basemap; road-map phrasings go to OSM; an unknown name returns an error; the reply reports the requested versus the resulting source (see doc 08).

### 1.2 Startup route and fallback order

1. Boot decides the best route: direct Google access if a Google key exists; otherwise server-issued short-lived Google tokens; then, if an ion token exists, the ion-hosted Google asset as a second attempt (recovery path if the direct route fails). **Token renewal (corrected in the census merge):** the client caches the issued token with its expiry and renews it one minute early; concurrent callers share one in-flight renewal. The retry is per request, not a count of two: when a tile request is refused as unauthorised or forbidden, the client asks the server for a replacement token (naming the one that failed) and retries that single request once, but only if the replacement differs from the token that failed. The server keeps the long-lived Google key private. [OBSERVED-IN-SOURCE, 13a NAV-45]
2. If one attempt yields a tileset, Photoreal 3D is the initial source and the plain globe stays hidden. If every attempt fails or no credential exists, the loader text says the 3D tiles are unavailable (with the reason) and the keyless globe loads, and Esri Satellite becomes the initial source.
3. Loader wording at boot: "configuring viewer", then "loading Google 3D" when any credential/token route exists or "loading the keyless globe" when none, then "initializing systems".
4. If the initially requested source (e.g. from a share link) is unavailable, the default source is used instead. An unknown source id resolves to Photoreal 3D; if that is itself unavailable the user gets a toast with the tailored reason and nothing changes.
5. Unavailability reasons are tailored: with no credentials at all, the message asks for a Google key (or an ion token for the hosted route); with credentials present the message instead says to check key restrictions, quota or network (it never tells a user to add a key they already added). Ion-dependent sources point at the ion token.
6. **Esri to OSM fallbacks:** (a) if the Esri provider cannot even be constructed, the controller silently selects OSM and reports "Esri Satellite is unavailable; using OSM"; (b) once the Esri layer is live, if two tile requests fail, the controller switches to OSM automatically and reports "Esri Satellite tile requests failed; using OSM". Automatic switches are marked as controller-chosen, not user-chosen, so a scene or lease can tell them apart.
7. If a manual switch throws, the controller tries the recovery source (Photoreal 3D when it exists), then reports the error. A resolved switch call does not mean success - callers must read the resulting active source.
8. Rapid switches are generation-counted; a stale completion is ignored.
9. Switching shows a "switching" state, releases the previous imagery layer (destroying it), swaps the credit line, shows/hides the globe and tilesets accordingly, and requests a render frame. Providers, terrain and tilesets are cached per source; creation is lazy (a hidden globe never triggers terrain downloads) and aborts on teardown. Further controller rules: "silent" switches (boot, scene hand-offs) emit no intermediate state; the origin of every switch is recorded as manual or automatic; re-applying a stack whose provider is unchanged keeps the live imagery layer, so the bare globe is never exposed for a frame; the imagery layer always sits at the bottom of the layer stack; subscribers are told once the active source has settled. [OBSERVED-IN-SOURCE, 13a NAV-50]
10. Imagery comparisons (before/after tools) can lease the map: only one lease at a time; policies are "never switch", "force Esri from any source", "force Esri only if Photoreal is active". The comparison baseline is Esri satellite. Releasing restores the earlier source only if the user has not chosen a different one meanwhile; releasing mid-acquire re-issues the earlier source so a late activation cannot survive; the lease stays owned until restoration settles.
11. Changing source broadcasts a state event so data layers that depend on the ground (camera feeds, ground resolution) can re-evaluate; every switching, ready and error state is re-announced, and listeners derive the surface regime from live scene state, so intermediate emissions are harmless. Display tray and share link persist the active source.
12. **Share-link behaviour for the source:** stored as a short id; restored after the camera and awaited before the final share-state write; a newer explicit choice by the user claims the "map" restore lane and wins over a delayed restore; a link without the field means Photoreal 3D. A retired road-map style id resolves to OSM. [OBSERVED-IN-SOURCE, 13a NAV-53]

### 1.3 Terrain

- **Keyless terrain:** open quantised-mesh terrain from the Re:Earth/Mapterhorn service (CC BY 4.0, attribution required), ellipsoid-referenced. If it cannot be created at all, a flat ellipsoid is used silently (console warning only).
- **Throttling policy for that service:** tiles answered with throttling (429) or transient gateway errors (502/503/504) are retried up to 3 times with exponential backoff starting at 750 ms, capped at 15 s, with up to 1.5 s random spread. All retries wait behind one shared cooldown per policy, so a burst of throttled tiles comes back as a jittered trickle rather than a second burst; a Retry-After hint (seconds or date) extends the cooldown. Anything else (404 outside coverage, network failure, abort) is not retried so flat terrain falls back promptly; a tile cancelled while waiting is not re-requested. One log line per cooldown window.
- Observed in the live walkthrough: under load the terrain can 429 repeatedly and "pop in" late; wait several seconds before capturing footage.
- **Token terrain:** with an ion token the Cesium world terrain (asset 1) is used with vertex normals (water mask off).
- Retry detail: a wait never exceeds the maximum backoff plus the random spread, and the Retry-After hint only lengthens the shared cooldown. The shared ground-height service that aircraft, trails and arrival framing use is described in doc 06 (ground height) and doc 03 (ground floor tiers).
- **Photoreal mode ignores separate terrain** on purpose (a separate terrain conflicts with the photoreal mesh at close range).
- Terrain is set only when a globe-based source is activated and the terrain mode actually differs.

### 1.4 3D tiles behaviour

- Tileset cache budget 1.5 GB with up to 1 GB overflow; collision enabled; imagery loaded asynchronously so tiles keep their own texture while draped weather loads.
- The tileset is a shared object reused by the map controller; only the tileset marked shown is the "imagery host" other tools (imagery boxes, comparison) can use.
- Because the plain globe is hidden under Photoreal, ground-height queries from the globe return nothing; the product therefore uses curated city ground elevations, a rendered-mesh height sampler, or an elevation service (see doc 06 for the ground guard).

### 1.5 Sky and atmosphere

- Sky atmosphere always on, with light intensity 18, saturation shifted -0.12 and brightness shifted -0.08 (slightly muted, darker sky than the engine default; the reduction exists so the distant edge of photoreal tiles does not show a cyan seam against the sky). Stars/space are visible at orbital altitudes; page background behind them is near-black blue.
- Four-sample multisampling antialiasing is requested; the drawing buffer is preserved so screenshots/recording can read the canvas; target frame rate is 60.
- Fog: engine fog stays on in ordinary looks. In NVG and FLIR (and the cockpit equivalents) fog is switched off and restored on exit, because fog blends distant 3D aircraft to a black silhouette beyond roughly 100 km (see 2.6).
- **Apple Metal atmosphere workaround (expanded):** on Apple Metal/ANGLE systems the translation layer rejects the engine's per-vertex atmosphere shader for 3D models; the link failure used to tear down the render loop on iPhone and iPad. Before any tile builds a draw command, the app runs a throw-away probe context that tries the offending shader pattern and releases the context at once. If the probe fails, or if no probe context can be created and the platform is iOS or iPadOS (backstop), the engine's "fog renderable" flag is cleared while fog stays "enabled", so tile level-of-detail scaling still sees the fog density. Sky atmosphere and ground atmosphere are unaffected. Cost: no distance fog on 3D tiles, and none on the keyless, Esri or Bing globe basemaps, on affected devices. A future driver fix restores the effect with no change. Needed for any Kawn iPad support. [OBSERVED-IN-SOURCE, 13a NAV-60, 13f SYS-172]
- **Viewer set-up:** every stock widget of the engine (timeline, clock widget, base-layer picker, geocoder, home button, scene-mode picker, help, fullscreen, VR, selection indicator, info box) is switched off; there is no default base layer (the map controller adds one); the engine's credit display is redirected to the app's own container; the sky atmosphere is on and the globe hidden from the first frame; a construction error destroys the half-built viewer. [OBSERVED-IN-SOURCE, 13a NAV-56]
- **No sun-driven lighting:** the app turns on no globe lighting from the sun and draws no day/night terminator; the application clock is frozen; the sun position is used only by the celestial ring (section 5) and the HUD sun-elevation readout. Photoreal tiles carry their own baked daylight, so cities always look as photographed. A night terminator tuned to Gulf time zones is a possible later addition. [OBSERVED-IN-SOURCE, 13a NAV-66; whether any other effect traces the sun is UNVERIFIED]

---

## 2. Visual looks (presets)

### 2.1 Selection, display and crossfade

- Seven looks in a tray titled "VISUAL PRESETS" (bottom dock, collapsible, pinnable): **Normal (1), CRT (2), NVG (3), FLIR (4), Anime (5), Noir (6), Snow (7)**. Number keys 1-7 select them (ignored while focus is in a text field, select or input; the key handler does not check for modifier keys, so a browser shortcut that includes a digit also changes the look when focus is on the page) [OBSERVED-IN-SOURCE]. Each button shows an icon glyph, a label and a key hint, with a one-line hover description.
- Exactly one look is active. The previous look fades out while the new one fades in at the same time over **500 ms** with an in-out ease (quadratic). A small "ACTIVE STYLE" indicator and the collapsed tray's "Style" mini-status show NORMAL / CRT / NVG / FLIR / ANIME / NOIR / SNOW.
- Selecting any look (except Normal) opens the **Parameters** panel under Display with one slider per look parameter (range, label, live numeric value with two decimals for 0-1 ranges, one decimal otherwise). Normal shows no parameter panel. Re-selecting the active look just re-reveals its parameters.
- Look and its parameter values are part of the share link (shared parameters are per-look); share links from a retired "AI edit" look are ignored.
- The looks are screen-space filters: they affect the entire canvas including 3D aircraft and cockpit view. The detection overlay, HUD and panels are separate layers on top and are not filtered, though detection recolours itself to suit (see 2.5).
- **Celestial ring** only works in Normal; leaving Normal turns it off, returning merely re-enables the button (see section 5).

### 2.2 Per-look description and parameters

Defaults below are the shader defaults used when the user has never touched a look, followed by the "tactical preset" values the three military looks apply when selected through the tray (see 2.3).

| Look | What it does visually | Parameters (range, shader default; tactical-preset value) |
|---|---|---|
| **Normal** | Unfiltered globe | none |
| **CRT** | Dynamic CRT terminal: pixelation snapped to a grid, 8x8 ordered dithering with posterisation and a small saturation boost, barrel (lens) distortion with black outside the bulge, chromatic aberration, rolling horizontal scanlines, RGB shadow-mask sub-pixel stripes (visible only at higher pixelation), edge darkening of pixel blocks, phosphor persistence/ghosting, horizontal jitter on a few lines at a time, brightness flicker (~50-60 Hz feel), rare bright glitch bars, a warm amber-green phosphor tint, and a curved-screen corner vignette. Animated (time-driven) | Pixelation 1-10 (default 5.0; tactical 1.0); Distortion 0-1 (default 0; tactical 0); Instability 0-1, jitter/flicker/glitch frequency (default 0.4; tactical 0.42) |
| **NVG** | Night-vision goggles modelled on a modern image-intensifier tube: P43 phosphor green, intensifier bloom/halo around bright sources with a wide kernel, circular tube vignette (black outside the tube, brightest at centre, falloff), barrel lens distortion, auto-gain, scintillation sparkle noise (stronger in dark areas), a fibre-optic honeycomb texture, subtle display scanlines, a thin crosshair and a seven-segment HH:MM:SS timestamp burned into the image. Animated | Gain 0-1, amplification vs noise/bloom balance (default 0.55; tactical 0.18); Bloom 0-1 (0.3; 0.22); Scanlines 0-1 (1.0; 0.96); Pixelation 1-6, tube resolution (2.5; 1.0) |
| **FLIR** | Forward-looking infrared: circular lens vignette (black outside), soft IR diffraction blur, luminance remapped to a pseudo-temperature, contour/temperature banding, slowly drifting coherent temporal noise, hot-spot bloom/bleed, sensor pixelation, and a burned-in sensor HUD (FLIR label and mode marker top-left, a simulated temperature readout of 20-50 degrees C derived from centre brightness top-right with a seven-segment look, a frame counter bottom-right, a vertical temperature scale bar on the right edge, a centre crosshair). Two ways to show heat: monochrome (white-hot or black-hot) or the "Ironbow" colour ramp (black to deep purple, magenta, red, orange, yellow, white). Animated | Sensitivity 0-1, contrast/range of the temperature mapping (0.75; 0.85); Bloom 0-1 (0.65; 0.2); WHOT/BHOT 0-1, below 0.5 = white-hot, at/above 0.5 = black-hot (0; 0.33); Pixelation 1-6 (1.5; 1.0); Ironbow 0-1, 0 = monochrome, 1 = colour ramp (0; unchanged) |
| **Anime** | Cel shading: luminance quantised into bands (from near-continuous at zero strength down to about 4 bands at full strength), saturation lifted, brightness up ~10%, soft edge-detected dark outlines, a slight warm colour shift | Saturation 0-2 (1.0); Edge Thickness 0-1 (0.5) |
| **Noir** | Desaturation, S-curve high contrast, film grain (random per-pixel noise), edge vignette, 15% sepia warmth | Contrast 0-2 (1.2); Grain 0-1 (0.5); Vignette 0-1 (0.5) |
| **Snow** | Cool blue colour shift, partial desaturation, global brightening, white frost on brighter areas, five layers of falling snow particles (nearer layers larger and faster, swaying with a bounded wind sway), faint low-screen fog. Animated | Density 0-1 (0.6); Wind 0-1 (0.5) |

- Sliders step by 0.01 for 0-1 ranges, otherwise 0.1.
- Each look is one full-screen fragment-shader pass; Normal has no shader. The NVG output is masked by the tube falloff so colour does not bleed past the tube edge (a fixed defect). The thermal look offers the Ironbow colour ramp as an option, with the accurate grey mapping as the default. [OBSERVED-IN-SOURCE, 13f SYS-160 to SYS-164]
- A project note (not the shader table above) states that first-run starting parameters for CRT, NVG and FLIR set pixelation to 1.2, just above the native floor of 1.0. This conflicts with the shader default of 5.0 for CRT and with the tactical-preset value of 1.0 in the table; the tactical value applies on a user selection, the shader default applies otherwise. [FROM-DOCS-ONLY; the 1.2 figure is UNVERIFIED against behaviour]
- Animated looks (CRT, NVG, FLIR, Snow) hold the render loop continuously while visible; static ones do not (see section 7).
- All looks blend by an intensity that also rides the 500 ms crossfade, so partial intensities are used mid-transition. A stage is switched off entirely once its intensity is effectively zero.

### 2.3 What selecting a look changes beyond the picture (per-look side effects)

On a user selection (not on restore from a share link or scene):

| Look | Parameters set | Bloom | Sharpen | HUD layout / visibility | HUD colour | Detection |
|---|---|---|---|---|---|---|
| Normal | none | unchanged | unchanged | unchanged | cyan | unchanged |
| CRT | Pixelation 1.0, Distortion 0, Instability 0.42 | forced off at 0 | forced on at 49% | forced to Tactical (unless Cyber is active) and visible | amber (255,170,0 at 80%) | auto-set to Dense at 75% unless the user has hand-edited detection this session |
| NVG | Gain 0.18, Bloom 0.22, Scanlines 0.96, Pixelation 1.0 | off | on, 49% | Tactical, visible | green (51,255,51 at 80%) | Dense at 75% (same rule) |
| FLIR | Sensitivity 0.85, Bloom 0.2, WHOT/BHOT 0.33, Pixelation 1.0 | off | on, 49% | Tactical, visible | white (255,255,255 at 70%) | Dense at 75% (same rule) |
| Anime, Noir, Snow | none (parameters keep shader defaults or the user's last values) | unchanged | unchanged | unchanged | cyan (default) | unchanged |

- Detection auto-enable: hand-editing detection at any time sets a session-long "user overrode detection" flag; after that, no look will auto-set detection. A factory default does not set the flag.
- The HUD mode label (see doc 07) shows NORMAL, CRT, NVG, FLIR, ANIME, NOIR, SNOW; the HUD colour swatches above come from this table.
- The HUD has an "auto" visibility mode in which it follows the look (shown for CRT, NVG, FLIR, hidden for the rest). The first-load baseline (section 3.4) and every military-look selection set the HUD explicitly on, which leaves auto mode, and the HUD button also leaves auto mode when pressed. After any of those, switching to a non-military look does not hide the HUD. Auto mode can only be re-entered through voice or scripting, or by the scene/recording system; in that state the HUD does show and hide with the look. [OBSERVED-IN-SOURCE] (resolved in the revision pass; no longer unverified)
- Entering **Cyber** HUD layout (explicit user choice only) additionally switches the look to FLIR with the Ironbow value set to 0.42, remembers the previous look, and restores it when the user explicitly leaves Cyber (unless they picked a look themselves while in Cyber). Restores from share links and scenes never trigger this swap and never overwrite later user FLIR tuning.
- Cyber sonar (a rotating-sweep contact highlighter) is enabled by default whenever Cyber is selected; its five sliders (rings 3-12 default 8; range 60-120% default 100; power 0-100% default 70; opacity floor 35-100% default 84; sector width 8-60 degrees default 24) are session-only and reset on reload. Reduced-motion hides the animated sweep.

### 2.4 Cockpit vision

Inside the cockpit (doc 06) the same seven looks are available as "vision" modes (Normal is called Optical; the order is Normal, CRT, NVG, FLIR, Anime, Noir, Snow) selected with previous/next/current chips. Rules:
- Cockpit starts in the vision that matches the current map look.
- Selecting a vision only changes which filter is at full intensity (all others zero); it does not change any look parameters, and it does not change the map look. On exit the filter intensities saved at entry are restored.
- Vision changes reveal the current look's parameter sliders inside the cockpit Display popover.
- NVG/FLIR cockpit vision also triggers the sensor boost described next.

### 2.5 Sensor boost for aircraft and detection recolouring

- **Sensor boost** (map or cockpit): while NVG or FLIR is effective, aircraft glyphs and 3D models of the civil and military flight layers switch to a high-contrast "hot" rendering (white tint, full colour blend, special shader; glyph icons use a hot variant), fog is disabled, and other layers whose sprites restyle for sensors receive a "vision changed" notification carrying the effective look and whether the cockpit owns it. Leaving the sensor look restores fog to its prior value. Switching boost state reloads the whole 3D fleet, dropping models in batches of 40 per frame tick so the render thread is not stalled (the billboard is shown first for each plane, so no gap or double image).
- **Detection recolouring:** the detection overlay has three themed palettes (CRT amber, NVG green, FLIR warm white) and a default cool-cyan palette used for Normal, Anime, Noir and Snow. Each theme sets line colour, label colour, label plate, glow, blend mode (screen), a contrast/saturation tweak and a scanline strength (about 0.085 for CRT, 0.09 NVG, 0.04 FLIR, 0.05 default). Colour tiers per theme are listed in doc 07.

- **Click correction under distortion (transit layer only):** because the NVG lens bulge and the NVG/FLIR pixel quantisation move what the user sees away from where the scene was actually drawn, a click on a transit vehicle is mapped backwards through the active look's distortion and pixel grid before the vehicle is picked, so the vehicle under the visible pixel is the one selected. No equivalent correction was found for other layers (CRT distortion is not corrected at all). [OBSERVED-IN-SOURCE]
- **Detection tone only follows the three sensor looks.** CRT, NVG and FLIR each have a palette; every other look (including Anime, Noir and Snow) uses the default cool palette. [OBSERVED-IN-SOURCE]

### 2.6 Why fog is special (design note)
The globe is hidden in photoreal mode, so engine fog blends far geometry to black; this turns distant 3D aircraft into black silhouettes beyond ~100 km. Infrared sensors "see through haze", so those looks simply turn fog off.

---

## 3. Post-processing controls (Display panel)

### 3.1 Bloom
- A toggle ("Bloom") plus a slider revealed when on. Range **0-200**, default **0 and off**. Older share links used a 0-100 inverted scale; on load any value at or below 100 in an unversioned link is converted (new = (100 - old) x 2) and any value above 100 is taken as already new-scale. A link version flag marks the new scale. The same migration is applied on every ingestion path: links, scenes and voice. [OBSERVED-IN-SOURCE, 13f SYS-167]
- Effective behaviour: strength = value / 200. Below about 6% strength nothing is drawn (the stage is disabled). Above that, strength is renormalised and eased with a smoothstep curve; the underlying engine bloom stage is retuned continuously: contrast 255 down to 87, brightness -0.5 up to -0.14, blur sigma 0.28 up to about 6.6, delta 0.2 up to 2.45, step size 1.0 up to 2.25 (glow-only mode off). The engine's own bloom stage is borrowed, and its previous settings are restored when the app tears down.
- The looks have their own bloom parameters (NVG, FLIR); those are independent of this global bloom.

### 3.2 Sharpen
- A toggle plus slider 0-100%, default **on at 49%**. It is an unsharp mask over a 3x3 neighbourhood (centre plus (centre minus 3x3 average) x amount), clamped to valid colour, with amount = 0.1 + 2.0 x (slider/100) (so the default is about 1.08). Alpha is preserved.
- Military looks reset it to on/49% (see 2.3).

### 3.3 Stage ordering and persistence
- **What the product itself sets up (in this order):** (1) the six look filters are registered first, in the fixed sequence CRT, NVG, FLIR, Anime, Noir, Snow, all starting switched off at zero strength; (2) the product then borrows the engine's built-in bloom stage (remembering its previous settings, switching it off, and retuning it) and registers its own sharpen stage after the looks. The product enables no other engine stages: no anti-aliasing post filter, no tone-mapping stage and no ambient-occlusion stage were found; smoothing relies on the four-sample multisampling of the main scene. [OBSERVED-IN-SOURCE]
- **Consequence for the picture:** the looks and sharpen are chained in registration order, so the sharpen filter acts on the output of whichever look is visible (a sharpened CRT, NVG or FLIR picture), not the other way round. Because only one look is normally at full strength, the order among looks only matters during the 500 ms crossfade, when two adjacent filters overlap and the later-registered one is applied on top of the earlier. [INFERRED from registration order; the engine's chaining rule is assumed to be "in the order added"]
- **Where the engine's bloom sits:** it is one of the engine's built-in stages, whose position relative to user-added stages is decided by the engine library, not by the product. The reference product's own notes do not state it. Treat "bloom before the looks and sharpen" as the likely order and verify empirically if the exact look matters. [UNVERIFIED]
- **Always outside the chain:** the scope darkening, celestial ring, detection overlay and its cards, HUD, annotations, panels and the cockpit cloud layer are all separate screen layers, so none of them is distorted, sharpened or bloomed by a look. The engine's picking buffers also precede the chain, which is why the transit click correction above exists. [OBSERVED-IN-SOURCE]
- **Persistence:** Bloom/sharpen enabled flag and intensity, look, look parameters, HUD layout/visibility, detection mode/density/allocation/fade/outside, celestial ring, scope state, map source are all part of the share link. Scenes (tours) can also set style, bloom, sharpen, HUD, detection and scope per shot.

### 3.4 First-load baseline
On first load (before any share-link restore) the app applies: bloom off at 0; sharpen on at 49%; HUD Tactical and visible; detection **on, Dense, density 75%, allocation Elastic (or the stored preference), fade 7%, outside opacity 1%**; celestial ring off; scope on with feather 11%; 3D aircraft on in Proximity mode. Shared links override these. Detection is on for every look including Normal (owner decision recorded in the source).

**Where a default lives, and what actually persists (census merge, 13f gap 11).** A first-run default is held in four places that must be changed together: the engine-side constant, the markup value and its readout, the UI baseline applied at first load, and the share-link generator's starting state. The parse fallback for a link that omits a field is a fifth, deliberately different place: it must stay at the value of the era in which that link was written (an old link without a feather field is read as the old default of 35%, so it keeps what its author saw). Only the 3D-models choice has durable browser storage; detection mode and density and the scope feather persist only in the share link, written on a 500 ms debounce, so a fresh tab returns to the defaults above. The first-load detection baseline does not count as the user's own choice, so it never blocks a military look from auto-setting detection.
- **Kawn owner decision (2026-10-16):** the Kawn first load is Scope on with feather at 100%, centred on Dubai; this replaces the 11% reference default above. Change it in all four places and keep the parse fallback for any future link without the field at 100%.

---

## 4. Scope (circular viewport)

(Detection fade/outside belong to doc 07; the Scope itself is a visual.)
- A "Scope" toggle (on by default) darkens everything outside a centred circular "keyhole" whose radius is 1.05 x half the viewport height. Inside is untouched; outside fades to near-black page colour.
- **Feather** slider 0-100% (default **11%**, shared state) = softness of the mask edge as a percentage of the keyhole radius: 0 is a hard crop, 100 is a long smooth fade. (The retired default values were 35%, then 0%, then 8%; 11% is the final.)
- **Altitude-adaptive terminus:** the darkness reached at the outer edge is 94% opaque only at true full-globe altitude (above 10,000 km camera height, where faint stars in the corners are kept) and ramps by smoothstep to 100% opaque by 7,000 km and below. A share-link value can pin it between 94% and 100%. The mask repaints only when the quantised step changes (about a dozen repaints across a full zoom), never per frame; a parked camera costs nothing; Scope off costs only the single clear.
- Repaints on resize, pixel-ratio change (e.g. dragging between monitors; backing store capped at 2x) and tuning changes.
- The mask sits above the globe and below the detection surface, so detection blending composites over the masked scene.
- Scope off removes the black surround entirely (full rectangular scene).
- Interacts with: NVG/FLIR have their own lens/tube vignettes inside the same circle; detection label fade is measured against the same keyhole; the celestial ring is positioned on the keyhole circumference; the cockpit altitude/speed tapes curve to follow the same circle.

---

## 5. Celestial ring

- Button "Celestial" in Display; **Normal look only** (disabled with an explanatory hover otherwise; entering any other look turns it off).
- When on and the whole Earth disc fits inside the keyhole (hysteresis: becomes visible when the disc clears the keyhole by 24 px, hides again below 12 px), a thin ring outlines the keyhole and two markers ride its circumference: a sun marker (warm gold, soft inward rays, tapered arc, sun-icon glyph) and a moon marker (cool cyan haze, tapered arc, moon-icon glyph). Marker angles come from real sun/moon directions projected into the camera plane; if the direction points almost straight at/away from the camera the marker holds its last stable bearing and fades. If the two markers would collide (within 6 degrees) the second is pushed inward by 32 px. Markers sit 36 px inside the ring, the ring 11 px inside the keyhole.
- Sun and moon directions are sampled from a planetary ephemeris once on enable and every 60 seconds from real wall-clock time (not the frozen app clock); a request-frame is issued so the ring repaints under the idle governor. Technique: a published low-precision planetary-position series (Simon 1994) gives the Sun and Moon in an Earth-centred inertial frame; they are rotated into the Earth-fixed frame with the engine's celestial-to-terrestrial matrix (a simpler fallback matrix is used if the first is unavailable), then normalised to direction vectors. The ring is cosmetic and tied to the Normal look; Kawn can skip it. [OBSERVED-IN-SOURCE, 13a NAV-65]
- Drawing is capped at 30 fps, with backing canvases limited to 2,000,000 pixels and 1,600 px per side at a pixel ratio of at most 1.25.
- Turning the toggle on **flies the camera out to the full-globe composition** (2.4 s, cubic in-out, straight down, heading kept) preserving the hemisphere under the camera; the Earth disc is framed so its radius is about 61% of the keyhole radius, and the flight never ends lower than about 1.55 Earth radii of altitude. If the globe is already framed nothing moves. If the user interrupts that flight and the globe is not framed, the toggle turns itself off. [OBSERVED-IN-SOURCE]
- **Automatic switch-off:** whenever the Earth disc is not fully inside the keyhole (the hide threshold above) and no focus flight is running, the toggle turns itself off, not merely the drawing. So the ring cannot be armed while zoomed in: enabling it from a share link or tool without the focus flight switches it off again on the next frame. [OBSERVED-IN-SOURCE]
- **Resolved altitude question:** the end altitude is computed from the window shape and the field of view, so it varies with the window (roughly 22,000 km for a 16:10 window; the walkthrough's 24,900 km was a different window shape). It is not a fixed number. [INFERRED from the framing rule]
- The sun and moon markers are the product's light-mode and dark-mode icon glyphs, tinted warm gold and cool cyan; this is why those two glyphs are in the pinned icon subset. [OBSERVED-IN-SOURCE]

---

## 6. Clean UI

- "Clean UI" is a button in Display (tooltip "hide UI chrome"; it shows lit while on) that hides interface chrome in one step. [OBSERVED-IN-SOURCE]
- **Hidden (fade to invisible and stop receiving pointer input):** the title block and its FPS readout, the active-style indicator, the top camera/share action cluster, the global loading chip and the traffic and camera-feed sync chips, the Display panel, the Parameters panel, the whole bottom command dock (location tray, visual-presets tray and voice area are inside it), the left layer rail and every panel in it (layers, camera feeds, scenes), the right rail (Context, Weather, Recent imagery, Street level, Radio and the rest), any popup attached to those panels, the expanded street-level viewer, the API-key setup chip and card, the first-run launcher card, the voice pointer reticle, and the wind-inspection marker.
- **Exit control:** a pill labelled EXIT CLEAN VIEW appears at the top centre (about 18 px from the top, small monospace uppercase text, translucent near-black fill, light border), above every other layer. Pressing it, pressing the Clean UI button (now hidden) or pressing **V** returns the interface; V works whether or not the pill is visible. Voice and scripting can also set or toggle it and receive the resulting state back.
- **Escape does not leave Clean UI.** No handler for Escape exists for this mode; Escape keeps doing whatever it does elsewhere (deselect, close a lightbox, leave Draw or Cockpit) while the chrome is hidden. [OBSERVED-IN-SOURCE] (resolved)
- **V is a plain key shortcut** with no modifier check, ignored while focus is in a text field, select or input. [OBSERVED-IN-SOURCE]
- **Not hidden:** the HUD (with its summary line), the detection overlay and its cards, the scope darkening and celestial ring, annotations and drawn shapes, the scene playback runtime bar, the **voice control pill and its card** (only recording mode hides it; resolved), the cockpit overlay if Cockpit is active, and the **required data/imagery attribution**, which moves to the bottom-left corner at a 36 px inset and stays fully usable (the Data attribution link still opens its lightbox). [OBSERVED-IN-SOURCE]
- **Not saved:** Clean UI is not stored in share links or scene shots; a reload or a recipient opens with the interface visible. [OBSERVED-IN-SOURCE]
- **Embed mode** (the app framed inside another page, or loaded inline in a host panel) switches Clean UI on automatically, also hides the HUD, setup prompts, dock, rails and the exit pill, and keeps only the globe plus the attribution line. [OBSERVED-IN-SOURCE]
- Recording mode (scene playback) hides a nearly identical set of chrome, additionally hides the voice pill and the exit pill, keeps the scene runtime visible and shows a 16:9 or 9:16 safe-frame guide (see doc 07). The attribution rule is identical in both modes.
- Chrome that sits above the map in Cockpit (the first-person overlay, its Display and Radio popovers) is separate from Clean UI and unaffected by it. [INFERRED]

---

## 7. 3D aircraft models and the proximity swap

### 7.1 Controls
- Display has a "3D" toggle (default **on**) and a "Models" two-way choice: **Proximity** (default) or **All**. The same setting drives both civil and military flight layers and persists via the layer state/share link. The choice row is visible only while 3D is on.

### 7.2 Swap rules (fleet)
- Below a camera altitude of **800 km**, aircraft in view can be drawn as 3D models; above, they remain flat icons. Eligibility is based on being on-screen (inside the view frustum and not behind the horizon).
- **Proximity:** new models are admitted for planes within 150 km of the camera and kept out to 185 km (hysteresis so a plane straddling the edge does not flicker); hard cap 150 models at once.
- **All:** admitted within 400 km, kept to 450 km; cap 350 models.
- Both are one draw call per model, so the caps bound frame cost. Beyond the add radius a model would clamp to a minimum pixel size and become a giant floating blob, so far planes stay as 2D dots. Slots go to on-screen planes nearest the camera first.
- In the cockpit the model budget is capped at 60 concurrent models and never raises the map cap; nearby aircraft inside the cockpit "near band" remain aircraft silhouettes even when 3D is off or loading; far ones become small contact pips (ring with a bright centre dot, civil cyan-white, military amber). The own aircraft is never drawn in first person.
  - **The near band is the same pair of radii as model admission** (add within 150 km and keep to 185 km in Proximity; add within 400 km and keep to 450 km in All), independent of whether 3D models are enabled, loaded or capped. [OBSERVED-IN-SOURCE, 13b AIR-88]
  - **Pips:** a 16 px ring with a 6 px centre dot, drawn from one shared image so hundreds of billboards reuse a single texture identity (a fresh texture per billboard stalled cockpit entry). Entering the cockpit never destroys models synchronously (the browser became unresponsive); the tracked aircraft's own model is hidden because the eye sits 7 m behind its origin and the airframe would fill the visor.
- **Billboard base size:** a far-view aircraft billboard is about 20 px (24 px when tracked), drawn from a 96-unit vector source rasterised at about twice that density; the 2D scale tables in docs 03 and 05 are multipliers of that base. Far aircraft also recede between about 3,500 and 4,500 km camera height (down to 45% scale and 35% alpha, never culled) so a whole-globe view is not a white blizzard. [OBSERVED-IN-SOURCE, 13b section 1.4]
- Minimum on-screen size for a model is 24 px (so distant ones read like the 2D icon); the tracked aircraft's model is clamped between 40 and 200 px apparent size, and scaled down further when the camera is extremely close so it cannot fill the viewport.
- Models are lightly tinted: 94% colour blend toward the contact's tint (white/civil, amber military #FFB800, red for transponder emergencies, cyan when tracked) so they read as clean light silhouettes. Grounded planes use a smaller 2D scale (x0.8) and are lifted so gear/belly rest on the surface.
- Grounded aircraft (taxiing, rolling) render exactly like airborne ones, with no trails; landing/takeoff restyles in place without removal.
- Heading uses a 180 degree model offset; icon rotation is computed from the course projected into screen space so it is correct at every camera tilt and when the camera orbits a tracked plane.

### 7.3 Tracked aircraft model (separate rule)
- The selected aircraft takes its 3D model on a different, always-on rule that ignores the Display toggle: it becomes a model when the camera is nearer than **150 km** altitude and returns to the flat cyan icon only once the camera climbs past **172.5 km** (15% hysteresis). A tracked converted-novelty contact has no model and stays an icon. A failed model load is retried at most 3 times with 1.5 s backoff per selection, then the icon stays.
- The tracked glyph stays solid cyan until the model is actually ready, then fades; so no gap or double image.

### 7.4 Model fleet (bundled, all CC BY 4.0, credited in the product and in a bundled attribution file) [OBSERVED-IN-SOURCE]
Nine third-party models ship with the product. Eight are used; the ninth (a low-poly cargo ship) is bundled and credited but nothing in the reference product loads it, so vessels are drawn as flat glyphs only. A clean-room rebuild should not assume vessels have 3D models.

All aircraft models are prepared to real-world metre size, share one nose direction, and are drawn at their natural size (no extra scale) when they have their own model.

| Aircraft class | Civil layer model | Military layer model |
|---|---|---|
| Helicopter | small light helicopter (Bell 206 JetRanger-style) | same |
| Light / general aviation | small single-engine (Cessna 172-style) | same |
| Business jet | small business jet (Cessna Citation-style) | same |
| UAV | MQ-9-style drone | same |
| Widebody | Boeing 787-9-style | same |
| Turboprop | ATR 72-600-style, flat abstracted colours (no textures) | same |
| Airliner, quad-jet, glider | one shared heavy-airliner model (a heavily simplified 747-style airframe), scaled by class: glider 0.75, airliner 1.0, quad-jet 1.45 | same shared model |
| Fast jet, unknown | shared airliner model, scale 0.8 for fast jet and 1.0 for unknown | a separate small jet model (a private-jet-style mesh) kept for these two classes, scaled like the class |

Notes: (a) the per-class scale factors only apply to classes that fall back to the shared model; classes with their own model are drawn at natural size; (b) in the military layer the amber tint dominates any paint scheme so military aircraft always read amber; (c) the model count budget, add/keep radii, pixel clamps and tint amount are in 7.2.

Classification of a plane comes from its ICAO type designator sets (helicopters, quad-jets, widebodies, turboprops, fast jets, business jets...), falling back to emitter category. Type/route enrichment from a community database is best-effort, rate-limited to a few requests per second, and swaps the glyph in place when it lands. [FROM-DOCS-ONLY for the rate]

### 7.5 Sensor interplay
In NVG/FLIR the models render white-hot (see 2.5); on style change models are re-created in batches. The reason is cost: dropping several hundred GPU-backed models in one synchronous handler stalls the render thread, so models are released 40 per tick with each plane showing its billboard first (the same rule applies to cockpit entry). Mechanism detail: 13b gap 14.

---

## 8. Performance and quality controls

### 8.1 Idle render governor
- The engine normally redraws every vsync. The product switches to **on-demand rendering** whenever nothing animates and returns to continuous rendering the instant any module registers a "hold". Holds are named owners (e.g. fleet interpolation, satellites, tracking, style animation, annotations, orbit, cockpit); an owner double-holding or double-releasing cannot corrupt state. In idle mode the camera or tile loads render automatically; every other change (slider write, layer tick, annotation change) requests one frame.
- Result per the project's own note: an idle globe with no layers went from burning roughly 60% GPU and over half a core to near-idle. Any scene with an animating owner behaves exactly like continuous rendering.
- Simulation-time changes never trigger re-rendering by themselves: the engine's own "re-render when simulation time changes" threshold is set to infinity so idle really means idle.
- **Governor mechanics:** holds are kept in an identity-keyed set (a double hold or double release cannot corrupt it); entering idle forces one settling frame; the governor is safe to call before a viewer exists (holds are remembered and applied at install); teardown clears all state. A one-frame render request is always forwarded to the engine, even in continuous mode, which closes the race between a request and the last release. A rolling log of the last 16 idle requests with reasons exists for diagnostics. Every per-frame animator (fleets, traffic, satellites, tracking, style crossfades, camera orbit, camera verbs, camera-feed projection, cockpit) holds for exactly the life of its animation. [OBSERVED-IN-SOURCE, 13a NAV-57 and NAV-58]
- **Detection and scope take no hold.** Detection repaints on change and asks for exactly one more frame only while work remains outstanding; the rules that make it free when idle are listed in doc 07 (zero idle cost). Measured: 0 renders per 5 s on a parked empty scene with detection on, against 301 with a hold. Aircraft brackets only stay prompt because the aircraft layers themselves hold the loop. [13f SYS-127]
- When a hidden tab returns, a visibility handler repaints once.

### 8.2 Throttles and caps (summary)
| Area | Value |
|---|---|
| Target frame rate | 60 |
| MSAA | 4 samples |
| Flights fleet motion tick | about 12 Hz (80 ms) |
| Detection label solve | every 125 ms |
| Cockpit cloud pass | 12 fps, internal buffer at most 520 x 320 |
| Celestial ring | 30 fps, 2 M pixel cap |
| Scope mask | repaint on quantised step only |
| Camera readouts (HUD) | every 250 ms |
| Model caps | 150 proximity / 350 all / 60 cockpit |
| Google tile cache | 1.5 GB + 1 GB overflow |
| Hidden browser tab | weather, clouds and celestial work pause; resume on return |
| Reduced motion | honoured by cloud pass, cyber sonar sweep, route dolly shaping |

### 8.3 FPS readout
Backtick toggles a small "FPS n" readout beside the title, counting rendered globe frames once per second (shows a dash when the tab is hidden). Ignored while typing or with modifier keys.

### 8.4 Measured baseline (one Apple M5, Chrome, 1440x900, 22 Aug 2026; not a minimum spec)
- Startup: app ready about 0.6 s, visually settled about 1.9 s, load event 0.44 s, 105 MiB heap, 60 fps motion and rest.
- Cold layer activation: camera feeds 19.6 s (48 sources), Space Missions 3.6 s, Radio 3.5 s, Bikeshare 2.1 s, Data centres 0.8 s (4,362), Flights 0.67 s (247), Submarine cables 0.61 s (2,629; the most memory at 412 MiB), Military flights 0.56 s.
- Detection: 25% about 39-41 fps (14 labels), 50% about 37-40 fps (28 labels), 100% about 34-36 fps (56 labels); Cockpit about 49 fps.
- Looks: Normal/CRT/NVG/Anime 60 fps; FLIR 49 motion / 60 rest; Noir 47/56.6; Snow 42/46 (Snow, Noir, dense detection and text-heavy stacks are the heaviest). Combined operational stress about 40-43 fps.
- Keyed sources (separate unfocused pass): fire detections 100k points about 32/55 fps; ship positions 12,000 vessels about 22/30 fps; road traffic 4,222 dots about 45/52 fps.
- Heavy stacks (camera feeds + flights + satellites + cables) can hang weaker browsers; the live walkthrough advises enabling layers one at a time. An embedded-browser shell was reported to stutter severely where an external browser did not.

### 8.5 Input quality
- Trackpad pinch (browsers report it as small ctrl-wheel events) is relayed as a zoom with an 8x multiplier capped at 120 pixels of delta per event. The relay intercepts the event in the capture phase and re-dispatches a marked synthetic wheel event on the canvas so it is not relayed twice; events not in pixel mode, with zero delta, or already relayed are ignored; if the engine already binds ctrl-wheel nothing is added; the hook is removed cleanly on teardown. [13a NAV-02]
- Zooming out is effectively unclamped (synthetic wheel input reached astronomical altitudes and a black screen in testing); the Reset Globe control is the recovery. Zoom-in minimum while following a tracked entity is 150 m. See doc 06.

---

## 9. Observed design language (words and approximate values)

### 9.1 Overall character
Dark, instrument-like "mission control" aesthetic: near-black space, frosted glass panels, thin hairlines, monospaced uppercase micro-labels with wide letter-spacing, cyan as the single dominant accent, and a deliberately military/surveillance flavour (classification-style banners, REC dots, bracket reticles). Tone is cinematic and slightly theatrical; every data source is labelled honestly (estimated, stale, fallback).

### 9.2 Palette roles (default theme)
| Role | Value (approximate) |
|---|---|
| App/page background | near-black blue #0a0a0f |
| Glass panel fill | rgba(12,12,20) at 72% opacity |
| Panel hairline border | white at 8%; hover white at 15% |
| Primary accent (cyan) | #00d4ff; dim fill at 15%; glow at 40% |
| Primary text | #e8eaed; secondary 50%; dim 30% |
| Opaque menu surface (native dropdowns) | #12121c |
| Detection/world-card plate | deep teal-black rgba(4,12,16) at 82%; selected 94% |
| World-card accent / selected border | #6be8ff / cyan at 72% |
| World-card title / detail text | near-white 96% / blue-grey 92% |
| Recording dot | red #ff3333 with glow |
| Detection toggle active | green border (0,255,80 at 60%) with 10% fill |

Semantic tier colours (default palette): civil air cyan #22e0ff, military amber #ffb347, sea teal #3fe0c8, space violet #bda4ff, vehicle slate #8fa6b4, traffic jam red #e05252, slow amber #f0b23e, free green #2ecc71, no-data grey #c9c9c9. Transit: bus green #5EF08A, tram amber #FFC24A, subway red #FF4538, rail lilac #D9A6FF, ferry sky #5FD6FF, unknown #D8DDE5. Trail colours: civil #00d4ff, military #FFB800, vessels #39ffd5, transit head green #5EF08A. Draw/annotation colours: primary #8be9ff, amber #ffb547, cyan #39d0ff, green #5dff9f, red #ff6b6b.

**HUD colour per look:** cyan 60% (Normal/Anime/Noir/Snow), green 80% (NVG), white 70% (FLIR), amber 80% (CRT).

**Cyber theme (alternative shell skin):** desaturated red #c45454 (bright #e16666) on slate (#26343a, raised #334a53), panel surface rgba(18,24,27) at 92%, telemetry fill almost opaque dark slate, grid lines at 10% pale blue, light-grey/white text, red hover borders, chamfered (cut-corner) polygon frames for panels, buttons and the bottom telemetry bar, panel widths 272 px (left rails), 312 px (right rails), 200 px collapsed. The whole shell (map and cockpit) changes skin when the Cyber HUD layout is chosen and reverts when it is left. Implementation facts worth copying if a skin is ever built: the decorative cut-corner frames are non-interactive pseudo-elements so popups are not clipped; the focus outline is inset and the red hover border survives clipping; the left rail is raised to clear the coordinate card; the right rail measures its rendered top. [13f SYS-105]

### 9.3 Typography
- Two typefaces: **Inter** (weights 300-600) for conversational text and **JetBrains Mono** (300-700) for almost everything else. Fallback stacks are system monospace/sans.
- Micro-labels are 8-10 px monospace, uppercase, letter-spacing 1.4-1.8 px (status words, "Style" labels); panel titles 10-12 px tracked 2-3 px; HUD readouts 11-13 px at 1.2-2.5 px tracking; mode label 16 px bold at 4 px; the cockpit and world cards use 10-13 px mono (card titles 12-13 px semibold, details 10-11 px).
- Icons: Google Material Symbols Outlined (a pinned subset of about 35 glyphs: radar, layers, flight, navigation, public, draw, flare, etc.), plus a few emoji glyphs on the preset tray buttons.

### 9.4 Shape and surfaces
- Corner radii: panels 16 px; buttons 10 px; chips/pills fully round; inner controls 3-8 px; world-overlay cards 4 px.
- Glass: translucent dark fill + backdrop blur 24 px with 1.4x saturation (16-28 px variants in other places), inner 1 px highlight line at the top edge (white 6%), soft drop shadow (0 8 32 black 50%), and a faint cyan radial "glow" bleeding below floating bottom panels.
- Buttons in grids (look tray) are tiles with an icon, a label and a key hint; hover lifts 1 px and brightens border; active gets cyan border, cyan 15% fill and a soft cyan outer glow with a glowing icon.
- Panels are collapsible with a header that stays visible while the body scrolls; collapsed panels show a compact title and a one-line mini-status (e.g. current style, current location/landmark). Panels can be pinned open, dragged (portable panels), docked, resized by edge/grip, and some themes allow only one expanded at a time. Left rail and right rail measure available height and share it.
- Layering ladder (bottom to top, approximate): scene; scope mask 1; celestial ring and HUD 2 (map mode); detection blend surface 5; detection/world cards and cockpit cloud layer 6-7; credits 90; panels 100-139 (promoted floating panels up to 139); bottom dock 145 and the cockpit first-person overlay 145; cockpit-mode HUD text 146; cockpit-mode left rail 147; voice pill 150; credit lightbox 200; toasts 200; clean-view exit 300. So in map mode the detection brackets and cards paint above the HUD text and below every panel. [OBSERVED-IN-SOURCE; the HUD-versus-detection order is INFERRED from these values]
- Fonts and icons are fetched from a public font service at page load (not bundled), so a fully offline or blocked-network session falls back to system monospace/sans and loses the icon glyphs; a clean-room build should self-host. [OBSERVED-IN-SOURCE]
- HUD and detection text carry a soft glow (about 4 px, in the look's colour) and are upper-cased by style, not by content. [OBSERVED-IN-SOURCE]
- Focus: an app-wide 2 px light outline with 2 px offset on every interactive element (inset inside scrolling rows), never removed by local styles.

### 9.5 Motion
- Standard transitions 150 ms (hover/press) and 300 ms (layout), both with a standard ease curve; opacity fades 180 ms; style crossfade 500 ms; camera flights 1.4-4.2 s ease in/out; bottom-dock reveal animation on startup; numeric cockpit values "roll" digit by digit (rate-limited to once per 220 ms); REC dot blinks every 800 ms; HUD summary types in two characters every 24 ms; cyber sonar sweeps a full turn every 5.2 s; live indicator pulses 1.6 s; detection fade-in on enabling; everything animated honours the operating-system reduced-motion setting where implemented.

### 9.6 Layout anatomy (desktop)
Brand title top-left with a small gaze-following logo; "active style" indicator; a compact centre-top action cluster (clear layers, copy share link, tilt, north-up, reset globe); a left layer rail; a right rail (Display, camera feed, Context, Weather, Recent imagery, Street level, Scenes); a bottom-centre command dock (Location tray, voice, Visual Presets tray); bottom-left credit line; bottom-right voice pill; toast notices centre; global loading/sync chips top-centre. Below 980 px width the dock reflows to two rows; below 575 px the credit lightbox becomes full-screen. Mobile is secondary (desktop-first density).

**Narrow-width breakpoints (width in CSS pixels)** [OBSERVED-IN-SOURCE for the rules; the visual outcome is INFERRED]:
- 980: the bottom dock becomes two rows (location and voice on top, presets below) and spans the width minus 24 px.
- 900: dock popovers span the width minus 24 px; open trays take nearly the full dock width and their siblings shrink to icon-width; the attribution line shifts to a 12 px left inset and may use the full width minus 24 px.
- 720: the Display panel narrows to about 176 px and joins the stack in normal flow; the top action cluster moves to the top-right corner (16 px inset); the active-style indicator disappears; the title shrinks (about 16 px type, 36 px logo) and the FPS readout moves under it; the location tray and voice pill move to 16 px side insets; the HUD loses its two vertical edge texts and its top and bottom bars are re-anchored to the screen width minus 24 px (Cyber hides both bars entirely).
- 620: dock labels truncate to about 2.3 rem and the left stack's bottom limit moves to just above the vertical middle of the screen.
- 520: the title's second word and the subtitle are hidden, leaving the logo and first word.
- Cockpit at 760 or less: the map HUD text, the left and right context windows and the map rails are all hidden; the cockpit's own utility buttons move to a bottom-right row above the dock position; the exit/reset buttons sit centred near the bottom.
Touch handling found: panel hover-to-open behaviour and floating-panel dragging/resizing are limited to mouse and pen input (touch must use the header buttons), and the logo's gaze-following is off for touch. No other touch gestures exist beyond the 3D engine's own. Hover-only tooltips have no touch equivalent. [OBSERVED-IN-SOURCE]

---

## Kawn notes

**Keep**
- Source ladder with graceful fallback and honest unavailability reasons; the single-active-source model with generation counting.
- Photoreal 3D as the hero mode for Dubai/Abu Dhabi/Sharjah (skyline and Palm are the demo shots); keep the Esri/OSM keyless safety net.
- Idle render governor and named-hold pattern; hidden-tab pause; the FPS readout.
- Sharpen on by default, bloom off; 500 ms look crossfade; per-look parameter sliders.
- Scope/keyhole as a signature visual with an altitude-adaptive edge.
- Proximity/All 3D model caps and hysteresis (150/350, 150 km/400 km radii) as a starting point.
- Required-attribution rules (always visible, even in clean view and recording); credits live in a dedicated container outside the clean-view hide rules.
- The four-places rule for defaults and the "old links keep the era's default" parse rule, so a Kawn default change never silently alters an old shared link.
- Apple Metal fog workaround if iPad or iPhone Safari is in scope (probe plus platform backstop; accept losing distance fog there).
- Version the bloom/look scales in links from day one.

**Owner decisions applied (docs/09, 2026-10-16)**
- First load: Scope on, feather 100%, Dubai, top-down at city scale; one tap to switch Scope off and a visible feather control remain. The reference's final 11% default is not copied.
- Basemap: Photoreal 3D over Dubai, Abu Dhabi and Sharjah is the hero; coverage quality there is UNVERIFIED and should be checked before launch.
- No sun shading in the reference: for the UAE decide whether Day/Night looks include a Gulf-time terminator (later) or are pure colour grades (first release).

**Skip**
- Fake classification banners, MGRS, "NIIRS" and spy-satellite framing; ship none of the military cosplay.
- CRT/NVG/FLIR as the primary identity; at most one "night / heat" look pair for operations use.
- Snow look (irrelevant for the UAE; also the heaviest look).
- The Cyber red/slate skin as a default; keep at most one optional alternative theme.
- Bing sources unless a licence exists; the reference needs a vendor token for them.

**Adapt**
- Looks for UAE use: Day, Night (low-light with warm lamps), Heat/Comfort (a thermal-style ramp used for heat-island and pedestrian comfort overlays - FLIR ironbow is the natural starting point), Dust/Haze (sandstorm reduced-visibility), and a Print/High-contrast look for government briefs.
- Extreme-heat reality: the reference's cool-cyan-on-black palette suits night; add a bright-daylight-legible theme (paper-white panels, dark text) because many users view on phones outdoors.
- Terrain: Gulf terrain is flat; the keyless terrain service adds little there, so default to imagery + building footprints and keep the ground-height sampler for tower roofs.
- Type: pair an Arabic-capable face (e.g. a UI Arabic family) with the Latin mono/sans; monospace uppercase micro-labels do not translate to Arabic (no case, joined letterforms) - replace tracking-heavy micro-labels with weight/size hierarchy for Arabic and avoid letter-spacing on Arabic text.
- RTL: mirror left/right rails, dock order, sliders and arrows; the circular scope and compass remain geographic (not mirrored). Use Western or Eastern Arabic numerals by user setting.
- Mobile-first: replace floating glass rails with bottom sheets; keep glass styling but raise opacity for sunlight legibility; 44 px targets.
- Licensing: model assets in the reference are CC BY 4.0; Kawn needs its own credited or purchased assets, with UAE-relevant types (A380/777/787 widebodies for DXB, helicopters for police and medical, business jets for DWC/NHD).
- Budget caps (labels, models) should be tied to a device-class detection rather than altitude alone.

## Open questions / UNVERIFIED
Resolved in the revision pass and moved into the body: Escape and Clean UI (Escape does not exit it); the voice pill in Clean UI (stays visible); the HUD auto-hide reachability; the celestial ring end altitude (window-dependent); the look-versus-sharpen order (looks, then sharpen); the vessel 3D model (bundled but unused); mobile/touch behaviour of panels (mouse and pen only for hover and drag).
- Where the engine's own bloom stage sits relative to the product's look and sharpen stages (decided inside the 3D engine library; not stated in the reference).
- Whether the 3D engine applies any anti-aliasing post filter of its own on top of the four-sample multisampling (the product does not switch one on).
- Exact visual appearance of Bing sources (no ion token was available in the observed session), and whether the Bing path behaves identically on a hosted HTTPS origin as on plain-HTTP localhost.
- Whether any part of the page uses the sun position for shading (none found by search).
- Which of the two Cyber sonar implementations (a graphics-card path and a scene path) is used when.
- How the sensor-boost affects non-aircraft layers beyond the notification (vessels, satellites); only a notification is sent, each layer decides.
- Photoreal attribution wording and Google logo handling beyond the credit line (the engine owns the credit text).
- Final on-screen position of the minimal-layout HUD blocks and the credit line at dock-reflow widths: several stylesheet rules compete and the later-loaded ones appear to win; not confirmed visually.

## Revision log
- Corrected the 3D model table: eight of nine bundled models are used; the cargo ship is bundled but never loaded (vessels are flat glyphs); added the civil versus military model mapping and clarified that per-class scale factors apply only to classes sharing the common airliner model.
- Rewrote Clean UI: full hidden list (including dock, rails, setup prompts, first-run card, reticle), exit pill details, Escape does not exit, voice pill stays, attribution behaviour, not saved in links, embed mode relationship.
- Added the post-process stage order (looks in a fixed sequence, then sharpen; engine bloom position unverified; nothing else enabled) and the list of layers that sit outside the chain; added click correction for transit picking under NVG/FLIR distortion.
- Celestial ring: documented that it switches itself off when the globe is not framed, resolved the end altitude question, and noted sun/moon glyphs.
- Added narrow-width breakpoints for the shell, a refined layering ladder (HUD versus detection), external font loading, and touch-handling facts; resolved the HUD auto-visibility question.
- Re-verified (unchanged) map-source ladder, terrain retry numbers, look parameters and tactical presets, sharpen/bloom maths, scope and feather numbers, model caps and radii, idle governor, atmosphere settings and design tokens.

### Pass 3 (census merge: 13a, 13b, 13e, 13f)
- Correction: the Google token renewal is a per-request single retry after an unauthorised or forbidden answer (replacement must differ from the failed token; renewals shared), not "after two refusals" (13a correction 6).
- Added: viewer set-up with all stock widgets and the default base layer off, and the fog "renderable" flag detail of the Apple Metal workaround with its probe, backstop and cost (13a corrections 7; 13f gap 7).
- Added: statement that no sun-driven lighting or terminator exists and the sun is used only by the celestial ring and HUD (13a correction 9); celestial ephemeris technique.
- Added: map controller rules (silent switches, origin tag, unchanged provider keeps layer, bottom-of-stack), chip-follows-displayed-source, Bing-via-ion procedure, share-link map lane, token terrain water-mask note, retry bound.
- Added: governor mechanics (identity-keyed holds, settling frame, forwarded one-frame requests, 16-entry log), detection and scope taking no hold, hidden-tab repaint; pinch relay edge cases.
- Added: where a default lives (four places plus a deliberately different parse fallback) and what persists (13f gap 11); bloom scale migration on every ingestion path; look shader facts; the unverified 1.2 first-run pixelation note.
- Added: cockpit near band equals model admission radii, shared pip texture, billboard base size and limb recession (13b AIR-88, section 1.4); model batching rationale.
- Added: Kawn owner decision (Scope on, feather 100%, Dubai) and UAE notes; confidence tags updated to cite census sources where touched.

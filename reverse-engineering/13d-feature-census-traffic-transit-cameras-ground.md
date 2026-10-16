# 13d - Feature census: ground-level live data (traffic, transit, bike share, public cameras, ALPR map, street-level imagery, routing, recent imagery, radio, mapped installations)

ID prefix: **GND**. Clean-room: behaviour and technique only, in plain words. No source identifiers, route names, env-var names or code. Numbers are quoted because they are behaviour (caps, thresholds, cadences), not code.

## Purpose and method

Purpose: an independent, exhaustive list of every ground-level feature of the reference app (the studied original), a plain-words explanation of how each works, and an audit of how well our reverse-engineering notes (00-12, specs 01-04, docs 05) already cover it.

Method, in words:
1. Enumerated from the reference app first, before reading our notes: the README feature table and walkthroughs, the data-sources list, the current-state log, known-issues list, the tool catalogue, the voice action schema, the panel markup templates, the per-layer source folders (traffic, transit, bike share, cameras, ALPR, street level, directions, recent imagery, radio, installations, local receivers, SDR) and the server provider modules, plus the names of the unit tests (they list intended behaviours).
2. The code-graph tool was attempted conceptually for execution flows (traffic load, camera activation, route flight); in this run the answers were taken by reading the layer folders directly, because each layer is split into small single-purpose modules whose header comments state their job. Where a flow crosses modules (e.g. flow tiles -> road matching -> dot spawn -> animation) it was traced by reading each hop in order.
3. Only then compared with our notes: doc 03 sections 8-10 (transit, bike share, street traffic), doc 04 sections 4-10 (installations, ALPR, cameras, recent imagery, street level, radio and local SDR, directions), doc 08 (voice tools, MCP tools, public-camera catalogue rules), doc 10 (backend), doc 12 (URLs), doc 02 (share state).
4. Each feature is marked DOCUMENTED (doc + section), THIN (present but a mechanism or number is missing), or MISSING.

Result in one line: our notes are already very strong on this domain (almost every row is DOCUMENTED). The real gaps are in *mechanism* detail: how flow is snapped onto roads, how the simulated vehicles are placed and moved, how the hybrid road source removes duplicates, how the route flight is tuned, how the camera pyramid/plane geometry is computed, and a handful of small behaviours listed in "Gaps found".

Legend for Kawn verdict: KEEP = build as is; ADAPT = build with changes; LATER = good but not first release; SKIP = do not build (privacy/legal/irrelevant). UAE relevance is noted per row group.

---

## How the two hardest ideas work (plain words)

### A. Live street traffic: why the dots look like real traffic

1. **Road geometry comes from a map, not from sensors.** When the camera is below 8 km altitude the layer asks for road lines for the area the user is actually looking at (the point the camera aims at, pulled back toward the spot directly under the camera if the user is gazing at the horizon, never more than 12 km away). Roads come from free vector map tiles (OpenStreetMap-derived) and, if the operator has a TomTom key, also from TomTom's traffic tiles. The user can pick the road source: TomTom only, OpenStreetMap only, or Hybrid.
2. **Two passes keep it fast.** First only major roads are fetched and drawn (always so above 4.5 km), then the full road network for the view. A re-fetch is skipped when the new view overlaps the old by more than 60% and the centre moved less than about 350 m. The first dot paint waits at most a quarter of a second for traffic data: if it is not cached it paints plain white dots immediately and recolours them when the data lands.
3. **Where the colours come from.** TomTom publishes, per road segment, the ratio of current speed to free-flow speed (1 = free). The layer buckets it: 0.85 and above = free (green), 0.55 to 0.85 = slow (amber), below 0.55 = jam (red). Closed roads carry a closure flag and spawn no cars at all. Segments with unusable data are treated as free flow, never as a phantom jam.
4. **Snapping flow onto OpenStreetMap roads (the hidden core).** TomTom's lines and the map's lines are cut differently, so they cannot be joined by id. The layer snaps them geometrically: every TomTom line is chopped into short pieces and indexed in a grid of cells about 100 m wide. Each map road is sampled at up to 7 evenly spaced points; each sample looks for the nearest TomTom piece within 35 m whose compass direction agrees with the road's direction of travel to within 30 degrees (two-way TomTom lines count for both directions). A road "matches" if at least half its samples (minimum 2) found a piece; its congestion value is the *median* of the matched samples (so one bad piece cannot paint a whole road), and it is flagged closed if any matched piece is closed. Roads that do not match stay white "simulated" (or are hidden if the operator chose the strict "hide uncovered roads" view). Honesty rule: colour = real data, white = simulation.
5. **Hybrid removes duplicates.** In Hybrid the TomTom line wins wherever it exists; the map road is kept only where TomTom has none. Overlap is tested by sampling the map road every 10 m against TomTom lines (35 m tolerance; but only 15 m when one is a motorway/trunk mainline and the other is not, so service roads that run 20-40 m beside a freeway are kept). A partly covered road keeps only the uncovered stretches that are at least 40 m long. Non-drivable categories (walking, parking, private) never become vehicles and never suppress map roads.
6. **Cars are simulated, not tracked.** Individual dots are not real vehicles. Each road gets a number of dots from its length: spacing between dots is 30 m when the camera is within 1 km of the ground, 80 m within 3 km, 150 m within 5 km, 250 m above; multiplied by a road-class density factor (motorway 3.0 down to 0.4 for minor roads) and, in live mode, by a congestion factor (slow roads get up to 2.5x more cars, up to 4x in the optional dense-jam mode). Dots are placed at random positions, each with its own speed = road-class speed x a random 0.7-1.3 factor x a congestion factor that never drops below 0.15 (jams crawl, never freeze). Direction alternates forward/backward to mimic two-way traffic, or follows the legal direction on one-way roads.
7. **Animation is cheap on purpose.** Each road's path is converted once to a list of 3D points with precomputed segment lengths. Every frame, each dot advances by speed x elapsed time (elapsed time capped at 0.1 s so returning from a background tab does not teleport cars), carrying leftover metres across corners, then is placed by straight-line interpolation between the two path points around it (no trigonometry per frame; objects are reused to avoid garbage). A dot reaching the end of a road shrinks away, then re-enters at the start as a *new* vehicle with a new identity. New dots fade in and retiring dots fade out. Near both ends of a road a car has a small chance (0.8% per segment entry) to pause 2-6 s, imitating a red light.
8. **Jam effects (live mode only).** In dense-jam mode, cars on a red road spawn as bumper-to-bumper platoons (queues of 4-8 cars, 6-12 m apart, each queue sharing one direction), and move stop-and-go: creep forward 1.2-3 s at 2.2x speed, then stand 1.5-5 s, which keeps the long-run average near the honest crawl speed. Optional heat lines draw up to 400 glowing corridors (jam 9 px, pulsing about every 1.6 s via one shared material; slow 4 px, faint).
9. **Sitting on the ground.** Dots are lifted a few metres above the sampled surface (the 3D city mesh or terrain), heights are eased in rather than popped, and dots are budgeted: at most 6,000 at close range, shrinking roughly with the square of altitude (never below 400); long roads are sub-sampled to 80 points; a fair per-road allocation prevents big roads from starving small ones.
10. **Styled looks.** Under night-vision, thermal, noir and retro looks the colour palette would be destroyed (night vision turns everything to brightness), so dots become bright white cores with a thin dark halo (jam dots a pixel fatter) in mono looks, or saturated primaries with larger size in the retro look; a kill switch can force the normal palette.
11. **Sources and costs.** TomTom tiles pass through a server proxy: the key never reaches the browser, tiles are cached 2 minutes (memory + disk), one fetch per tile at a time, and a persistent daily request budget (default 6,000, sized to a 200,000/month free allowance) after which stale tiles are served or an explicit "budget" error is returned.

### B. Public camera "projection": how a flat CCTV picture appears inside the 3D city

Think of each camera as a **pyramid of sight** and a **TV screen floating at its base**.
1. **A camera pose is an estimate.** Cities publish a position, sometimes a facing direction, almost never the tilt, zoom or mounting height. The app invents a "raw prior" pose: a facing (from the feed, or a fixed pseudo-random bearing derived from the camera's id and *marked low-confidence*), a downward pitch, a horizontal field of view and a range (how far the picture plane stands from the lens), plus a mounting height.
2. **The pyramid.** From the lens point, four edges run out to the corners of a rectangle at the "range" distance. The rectangle's width follows from the field of view (half-width = range x tan(half FOV)); its height follows from a 16:9 picture shape; it is tilted by the pitch. That is the **coverage cone / frustum**: five points (lens + four corners). The coverage mode draws these as wire edges (always for the active camera, plus the nearest 14 neighbours within 1.8 km), the **viewshed** mode fills the same five points as a translucent coloured solid, each camera in its own hue (hues are spaced by the golden angle over the sorted catalogue so neighbours always differ). The solid is built from the very same five points as the wires, so wires and fill can never disagree.
3. **The screen.** The latest picture from the camera is painted onto a 1920x1080 canvas and shown on a flat rectangle exactly at the far end of the pyramid, so the picture appears to hang in the scene "like a hologram". Only one such screen exists at a time (the active camera); idle neighbours get faint wires only.
4. **Not burying the screen in the city.** Three safeguards: (a) when a camera becomes active, one test ray along the pyramid's axis finds the first building/terrain hit and shortens the range to 4 m before it (never below 12 m) so the screen does not end up inside a wall; (b) the whole rectangle is lifted rigidly by the smallest amount that clears the ground under a 3x3 grid of support points (lifted by at most 60 m, so a tall building does not push it into the sky); (c) the screen centre never sinks below 2 m above ground, so a steep pitch cannot bury it.
5. **Ground height is resolved once, not every frame.** Each camera is grounded once (from a terrain-height lookup, then refined from the rendered 3D mesh when tiles have finished loading, with precomputed heights for ~3,445 cameras shipped so most placements need no sampling). After that it is frozen until the user moves or calibrates it. Work is drained in small batches (4 cameras every 120 ms; 2 every 250 ms while tracking or cockpit owns the view) so enabling the layer never freezes the frame rate.
6. **Keeping the picture fresh without flicker.** The active picture refreshes every 10 s (idle 60 s). A tiny 64x36 fingerprint of each new frame is compared with the last; if identical the texture is not re-uploaded. The picture texture is swapped at most once a second between two alternating canvases (the 3D engine only re-uploads when it gets a *new* image object, and redrawing in place is invisible to the GPU). Live video (HLS) uses a decoder whose output the engine reads natively each frame.
7. **User correction (calibration).** Because poses are guesses, ADJUST puts a manipulator on the active camera: rotate rings for heading and pitch, arrows for east/north/up, a handle at the screen centre for range, handles on its side edges for field of view. Each drag is converted to one of 7 offsets applied on top of the original estimate. Edits are live but unsaved; SAVE stores them in the browser as a manual calibration, RESET discards.

---

## Feature table

Columns: ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE relevance.

### 1. Street traffic

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| GND-01 | Street Traffic layer toggle (off by default) | Row in Data Layers; dots appear on roads when closer than 8 km | See explainer A. Layer idles above 8 km; honest "simulated" disclosure | OSM vector tiles; optional TomTom | doc 03 s10 DOCUMENTED | ADAPT; UAE roads need a UAE-capable flow provider |
| GND-02 | Road source chips: TomTom / OSM / Hybrid | Three chips; tooltips say which need a key | Mode resolved against key availability: no key -> OSM always; with key -> Hybrid default, explicit choice wins; a share-link/voice/tool choice beats saved value | Same | doc 03 s10 DOCUMENTED | KEEP |
| GND-03 | Two-pass road load (major then full) | Major roads appear first, detail fills in | Major-only above 4.5 km; second pass for all classes; failures in pass two keep pass one and say coverage is reduced | OpenFreeMap tiles | doc 03 s10 DOCUMENTED | KEEP |
| GND-04 | Viewport targeting / horizon-gaze pull-back | Roads load where the user looks, not tens of km away | Fetch centre taken from the look-at ground point, pulled toward nadir beyond 12 km | n/a | doc 03 s10 DOCUMENTED (centre-pull) | KEEP |
| GND-05 | Debounce, overlap and shift thresholds | No refetch on tiny pans | 320 ms settle; skip if >60% overlap and <0.35 km shift; tile cache 64 entries | n/a | doc 03 s10 DOCUMENTED | KEEP |
| GND-06 | Live flow colouring (green/amber/red) | Coloured dots with real congestion | Flow ratio buckets 0.85 / 0.55; 90% alpha | TomTom flow tiles (relative style, zoom 12) | doc 03 s10 DOCUMENTED | ADAPT |
| GND-07 | Flow -> OSM road snapping (35 m, 30 degrees, 7 samples, median) | Colours land on the right road and direction | Explainer A.4 | Same | **MISSING** (doc 03 says "matched" only) | KEEP - fill in Gaps G1 |
| GND-08 | Directional flow (per travel direction) | Opposite lanes can differ | Two-way roads are modelled as two travel directions; a TomTom line marked "full coverage" serves both | Same | THIN (doc 03: "Live flow is directional") | KEEP - G1 |
| GND-09 | Hybrid de-duplication | No double roads where both sources have a line | Explainer A.5 (10 m sampling, 35/15 m tolerance, 40 m minimum fill) | Same | **MISSING** | KEEP - G2 |
| GND-10 | Road-class filtering of flow segments | No dots on footpaths/parking | Only recognised drivable flow classes (motorway ... minor local) become roads; unknown/private/walking ignored | Same | **MISSING** | KEEP - G2 |
| GND-11 | Closure handling | Closed roads show no cars | Closure flag decoded; zero dots spawned; counted as closed in flow tool | Same | THIN (doc 03 mentions closed in tool only) | KEEP |
| GND-12 | Uncovered-roads policy: simulate (default) or hide | White ambient dots on roads without data, or none | Param applies on next camera-driven load; programmatic/tool only | n/a | **MISSING** | KEEP - G3 |
| GND-13 | Dot spawn count by altitude and class | Denser when close | Explainer A.6 (30/80/150/250 m spacing by clearance) | n/a | THIN (doc has class multipliers, not spacing) | KEEP - G3 |
| GND-14 | Dot speed model and noise | Plausible varied speeds | Class speed x 0.7-1.3 random x congestion (>=0.15) x user speed scale 0.3-3.0 | n/a | DOCUMENTED (class speeds, scale range) | KEEP |
| GND-15 | One-way roads | Cars only go the legal way | Direction from the road's one-way flag, else alternate | OSM tags | **MISSING** | KEEP - G3 |
| GND-16 | Traffic-light pauses | Cars occasionally stop near junction ends | 0.8% chance per entry into an end-segment, 2-6 s pause | n/a | **MISSING** | ADAPT - G3 |
| GND-17 | Fade in/out and recycle at road end | No popping | Size ramps in at birth, out on retire and near road end; recycle = new identity | n/a | THIN | KEEP - G3 |
| GND-18 | Frame-time cap | No jump after background tab | Delta time capped at 0.1 s | n/a | **MISSING** | KEEP |
| GND-19 | Dot budget and fair allocation | Dense but smooth | 6,000 cap scaled by altitude-squared law, min 400; fair per-road share | n/a | DOCUMENTED | KEEP |
| GND-20 | Surface alignment | Dots ride on road surface in 3D city | Heights sampled from rendered mesh (or terrain), eased; unique height stations bounded per pass | Google 3D / terrain | THIN | KEEP |
| GND-21 | Jam density mode: platoons + stop-and-go | Red queues look like queues | Explainer A.8 | n/a | THIN (doc: "denser jam mode"; creep timings present) | KEEP - G4 |
| GND-22 | Heat-line corridors (jam pulsing, slow faint, 400 cap) | Glowing red/amber ribbons | Polylines with shared pulsing alpha; rebuilt on restyle | n/a | DOCUMENTED | ADAPT |
| GND-23 | Jam-visualisation mode param: none / density / heatline / both | Owner A/B switch | Applies on next load | n/a | DOCUMENTED (partly) | LATER |
| GND-24 | Style-preset dot treatment (mono / retro / normal) | Dots stay visible under NVG/FLIR/CRT | Explainer A.10; sim (white) dots never restyled | n/a | THIN (doc 03: "coloured dots in sensor looks >=5 px") | KEEP - G5 |
| GND-25 | Sync chip and status text | "Syncing flow - Roads: ..." chip, split-flap text | Chip shows while busy, flashes 1.5 s after load; text names road source, flow, unmatched = simulated | n/a | DOCUMENTED (doc 03 failure text; doc 04 s6.2 chip pattern) | KEEP |
| GND-26 | Failure honesty | Distinct reasons | Key missing 503, budget 429, upstream 502/504, road tiles rate-limited / timed out / unavailable | n/a | DOCUMENTED | KEEP |
| GND-27 | TomTom proxy + budget governor | Invisible; protects free tier | Explainer A.11 | TomTom | DOCUMENTED (doc 03, 10, 11) | ADAPT |
| GND-28 | Flow tool for assistants | "How is traffic in X?" | City-sized area summary: speed vs free flow, congested and closed road | TomTom | DOCUMENTED (doc 08, 10) | KEEP |
| GND-29 | DETECT overlay feed from dots | Brackets on cars with congestion tier colour | Deterministic stride sampling of dots with seed; ids "vehicle-n" | n/a | DOCUMENTED | LATER |
| GND-30 | Attribution | "Traffic flow data (c) TomTom" only in live mode; OSM credit while shown | Credits service | n/a | DOCUMENTED | KEEP |
| GND-31 | Known issue: late fill in dense cores | Roads appear late after fast pans | One viewport tile at a time; proposed fixes: visible-first, neighbour prefetch, adaptive cap, multi-phase chip | n/a | DOCUMENTED | note |
| GND-32 | Dev-only timing diagnostics | none in production | Stripped from production builds | n/a | MISSING (irrelevant) | SKIP |

### 2. Transit

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| GND-33 | Transit layer toggle | One switch; buses/trams/metros/trains/ferries as mode-coloured icons | Active below 3,000 km altitude with +/-150 km hysteresis | GTFS-Realtime VehiclePositions | doc 03 s8 DOCUMENTED | LATER; needs RTA / ITC feed |
| GND-34 | Feed registry (7 regions) | Boston, Austin, Minneapolis-St Paul, Helsinki, Netherlands, Norway, South East Queensland | Server fetches only registered feeds; clients send ids; admission rule: keyless, open licence, real coordinates; each feed has a "default enabled" decision recorded with the terms passage | Operator feeds | DOCUMENTED | ADAPT |
| GND-35 | Mode inference from route id | Correct mode colour | Per-operator rules (MBTA colour names/CR-/Boat-; Entur codespaces; HSL numbering) refine feed default; a defaulted mode is flagged "inferred" | n/a | THIN (doc: "mode silhouettes") | KEEP - G6 |
| GND-36 | 15 s polling, nearest feeds only | Near-camera feeds poll | Range + 40 km slack so edge feeds do not flap; debounce 340 ms; camera change threshold 5% | n/a | DOCUMENTED | KEEP |
| GND-37 | Server snapshot cache, conditional requests, back-off | Invisible resilience | 15 s memory cache, serve-stale to 10 min, limiter, back-off ladder to 5 min | n/a | DOCUMENTED | KEEP |
| GND-38 | Delayed playback (interpolation between real reports) | Smooth, never-too-fast motion | Display clock runs a feed-derived lag behind real time (95th percentile of observed report lateness plus 5 s, clamped; min 25 s, reset after 90 s silence) and interpolates between the two bracketing reports at real speed | n/a | DOCUMENTED | KEEP |
| GND-39 | Plausibility filter | No teleports | Fix implying faster than mode ceiling (bus 130, tram 100, subway 150, rail 400, ferry 90, unknown 200 km/h) refused; outliers quarantined, replacement path needs two coherent observations | n/a | DOCUMENTED | KEEP |
| GND-40 | Course smoothing and STOPPED logic | Icons turn smoothly | 120 deg/s limit; course trusted after 12 m; STOPPED after 2 identical fixes; motion "unknown" after 90 s | n/a | DOCUMENTED | KEEP |
| GND-41 | Time-source flags | Honest age | Each fix records whether time came from vehicle, feed or receipt | n/a | **MISSING** | KEEP - G6 |
| GND-42 | Stale/evict policy | Dead feeds empty out | 2 missed polls; fix age >10 min removed; feed stale 90 s; evict 5 min | n/a | DOCUMENTED | KEEP |
| GND-43 | Fleet cap 15,000 | n/a | Hard cap across feeds | n/a | DOCUMENTED | KEEP |
| GND-44 | Off-view vehicles skip animation | Smooth at street level | Only vehicles within view padded by 60% are glided; others jump to newest fix; visibility sweep every 250 ms; rotation recompute every 200 ms | n/a | DOCUMENTED | KEEP |
| GND-45 | Ground anchoring | Vehicles on streets | Ellipsoid above 60 km; below, up to 300 ground cells warmed per poll, re-read up to 6 times at 0.9 s | Mesh / DEM | DOCUMENTED | KEEP |
| GND-46 | Selected-vehicle trail | Click a vehicle: 15 min trail | Draped corridor, mode colour (white in mono), <=640 vertices (~every 25 m), age fade | n/a | DOCUMENTED | KEEP |
| GND-47 | Server trail history (Boston only) | Reload recovers trail | 15 min / 128 fixes per vehicle, 16 MiB per feed, 64 MiB total, memory only | MBTA | DOCUMENTED | LATER |
| GND-48 | Selection card | Route, mode, operator, motion, delay, age, speed, status, id | 4 Hz text refresh | n/a | DOCUMENTED | KEEP |
| GND-49 | DETECT labels for transit | Route + mode + motion | Shared overlay | n/a | DOCUMENTED | LATER |
| GND-50 | Mode icons and look variants | Silhouettes by mode | 20 px (30 selected) | n/a | DOCUMENTED | KEEP |
| GND-51 | Transit tool for assistants | "Buses near X" | Area + optional route | Same | DOCUMENTED (doc 08 s9 removal from MCP) | LATER |
| GND-52 | Operator credits while in use | Courtesy attribution | Credits service | n/a | DOCUMENTED | KEEP |

### 3. Bike share

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| GND-53 | Bikeshare layer | Station dots coloured by availability | Active below 50 km (+/-2 km); cities within ~100 km fetched | GBFS | doc 03 s9 DOCUMENTED | SKIP/LATER (no UAE GBFS known) |
| GND-54 | 32-system registry | US cities (Lyft family, BCycle, others) | Static registry with per-system radius | GBFS | DOCUMENTED | ADAPT |
| GND-55 | Info once, status every 60 s | Fresh counts | Station info cached per city | GBFS | DOCUMENTED | KEEP |
| GND-56 | Colour/size encoding | Green >60%, amber 30-60%, red <30%, grey no data, dark offline; size 4-12 px by capacity | n/a | n/a | DOCUMENTED | KEEP |
| GND-57 | Selected card, warnings, DETECT, 8,000 cap | Station name, bikes/docks, warnings | n/a | n/a | DOCUMENTED | KEEP |
| GND-58 | Proxy hardening | Invisible | Host allow-list, no redirects, 5 MiB, 12 s | n/a | DOCUMENTED | KEEP |
| GND-59 | Bike-share tool | "Bikes near X" | Registry + source | n/a | DOCUMENTED (doc 08) | LATER |

### 4. Public cameras (CCTV)

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| GND-60 | CCTV layer toggle and enable choreography | Panel button CCTV OFF/ON; first enable shows "loading frames n/total" then "camera grid ready" | Catalogue loaded, camera icons created, ground geometry drained in small batches, nearest-first, active camera first | Per-agency catalogues | doc 04 s6.2 DOCUMENTED | ADAPT (privacy) - UAE law restricts aggregation; skip public CCTV, keep technique |
| GND-61 | Camera catalogue assembled from "packs" | About 3,900 cameras | Each agency pack fails independently; shared ceiling filled round-robin so no region starves; duplicate id = last pack wins so operator overrides replace live records; catalogue cached 15 min, single-flight refresh, stale-on-empty | Austin, Texas, California, London, Ontario, Finland, British Columbia, Estonia (Tallinn + road-weather), New South Wales, Calgary, Norway, Delaware (video), Warendorf, opt-in New York | doc 04 s6.1 DOCUMENTED | ADAPT |
| GND-62 | Per-pack rules (caps, anchors, status filters, heading source) | Different regions show different counts | See doc 04 s6.1; plus Texas defaults to Austin and San Antonio districts and ranks by nearest of several anchors; California elevations are in feet and clamped; image hosts pinned to the official origin | Same | DOCUMENTED | ADAPT |
| GND-63 | Operator overrides: local file and inline JSON sources; curated packs | Curated cameras with human-vouched poses | A file or inline list can add/replace cameras; entries may be marked "curated pose" | Operator | THIN (doc 10 lists the curated JSON packs, not the override mechanism) | LATER - G10 |
| GND-64 | Opt-in agreement-gated pack (New York) | Off unless operator holds an agreement | Flag must be set explicitly so an unset flag never reaches upstream | NYC DOT | doc 11/07 DOCUMENTED | SKIP |
| GND-65 | Seed cameras fallback | 18 hand-posed cameras if nothing live | Built-in list over 8 cities incl. Dubai | Built-in | doc 04 s6.1 DOCUMENTED | n/a |
| GND-66 | Camera pose model ("raw prior") | HDG/FOV/RANGE/pitch/height numbers | Explainer B.1; defaults and clamps in doc 04 | Derived | DOCUMENTED | KEEP idea |
| GND-67 | Heading provenance: ESTIMATED tag and dashed wires | "(ESTIMATED)" next to HDG; dashed coverage lines | Feeds with no facing get a bearing from a hash of the id (16 steps of 22.5 degrees) and a "low" confidence flag; a manual calibration or curated pose overrides the flag | Derived | doc 04 s6.2 DOCUMENTED | KEEP |
| GND-68 | Camera icons | Cyan icon (amber when active), always on top, scale 1.25 -> 0.42 | Horizon culling hides far-side icons because the globe is not drawing depth under 3D tiles; event-driven (camera settle), not per frame | n/a | DOCUMENTED | KEEP |
| GND-69 | Click / tap to activate | Clean tap on icon or card activates; drag or long-press does not; empty-globe click deselects | Selection bound to clean taps; re-selecting the active camera is a no-op (avoids a visible flash of the screen) | n/a | doc 06, 01 DOCUMENTED | KEEP |
| GND-70 | World-click focus request | Clicking a camera in the world also flies there (1.9 s) via the cockpit-safe path | Dispatches a focus request only for real activations by click; auto-hop and programmatic activation never fly | n/a | doc 06 DOCUMENTED | KEEP |
| GND-71 | NEAREST | Activates the camera nearest to the viewer camera's ground position and flies there | Great-circle distance over the whole catalogue; target is the *viewer*, not a selected object | n/a | doc 04 s6.2 DOCUMENTED (cross-link wording THIN - see C3) | KEEP |
| GND-72 | PREV / NEXT / dropdown / FOCUS | Cycle in catalogue order (wraps; NEXT from none = first, PREV from none = last); dropdown "city - name" flies 2.2 s | Each also turns the layer on first | n/a | DOCUMENTED | KEEP |
| GND-73 | Focus framing | Flies to look along the camera's heading | Offset along heading, pitch -22 degrees, range max(280 m, 1.18 x camera range) around a sphere of max(40 m, 0.36 x range); refused with a result code while an object is tracked or cockpit active | n/a | DOCUMENTED | KEEP |
| GND-74 | Hand-off from a tracked target | While following a plane/ship/fire, NEAREST activates the nearest camera but the view stays on the target | Activation allowed; flight refused (tracker owns the camera); also on layer enable: nearest camera activated, flight only when nothing owns the view | n/a | THIN (doc 04 "cross-links") | KEEP - G7 |
| GND-75 | Auto hop | AUTO HOP OFF/ON; cycles cameras every 18 s | Interval clamp 8-90 s (voice/tool only); needs 2+ cameras; if the viewer moved to a new coarse view cell since last hop it snaps to the nearest camera instead of next; empty-space deselect suspends it | n/a | DOCUMENTED | LATER |
| GND-76 | Coverage modes OFF / ON / VIEWSHED | One button cycles three states (default ON) | Explainer B.2 | n/a | DOCUMENTED | KEEP (as sensor-FOV visual) |
| GND-77 | Coverage neighbours | Active + nearest 14 within 1.8 km get wires | Lazy creation; idle neighbours never built until needed | n/a | DOCUMENTED | KEEP |
| GND-78 | Viewshed colours | Each cone its own hue | Golden-angle hue spacing over id-sorted index; translucent fill idle/active, two-sided, not pickable | n/a | DOCUMENTED | KEEP |
| GND-79 | Projection (monitor plane) toggle | PROJECTION ON/OFF (default ON) | Explainer B.3-B.6 | n/a | DOCUMENTED | KEEP |
| GND-80 | Obstruction probe and footprint lift | Screen not buried in buildings | Explainer B.4 (probe 4 m short, floor 12 m; lift <=60 m; 3x3 support grid; 2 m clearance) | Mesh / DEM | THIN (3x3 grid + probe on re-activation not described) | KEEP - G8 |
| GND-81 | Ground resolution per surface regime | Correct heights on both 3D-city and plain-globe maps | Two regimes (3D photoreal vs terrain globe); ellipsoidal ground prior from a terrain-height batch (waits up to 8 s then applies late); one-shot refine from the rendered mesh when tiles finish loading; catalogue elevations used only as fallback | Terrain heights | THIN (doc mentions ground priors) | KEEP - G8 |
| GND-82 | Shipped precomputed ground heights | Faster, more accurate placement | Sidecar of heights under mount and 9 plane support points for ~3,445 cameras; valid only while the camera's pose hash matches, otherwise runtime sampling | Offline job | THIN | LATER - G8 |
| GND-83 | Snapshot frames | Panel preview and plane image refresh | Still: 10 s active / 60 s idle; fallback chain upstream -> Street View still -> generated placeholder; 8 s deadline | City feeds | DOCUMENTED | KEEP |
| GND-84 | Live video (HLS) | Moving picture on plane and panel | Lazy-loaded HLS decoder; one decoder per camera feeds both surfaces; panel surface copy bounded to 640 px / 15 fps; live HLS never loops; bounded drift correction when encoders publish less media time than wall time; server relay with client leases | Delaware, Norway | DOCUMENTED (relay), THIN (copy surface and drift correction) | LATER |
| GND-85 | Server HLS relay limits | Invisible | 2 sessions; 12 segments / 24 MiB; 4 MiB per segment, 10 s; no redirects, encryption, fMP4, byte ranges, RTMP; 8 client leases per session; 15 s idle expiry | n/a | DOCUMENTED | n/a |
| GND-86 | Media proxy hardening | Invisible | Ranges: single well-formed byte range only, bounded; abandoned viewer cancels upstream; inter-chunk stall deadline; no persisted frames | n/a | THIN (doc 04: 8 s, 16 MB) | n/a - G9 |
| GND-87 | Health tracking | Source badge status | Per-camera OK/degraded; synced every 7 s; map bounded to catalogue ceiling | n/a | DOCUMENTED | KEEP |
| GND-88 | Source badge | "KIND - STATUS", "FRAME - LOADING", "FRAME - UNAVAILABLE", "SOURCE - UNKNOWN" | Settles after preload | n/a | DOCUMENTED | KEEP |
| GND-89 | Ambient thumbnail cards | Small frame cards on nearby cameras | LOD pick + grid distribution + incumbency + grace + paced fetch, never per frame | n/a | DOCUMENTED | LATER |
| GND-90 | Hover-summoned card | Pointing at an icon pops its card | Throttled pick (120 ms) while pointer moves; budget-exempt; 1 s linger | n/a | DOCUMENTED | LATER |
| GND-91 | Optional protected thumbnail for the active camera | Off by default | When on, active camera also gets a protected card in addition to the screen | n/a | **MISSING** | SKIP - G11 |
| GND-92 | Calibration gizmo (ADJUST) | Rings, arrows and handles on the active camera | Explainer B.7; ray-to-axis closest-point and ray-to-plane maths; grazing-angle guard rejects near-parallel hits (<0.08) so one pixel cannot slam a value; only event-driven picks | n/a | DOCUMENTED | KEEP |
| GND-93 | Click-to-type pose readout | HDG, PITCH, FOV, RANGE, HGT, deltaN, deltaE chips become number inputs | Enter/blur commits; Esc cancels | n/a | DOCUMENTED | KEEP |
| GND-94 | Save-gated calibration persistence | CAL badge CALIBRATED / CURATED / RAW PRIOR, "EDITED (UNSAVED)" | v2 browser store with provenance; old store deliberately ignored; range-floor migration | Browser storage | DOCUMENTED | KEEP |
| GND-95 | Gizmo commit grounding | Dragging east/north does not re-sample ground each move | Ground frozen during drag; one re-resolve at release with revision guard | n/a | DOCUMENTED | KEEP |
| GND-96 | Scene summary typing line | Typed one-line summary | Area of FOV wedge, overlap count, view band | n/a | DOCUMENTED | LATER |
| GND-97 | Panel auto-expand on active change | Panel opens for a new camera | Not for timed hops | n/a | DOCUMENTED | KEEP |
| GND-98 | Voice/assistant control of cameras | enable, disable, select by name, next, prev, nearest, focus, coverage, viewshed, adjust, projection, autohop | Selection by fuzzy name query over loaded cameras; refuses when layer off ("enable it first") | n/a | doc 08 DOCUMENTED | KEEP |
| GND-99 | Camera tools for external assistants | "cameras near X", "show me camera Y" | Area search nearest-first with facing/feed/credit; snapshot returned as an image; partial pack disclosure "N of M" | n/a | doc 08 s9 DOCUMENTED | n/a |
| GND-100 | Content stance | No blur, face/plate/object recognition | Frames relayed unaltered | n/a | DOCUMENTED | n/a |
| GND-101 | Credits / attribution per pack | Licence text where required | Required for Finland, NSW, TfL, BC, Calgary, Norway | n/a | DOCUMENTED | n/a |

### 5. Mapped ALPR

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| GND-102 | ALPR layer | Camera badges at city scale | Hourly OSM extract served as vector tiles; max span 3 degrees; zooms 0-8 heat-only, 9-12 attributes; finest zoom fitting 16 tiles | OpenStreetMap / DeFlock | doc 04 s5 DOCUMENTED | SKIP |
| GND-103 | Viewport snapping and reuse | Fewer reloads | Box snapped outward to a 0.05 degree grid; reuse 10 min | n/a | DOCUMENTED | SKIP |
| GND-104 | Caps and "coverage limited" | Notice when cut | 1,500 rendered; 6,000 decoded; 20,000 coordinate memory | n/a | DOCUMENTED | SKIP |
| GND-105 | Badge markers, scale by altitude, ground placement | 38 px (60 selected) | 1.5 m above resolved ground, <=4 surface samples per paint | n/a | DOCUMENTED | SKIP |
| GND-106 | Facing wedge | Illustrative cone when bearing known | 90 m depth, 20 degrees half-angle, labelled "not measured" | n/a | DOCUMENTED | SKIP |
| GND-107 | Click card | ALPR-nnnn, OSM MAPPED, DIRECTION, manufacturer/operator/type, PUBLIC MAP DATA | Elbow leader | n/a | DOCUMENTED | SKIP |
| GND-108 | SHOW NEAREST chip | Selects and flies to nearest loaded ALPR camera | 1.2 s, 800 m frame, 45 degrees pitched; disabled when off / empty / tracking | n/a | DOCUMENTED | SKIP |
| GND-109 | Row status strings and attribution timing | See doc | OSM credit collapses after 5 s | n/a | DOCUMENTED | SKIP |
| GND-110 | Assistant tool for ALPR | US/Canada area <= 3 degrees | n/a | n/a | MISSING by name (doc 08 lists the catalogue generically) | SKIP |

### 6. Street-level imagery

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| GND-111 | Provider-neutral core + registry | One panel, one chip per provider | A provider declares identity, colour, key need, credit, supported filters, coverage style, required map, default state and date semantics | n/a | doc 04 s8 DOCUMENTED | ADAPT |
| GND-112 | One provider lit at a time | Lighting one switches others off and drops stale state | n/a | n/a | DOCUMENTED | KEEP |
| GND-113 | Mapillary coverage + photos | Green coverage; click line -> cones -> photo | Vector tiles by zoom band; sequence lines below ~60 km; 24 h server tile cache with image layer stripped from high-zoom tiles; rate-limit holds | Mapillary (free token) | DOCUMENTED | ADAPT; UAE partial |
| GND-114 | KartaView coverage + photos | Purple lines below ~6 km; click -> nearest photo; filtered PNG mask | Direction derived from travel; archived photos skipped | KartaView | DOCUMENTED | ADAPT |
| GND-115 | Google Street View (Google 3D map only) | Click street below 3 km / OPEN STREET VIEW button | Nearest outdoor panorama within 50 m; no coverage lines; billed per panorama | Google | DOCUMENTED | LATER |
| GND-116 | IMAGERY (ALL/360/FLAT) and SINCE filters | Slider with 9 stops | Stored as relative days 0-36,500 so shared links keep meaning | n/a | DOCUMENTED | KEEP |
| GND-117 | Embedded viewer: EXPAND, FIT/FILL, FOLLOW, close; 360 WebGL | Photo in right-rail panel | FOLLOW = globe camera follows the photo's view at 2.4 m eye height (3D map only) | n/a | DOCUMENTED | KEEP |
| GND-118 | Sequence stepping + keyboard | Arrows step; Shift+arrows, up/down, +/- look around; pinch on phone | n/a | n/a | DOCUMENTED | KEEP |
| GND-119 | Surface mode | Lines sit 2 m above bare earth on 3D map | Switch in below 1,400 m, out above 1,800 m; within 900 m sample the rendered surface in ~11 m cells | n/a | DOCUMENTED | KEEP |
| GND-120 | Position marker, cones, credits, key states, floating panel, share state | See doc | n/a | n/a | DOCUMENTED | KEEP |

### 7. Directions, route annotation and walking flythrough

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| GND-121 | Directions A->B (drive/walk/bike) | Chips SET A / SET B / SWAP / CLEAR / FLY, step list | Click-armed endpoints through a rate-limited proxy to community OSRM; draped flowing-dash route; dot per manoeuvre | OSRM (FOSSGIS) | doc 04 s10 DOCUMENTED | KEEP; UAE needs own router |
| GND-122 | Pointer ownership message | Names the tool holding the pointer and how to put it down | Avoids silent failure of arm click | n/a | **MISSING** | KEEP - G12 |
| GND-123 | Maneuver dot height | Dots sit on their junction in high-altitude cities | Hidden until ground cell warm; re-read for up to 90 s | n/a | DOCUMENTED | KEEP |
| GND-124 | Active-step highlight during FLY | Row highlights current instruction | Step in force = last whose cumulative start is behind the camera; repaint ~every 0.25 s | n/a | DOCUMENTED | KEEP |
| GND-125 | Route flight cinematics | Camera dollies along route | See Gaps G13: speeds, ramps, bank, breathing, clearance, warm-up | n/a | THIN | KEEP - G13 |
| GND-126 | Voice: draw route then "fly it" | Spoken request draws the route; the route flythrough tool action with speed word | Fly operates on an existing route annotation; speed words slow/normal/fast | n/a | THIN (doc 08 mentions annotation route; the fly action's speeds are not recorded) | KEEP - G13 |
| GND-127 | Assistant route tool | Plan route with distance/duration/simplified path | Same routing service | OSRM | doc 08 DOCUMENTED | LATER |

### 8. Recent imagery (before/after)

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| GND-128 | 30-day day strip, boxes, modes IMAGE / VS BASEMAP / A-B, swipe, export, START HERE ranking, share | See doc | Catalogue search by box, tile drape, swipe divider | NASA CMR, GIBS, Worldview | doc 04 s7 DOCUMENTED | KEEP (UAE high value) |
| GND-129 | Assistant tool for latest clear image | Image of area | Landsat/Sentinel with VIIRS fallback | NASA | doc 08 DOCUMENTED | KEEP |

### 9. Radio and local SDR

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| GND-130 | Radio layer (markers by category, clusters) | Coloured station points, cluster badges | 750-station directory; category from tags; cluster identity inheritance (mutual-best overlap) so badges do not flicker | Radio Browser | doc 04 s9 DOCUMENTED (identity inheritance THIN) | KEEP; UAE stations present |
| GND-131 | Analog tuner | Drag needle, globe flies to each broadcaster | Needle position -> station by filtered directory order; simulated static between; commit/cancel semantics | n/a | DOCUMENTED | KEEP |
| GND-132 | Playback controls, volume, voice hand-off | PLAY/STOP/NEXT/PREV, ducking when voice speaks | Muted during live voice, gently restored | n/a | DOCUMENTED | KEEP |
| GND-133 | Voice radio control with place/category/country/name | "Play news radio near Dubai" | 8 preset locations plus free text/coordinates | n/a | DOCUMENTED | KEEP |
| GND-134 | Local RTL-SDR FM | CONNECT, TUNE, SEEK, SDR VOL, gain | WebUSB; mono wideband-FM demodulation in a background worker, audio resampled to 48 kHz and played through an audio worklet (stereo decoding off); spectrum-peak search finds the nearest broadcast channel for SEEK; per-mode gain memory; one tuner shared with ADS-B | User dongle | doc 04 s9.1 DOCUMENTED | LATER |
| GND-135 | Local ADS-B decode and decoder feeds | Magenta local aircraft | CPR decode, surface reference rules, CRC-valid timeout; merged by ICAO | User SDR / dump1090 feeds | doc 04 s9.1 DOCUMENTED (feeds), THIN (decoder rules) | LATER - belongs to aircraft census |
| GND-136 | Device selection memory | Reconnect without picker | Remembers identity (vendor, product, name, serial) per mode; prefers ADS-B-named channel for ADS-B; dual-channel boards enumerate as two devices | n/a | **MISSING** | LATER - G14 |

### 10. Mapped installations

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| GND-137 | Installations layer | Markers and footprints of mapped military land | Vector-tile land-use polygons, merged fragments, lazy name pack, wide-view point index; optional Overpass and Places search | OSM, Overture/OSM names, Google Places | doc 04 s4 DOCUMENTED | SKIP |
| GND-138 | Contacts coupling | Sites near a tracked plane/ship | 100 km window, moves after 20 km | n/a | DOCUMENTED | SKIP |


## Gaps found (with the missing behaviour written out so it can be filled now)

**G1 - Flow-to-road snapping (GND-07, GND-08).** TomTom flow lines and map road lines do not share ids, so flow is attached geometrically. Every flow line is split into short pieces; each piece is filed into a grid of roughly 100 m cells by its midpoint (pieces longer than a cell are subdivided so no cell is missed). Each map road is sampled at up to 7 evenly spaced points along its length. For each sample the nearest flow piece within 35 m is taken, but only if the piece's compass direction is within 30 degrees of the road's direction of travel (a flow line flagged as describing both directions may match either travel direction; two-way map roads are modelled as two separate travel directions, each matched on its own). A road counts as matched if at least half of its samples (minimum two) matched. Its congestion number is the median of the matched samples' values, which resists a single wrong piece. It is closed if any matched piece is closed. Output per road: a level 0-1 and a closure flag, or "no flow". Flow tiles are the "relative" style at zoom 12 (about 2.4 m quantisation), decoded in the browser from the standard vector-tile format into plain lon/lat lines; features without a numeric level are skipped unless closed (closed = level 0).

**G2 - Hybrid road de-duplication and drivable filter (GND-09, GND-10).** Hybrid draws TomTom roads wherever TomTom has them; a map road is kept only where TomTom has none. Overlap is tested by sampling the map road every 10 m and asking whether a TomTom line lies within 35 m. When exactly one of the two is a controlled-access mainline (motorway/trunk) and the other is not, the radius shrinks to 15 m so that frontage and service roads (20-40 m from a freeway) survive. A partly covered map road keeps only its uncovered stretches, and only stretches of at least 40 m. Only TomTom road categories that are real drivable roads (motorway, international road, major road, secondary, connecting, major local, local, minor local) become vehicles or suppress map roads; unknown, private, parking and walking categories never do. TomTom classes map to the map-road classes used for speed/size/density (e.g. "Major road" behaves like primary, "Local road" like residential).

**G3 - Dot placement and motion details (GND-12, 13, 15, 16, 17, 18).**
- Count per road = floor(road length / spacing x class density x user density scale x congestion factor x road weight), at least 1. Spacing depends on camera clearance above the viewed ground: under 1 km clearance 30 m; under 3 km 80 m; under 5 km 150 m; otherwise 250 m. Two-way roads are weighted 0.5 per direction.
- Direction: one-way roads use the legal direction; two-way alternate forward/backward by index.
- Start position random along a random segment; per-dot speed = class speed x user speed scale x random 0.7-1.3 x congestion factor (floor 0.15).
- Near road ends (first or last two segments) each entry into a segment has a 0.8% chance to stop the car for 2-6 s.
- Cars at the end of a road shrink to nothing then re-enter at the start as a new identity (new id, new birth time); newly born and retiring cars fade over a short fixed period; a car's size also scales down in the last quarter-second of road.
- Frame delta capped at 0.1 s. Movement is carried in metres across bends so short segments cannot cause jumps.
- Live mode can show white (simulated) dots on roads without data (default) or hide them for a strict data-integrity view.
- Dots on closed roads: none. In live mode the speed and density of a late flow match are rescaled in place without respawning; queue positions wait for the next natural re-render.

**G4 - Jam queues (GND-21).** In the shipped default jam mode (density), a jam-bucket road gets platoons: each platoon anchors at a random distance and trails 3-7 further cars at 6-12 m bumper-to-bumper gaps (platoon sizes 4-8), wrapping around the road start; all cars of a platoon share one direction (alternating by platoon on two-way roads). Jam cars carry a stop-and-go state: move 1.2-3 s at 2.2x, then stand 1.5-5 s. Cars on jam roads draw a pixel larger and farther away shrink less than others (far-distance scale floor 0.55). Heat lines are a separate, default-off option (the A/B choice is none / density / heat lines / both).

**G5 - Style-preset dot treatment (GND-24).** Rationale: dots are real 3D points that pass through the full-screen look effects. Night-vision and thermal reduce everything to brightness, so the normal palette would make a jam dimmer than free flow; the retro look pixelates and dithers 4-6 px dots away. Treatments: mono (surveillance/night vision, thermal, noir) = bright white core + thin black halo; jam keeps a larger size; retro = saturated primaries (survive heavy posterisation) and a size boost; normal/anime/snow = untouched. Coloured (live) dots are floored at 5 px under any non-normal look. White simulated dots are never restyled. Classification under these looks is carried by the detection brackets' colour tier, not by dot colour. A switch can force the normal palette; restyle is applied in place without refetch.

**G6 - Transit mode inference and time provenance (GND-35, GND-41).** A vehicle's mode comes from its route id using operator-specific conventions (e.g. Boston: colour-named lines are rapid transit, an "express rail" prefix is commuter rail, a "boat" prefix is ferry, numbers are buses; Norway: operator codespace; Helsinki: leading digits of the route code, with a 10xx range for trams, "M" suffix for metro, 1019 for ferry), falling back to the feed's default mode. When the mode is only the feed default it is flagged as inferred and the permissive 200 km/h plausibility ceiling is used (a feed whose default is bus also publishes fast rail). Every stored position is tagged with where its timestamp came from (vehicle clock, feed header clock, or time of receipt), so age text can be honest. The playback lag is learned per feed: the 95th percentile of how late reports arrive (and the feed-wide 90th percentile) plus 5 s, clamped, reset when a feed is silent for 90 s; while a vehicle is mid-segment the segment finishes at 1x even if lag grows.

**G7 - Hand-off from a tracked target to a camera (GND-74).** Actual behaviour: there is no "nearest to selected target" lookup. NEAREST, PREV/NEXT, FOCUS and layer enable measure from the *viewer camera's ground position*. When something is being tracked or the cockpit view is active, activation still happens (the active camera, its plane and wireframe switch) but the flight is refused and a result code (tracking holds the view / cockpit active) is returned; the user must stop tracking to be flown there. Because tracking keeps the camera over the target, the chosen camera is normally near the target, which is why the mission copy pairs ports and fires with this button. On layer enable the same policy runs once (activate nearest, fly only if nothing owns the view; ownership is sampled before and after the asynchronous enable so either observation suppresses the flight).

**G8 - Ground/footprint resolution (GND-80, 81, 82).** The screen rectangle is described by nine support points (a 3x3 grid: bottom/middle/top rows by left/centre/right columns); their horizontal positions depend only on the pose. The ground height under each is measured (from precomputed heights shipped for the 3D city surface, valid only while a short hash of the exact pose still matches, or from terrain heights on demand when a camera is activated or calibrated). The plane is lifted by one rigid amount so every support point clears its measured ground by 2 m; missing measurements fall back to the ground at the mount, making the pre-measurement result deterministic instead of buried. Two surface regimes exist: 3D city (globe hidden; rendered mesh is truth) and terrain globe (terrain model is truth). Per camera and per regime there is a one-shot "resolved" latch; the first update tick after 3D tiles finish loading re-queues cameras that were drained while tiles still streamed, then the camera never resamples on a timer. Re-selecting a camera re-arms one sample; a re-activation re-probes obstruction from real ground.

**G9 - Media relay hardening (GND-86).** The media relay accepts a client Range request only when it is exactly one well-formed byte range (first-last, first-open, or suffix); anything else (multiple ranges, CR/LF) is dropped and the full body returned; every accepted form is bounded to the same ceiling as the relay's declared-body limit. A declared oversized body is refused; unbounded live bodies stream through with backpressure; a stall timer measures gaps between upstream chunks (not stream life); when the viewer disconnects, the upstream request is cancelled even before headers arrive. A client's own string is never reflected into a health message.

**G10 - Operator-defined cameras (GND-63).** Operators can add or replace cameras via a local JSON file or an inline JSON setting; entries may carry a "curated pose" mark, which shows the CURATED calibration badge and suppresses the ESTIMATED tag. File/inline entries override live ones by id. A camera whose live pack stops serving a stable id but keeps an operator entry keeps working.

**G11 - Optional protected active-camera card (GND-91).** The active camera normally has only the floating screen (no thumbnail card); a programmatic option can additionally publish a protected card for it (never faded or capped). Default off. Low value for Kawn.

**G12 - Pointer-ownership message for Directions (GND-122).** Arming SET A/SET B while another tool owns the globe click names that tool in plain words and says how to put it down (previously it failed silently and testers thought Directions was broken). Reuse this pattern wherever tools share the pointer.

**G13 - Route flight tuning (GND-125, GND-126).** Speed words set the mean ground speed: slow 20 m/s, normal 40, fast 90; a trapezoid profile eases in and out over 2.4 s ramps (a ramp may take at most 35% of a short route), so duration = length / mean speed. The camera rides a mean 260 m above ground, pitch locked at -32 degrees. Gaze looks ahead 6.5 s of travel (clamped 120-600 m) and is smoothed. Bank (roll) is derived from the turn angle measured over a 4 s window centred on the camera, 0.44 degree of bank per degree-per-second of turn, filtered twice, capped at 10 degrees (about 7.5 for a 90-degree corner). Altitude breathes +/-20 m over a 2.2 km wavelength and rises up to 26 m into a turn, smoothed; minimum clearance 90 m above the sampled ground is a hard clamp; descent speed is capped (climbs are not). Before the camera moves it "arms" for up to 1.2 s waiting for ground data; the route corridor is warmed ahead (4 km at start, then a 3 km rolling window every 2 s, sample every 55 m, at most 96 cells), and the mesh probe fires once per flight over at most 8 corridor cells. Reduced-motion zeroes bank, breathing and lead shaping. Only a flight the layer itself started is stopped by reroute/CLEAR/disable. Refused in cockpit mode.

**G14 - SDR receiver memory (GND-136).** The browser remembers the full identity (vendor, product, product name, serial) of the receiver per mode (FM vs ADS-B). Dual-channel boards appear as two USB devices that can share a serial number, so for ADS-B the channel whose name mentions ADS-B/1090 MHz is preferred and for FM a channel not named for ADS-B or UAT; an already-authorised device is used without opening the picker. Stalled reads (5 s) tear the session down; closing is bounded (2 s, then raw close). Gain defaults: AUTO for FM; 28.0 dB for ADS-B, because the browser USB library maps dB to tuner stages differently from the usual desktop tools (20.7 dB heard about 1 msg/s, 28.0 dB about 9 msg/s on the same antenna).

**G15 - MCP/tool catalogue entries by name.** Our docs describe the assistant tools generically; the ground-level ones are: public cameras nearest-first, camera snapshot (image), mapped plate readers (US/Canada, <= 3 degrees), radio stations (area + terms), plan route, bike share, transit vehicles (optional route), traffic flow (city-sized area; speed vs free flow, congested and closed road), recent clear imagery, mapped military installations (<= 10 degrees per side). MCP withholds place search, routing, radio, bike share and transit as "assistants already cover these or they add little without the globe".

## Corrections to existing docs

- **C1 (doc 03 s10, jam visualisation).** Doc is right that density mode is the default, but it implies the "up to 2.5x" density curve is the norm. With the shipped default the deeper curve (to 4x) and the platoon/creep behaviour are on by default in live mode; heat lines are the default-off option. Say so explicitly.
- **C2 (doc 03 s10, Caps).** "roads longer than 80 waypoints are sub-sampled" is correct; add that the dot count per road uses clearance-based spacing (30/80/150/250 m) and a fair share of the global cap.
- **C3 (doc 04 s6.2 Cross-links).** "A tracked fire or vessel can hand off to the nearest live camera": the nearest camera is computed from the viewer camera's ground position, not the target's, and while the target is tracked the camera is activated but not flown to. See G7.
- **C4 (doc 04 s6.2 Projection).** Add the 3x3 support-point grid and the pose-hash validity of shipped ground heights (G8); "footprint lifted by at most 60 m" refers to the rigid lift beyond what the mount ground needs.
- **C5 (doc 03 s8 Playback).** Lag is learned per feed from observed report lateness (G6) rather than a fixed 25-120 s; 25 s is the floor.
- **C6 (doc 04 s10 FLY).** Add speed words and the arming/warm-up/clearance details (G13).
- **C7 (doc 04 s9.1).** Add that FM is mono wideband demodulation (stereo off) and the receiver-memory behaviour (G14).

No statement in the studied sections was found to be flatly wrong; corrections are completeness fixes.

## UNVERIFIED

- Whether the GND-29 DETECT overlay sampling for traffic is identical for transit and bike share (only traffic's sampling was read directly).
- Current live availability of each camera pack (counts quoted are per documentation and source defaults, not a live run).
- Hybrid-mode behaviour with a TomTom key was read from code and the project's own QA descriptions; no live TomTom call was made.
- Mapillary/KartaView/Google coverage in the UAE.
- Exact stereo behaviour of the FM path beyond "stereo off" in the shipped configuration.
- Percentages in the heading hash (16 fixed bearings) apply to packs that publish no facing; Finland's and TfL's behaviour is as documented in doc 04 s6.1.

## Counts

- Features listed: 138 (GND-01 to GND-138).
- MISSING: 13 rows (mostly traffic mechanism, DETAIL-level behaviours, device memory).
- THIN: 20 rows.
- Remaining rows DOCUMENTED (dominant outcome: this domain was already covered in depth by docs 03 and 04).
- Gaps written out in full: G1-G15. Corrections: C1-C7.

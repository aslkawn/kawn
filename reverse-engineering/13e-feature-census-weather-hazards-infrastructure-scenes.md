# 13e - Feature census and gap audit: weather, hazards, infrastructure, Director/scenes, annotations, recording and share (ENV)

## Purpose and method

Purpose: an independent census of every feature in the studied globe application that falls in this domain (wind particles, rain radar, satellite clouds, lightning, cyclones and the weather timeline, earthquakes, fire detections and perimeters, submarine cables, data centres, dams, installations, regional extras, Director/scenes, annotations/draw/measure, recording and safe frame, share), with a plain-words explanation of how each works, then a comparison with our notes 00-12. Clean-room: no code, no identifiers, no copied sentences. Numbers are values observed in the studied product and are sizing hints, not requirements.

Method. The inventory was built first from the original product's own material (readme, change log, data-source notes, current-state notes, the tools catalogue, the Director/scene-format notes, the markup templates, the bundled event pack, the voice-action schemas, and the layer/director/annotation source), then compared with our notes. The code-graph index of the original (GitNexus) was queried for: the wind particle animation (it only surfaced the generic animation loop, so the wind pipeline was traced by reading the wind modules), scene director playback (surfaced the playback adapter and its travel/hold phases), and the observed-weather history clock (surfaced the event-panel sync and the weather manifest parser). Source was then read to confirm numbers and to test our notes' claims. No live run was made. Confidence is source-level unless marked UNVERIFIED.

Verdict key: KEEP / ADAPT / LATER / SKIP for Kawn (UAE city intelligence: Dubai, Abu Dhabi, Sharjah; Arabic + English, RTL). Documented key: DOCUMENTED (doc + section) / THIN / MISSING.

Headline result: our notes already cover this domain very well (04 sections 1-4 and 11-15, 03 sections 11-13, 09, 07 sections 5-7, 02 part C). The audit found 3 outright errors in our notes, 6 thin areas where the "how it works" was absent or imprecise, and no wholly missing feature. The gaps are written out in full below.

---

## Plain-words explanations (the three mechanisms you asked about)

### How the wind particle flow works

1. Data. A server-side step picks the newest published global forecast run (NOAA GFS or ECMWF IFS, choosable), downloads only the byte ranges holding the east-west and north-south wind components at 10 m (plus optionally 2 m temperature and sea-level pressure from the same run and forecast step), decodes them, and shrinks the 0.25 degree grid to about a 1 degree grid. The browser receives a small manifest plus raw number arrays. It is a forecast: one frozen moment, not an observation, and not a live feed.
2. Baking the "lines of flow". When a field arrives, the browser seeds up to 7,200 start points on desktop (1,200 on screens under 700 px wide), spread evenly by area over the globe using a golden-angle spiral so the poles are not over-crowded and no seed pattern repeats. From each seed it walks the wind field both forward and backward in time, 16 steps each way, each step representing half an hour of wind, using a two-stage (midpoint) integration so curved flow stays smooth. Each step is length-limited so fast winds near the poles cannot jump across the map. A path stops at calm air (under about 0.15 m/s), at the poles (beyond about 88.5 degrees), at the map seam, or at missing data; paths shorter than three points are dropped. Result: each surviving path is a curve of up to 33 points that follows the wind through that seed. Speed at the seed is remembered for colouring. This baking happens once per field, not per frame, and is not repeated when the camera moves.
3. Grouping and culling. Paths are grouped into 30 degree regional cells. Each frame, cells that are behind the horizon or outside the view are skipped; if no cell is visible the animation loop stops and restarts on the next camera event.
4. Animation. The curves are drawn as ribbons lifted about 12 km above the surface (a visual aid; the data is still 10 m wind). Motion is done on the graphics card: a single "phase" clock value advances, and the ribbon material shows a bright head with a fading tail that slides along each curve, wrapping about every six seconds; each path starts at a different offset (a golden-ratio offset) so heads are not in lock-step. A faint "ghost" of the whole curve stays visible (slightly brighter when paused). Curve ends, edges and the horizon side fade out. Ribbons fade when the camera is under 60 km and disappear under 15 km. Because only the phase changes, no per-point processing happens per frame.
5. Colour. Plain trails are the default. The optional colour field (speed 0-30 m/s, pressure 960-1050 hPa, temperature -40 to 50 C) is a 360 x 181 raster drawn under the trails: draped on the globe, or a raised shell 5 km up on the photoreal 3D map; it fades out below about 1,200 km camera height and is hidden at or below 200 km.
6. Fallback. If the graphics path is not available a canvas renderer moves individual particles: each particle samples the wind at its location, advances by the wind scaled by camera height (so apparent speed looks similar at any zoom), lives 8-12 seconds, respawns at random spots within the visible region, and leaves a trail that is erased with a half-life of about 0.65 s. Pause and reduced-motion show the static flow with no idle loop; hidden tabs suspend.
7. Time. The animation never advances forecast time. The model run and valid time are fixed until the next hourly refresh or a model switch.

### How the weather timeline playback works

1. Each observed product (rain radar, satellite clouds, lightning density) publishes a manifest listing at most 13 recent frame times. The server reads the list from the provider's map-service capabilities, keeps up to 26 recent times in a validation list, and shows the newest 13. Manifests refresh every 2 minutes (every 10 for lightning); exact-time images are cached for 24 hours; manifests and errors are never cached.
2. One transient "observed clock" drives all enabled products. It has two modes: LATEST (each product shows its own newest frame, so times can differ) and HISTORY (a single target time).
3. The timeline's ticks are the sorted union of all non-suspended products' times. In HISTORY each product shows its nearest frame at or before the target, never a future frame and never a blend, provided the frame is no older than the product's allowed gap (30 minutes for radar, regional infrared and lightning; 3 hours for hourly global infrared). If none qualifies the product hides and says so. Each product labels what it actually shows as "synced" (exactly the target) or "nearest".
4. Controls: Earlier/Later step through ticks, a slider scrubs (the local readout updates at once, the clock commit is coalesced to every 150 ms, and the final commit on release is immediate), Latest returns to live, Play auto-advances.
5. Play waits for every product to finish showing its frame, then advances after 2 seconds, wraps at the end, and stops if fewer than two union times exist or all products are suspended. While a frame shows, the next frame is fetched and decoded in advance on a best-effort basis (with a deadline, cancelled on pause or replacement); on the 3D map the sharper detail image for the next frame is also warmed. A frame that fails keeps the previous picture on screen.
6. Suspension: hidden tab, the user's reduced-motion setting, or a map source with no imagery host all pause the products. Wind does not take part (its card says it does not follow history). History is transient: share links always open at LATEST.

### How scene (Director) playback works

1. A project holds scenes; a scene holds shots; a shot holds a camera pose (or an authored start-to-end move), a flight time, a hold time, the visual look, the on/off state of the layers it names, optional data packs and optional click actions.
2. START builds a queue of every shot beginning with the selected scene and continuing through the remaining scenes in order, wrapping round to the earlier ones (the whole project plays unless the caller asks for a single scene, which voice does). The queue is fixed at the start.
3. The runner claims the camera for the whole run (it refuses if cockpit mode owns the camera or another run is active), turns on the presentation chrome (panels hidden, safe-frame guide, scene-playing mode) and then, for each shot, does seven phases in order: select the shot; apply the visual look; reconcile layers (only the layers the shot names; refused layers are reported; tracking selections are never re-established); load the shot's data packs; fly the camera; settle; hold. After every phase it checks for cancellation, and the cancellation signal is also handed to the in-flight camera flight and layer transitions so Stop is immediate.
4. Camera flight for an ordinary shot is the engine's own eased flight (cubic in/out) with a safety timeout 0.6 s longer than the flight; an authored move instead samples between its two poses each animation frame (linear or cubic easing, shortest-arc heading/roll/longitude).
5. Hold normally just waits; if a layer has a media owner (witness-clip evidence in the Nepal scene) the hold waits for the media to finish, with a hard cap of 20 s, and an overrun stops the scene with an error rather than hanging.
6. A progress bar and a separate scene clock tick while it runs; every phase is written to an in-memory run log that RUN LOG can download. When the queue ends (or Stop, Escape, or a navigation takes over) cleanup runs: timers and animation callbacks are revoked, data packs and click actions are cleared, the camera flight is cancelled, the scenes' "release" list of layers is switched off, and the user's HUD mode and layout from before are restored. There is no pass that restores the layers the user had before the run.
7. Scrubbing (event scenes) and LOAD do not run the queue: they compute where a given scene time falls (which shot, flight or hold phase) and place the camera and layers accordingly, newest request winning.

---

## Feature table

Columns: ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict and UAE relevance.

### A. Wind

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| ENV-01 | Wind layer on/off, row and WEATHER card | Row in Weather group with toggle and one status line; card with Settings (default closed) | Forecast field fetched hourly; last-good retained; failure backoff of one minute; six model/field cache slots each keeping the last two issued grids | NOAA GFS / ECMWF IFS open data (keyless) | DOCUMENTED 04 s11 | ADAPT; high UAE value (shamal, dust) |
| ENV-02 | Animated wind trails | Glowing streaks flowing along the wind over the globe | Baked streamlines + graphics-card phase animation (see explanation) | same | DOCUMENTED 04 s11 (limits); technique THIN | ADAPT with finer regional grid |
| ENV-03 | Model choice (GFS / ECMWF) | Chips in card; choice persists and travels in share link | Model change clears old field immediately, cancels old fetch, ignores superseded results; cycle and step nearest to now chosen, previous run used if newest not yet published | same | DOCUMENTED 04 s11 | KEEP |
| ENV-04 | Colour field (None / Speed / Pressure / Temperature) | Raster tint under trails with a fixed-scale legend | Fixed physical colour scales (speed 6 stops, temperature 10, pressure 5) rasterised at 360 x 181; draped or 5 km shell; fades below ~1,200 km, hidden at <= 200 km; a missing companion field leaves wind usable and names the field unavailable | same, same model run | DOCUMENTED 04 s11 | ADAPT (temperature scale for Gulf summer) |
| ENV-05 | Wind units chips | km/h (default), m/s, mph | Only shown while a speed legend or reading is on screen; reformat captured sample without resampling | n/a | DOCUMENTED 04 s11 | KEEP (add knots for marine) |
| ENV-06 | Inspect centre / reading | Button captures map-centre point; card shows coordinates, speed, "from" direction (16 points, "calm" under 0.2 m/s), scalar, model, valid time, freshness; passive marker | Bilinear sampling of the grid; marker follows snapshot, hides behind globe; cleared by Clear, model/field change or refresh | same | DOCUMENTED 04 s11 | KEEP |
| ENV-07 | Pause / Resume, reduced motion, hidden-tab suspend | Static flow view; chip reads reduced motion and disables | No idle loop when paused; resumes on camera events | n/a | DOCUMENTED 04 s11 | KEEP |
| ENV-08 | Relief shading helper | Subtle terrain-dependent shading while wind is on | Uses terrain surface normals for view-directed shading, falls back to whole-globe curvature shading; never replaces an existing globe material; not sunlight, no elevation | n/a | DOCUMENTED 04 s11 | LATER |
| ENV-09 | Canvas particle fallback | Moving dots with fading tails when GPU path unavailable | Per-particle advection, 8-12 s life, camera-height-scaled speed | same | THIN (named only) | KEEP as fallback |
| ENV-10 | Wind query by voice/automation | "What is the wind at X" | Returns 10 m model wind at a point; also present in situation brief | same | DOCUMENTED 08 | KEEP |
| ENV-11 | Wind share state | Model, field, units, pause in link | Allow-listed layer options | n/a | DOCUMENTED 04 s0 | KEEP |

### B. Rain radar, satellite clouds, lightning, weather timeline

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| ENV-12 | Rain radar layer | Coloured reflectivity over the contiguous US; legend 10-80 dBZ | Map-service tiles (geographic, 256 px, to level 6) draped on globe; whole-extent image shell on 3D map; "reflectivity not rain rate" honesty copy | NOAA nowCOAST (MRMS), keyless | DOCUMENTED 04 s12 | SKIP data (US only); keep pattern |
| ENV-13 | Satellite clouds, regional (GOES) | Cloud imagery over North America | Band-14 longwave infrared, ~5 min, tiles; display-only brightness ramp | NOAA nowCOAST | DOCUMENTED 04 s12 | SKIP region |
| ENV-14 | Satellite clouds, global | Slower worldwide infrared mosaic, hourly, 2-3 h late; covers UAE | One 2048 x 1024 mosaic per frame fetched, processed once, cropped into 256 px tiles to level 3 (avoids tile-to-tile contrast seams); up to 6 processed mosaics cached | NOAA nowCOAST | DOCUMENTED 04 s12 | KEEP; only covers UAE |
| ENV-15 | Clouds-only vs Full image | Chips; clouds-only dims dark pixels | Soft brightness ramp (smooth step 0.40-0.70 in linear light), once per decoded image; full keeps source alpha | n/a | DOCUMENTED 04 s12 | KEEP |
| ENV-16 | Lightning density | 15-minute strike density, Americas/Pacific | Derived density product (~8 km grid); not a live counter | NOAA nowCOAST from commercial networks | DOCUMENTED 04 s12 | SKIP (no UAE coverage) |
| ENV-17 | Per-product controls | Soft/Vivid opacity chips; coverage badge; view-coverage button; "Map center outside coverage" | Flight to product bounds in 1.4 s | n/a | DOCUMENTED 04 s12.1 | KEEP |
| ENV-18 | Observed-history timeline (Earlier, Later, slider, Latest, Play) | Single timeline for all observed products | Union-of-times, nearest-at-or-before rule, gaps, 2 s play step (see explanation) | n/a | DOCUMENTED 04 s12.3 | KEEP the pattern |
| ENV-19 | Delayed/stale flags | "Source observations delayed" after 20 min (radar, regional), 45 min lightning, 4 h global | Compares newest advertised frame to now | n/a | DOCUMENTED 04 s12.1 | KEEP |
| ENV-20 | Frame prefetch and failure retention | Smooth play; previous frame stays if a frame fails | Best-effort next-frame warm with deadline; throttled upstream (429/503) retried up to three times per tile per frame | n/a | DOCUMENTED 04 s12.2 | KEEP |
| ENV-21 | Raised translucent shells on photoreal 3D map | Weather floats above city: wind field 5.0, global IR 5.5, regional IR 5.8, radar 6.2, lightning 6.6 km | One rectangle per product, alpha-blended, no depth write, drawn first in height order, unpickable; camera-height rise for far views; sharper 4096 x 2048 detail window around the view (not for global IR) | n/a | DOCUMENTED 04 s12.2 | KEEP technique |
| ENV-22 | Globe tile hosting and imagery order | Weather below/above other imagery in fixed order | Replacement layer shows before old retires; skip already-ordered layers | n/a | DOCUMENTED 04 s12.3 | KEEP |
| ENV-23 | WEATHER panel | Context-rail panel with cards (cyclones, wind, radar, satellite, lightning), opens itself once, hides when empty | One card open at a time; status lines reserve space; refreshes coalesced per frame | n/a | DOCUMENTED 04 s12.3 | KEEP |
| ENV-24 | Weather map image tool | "Latest radar/satellite/lightning image over an area" for assistants | Returns a picture from the same image service | NOAA | DOCUMENTED 08 (THIN on size limits) | ADAPT |
| ENV-25 | Recording/clean view hides weather chrome | Panel and controls hidden while imagery stays | Shared body-level mode | n/a | DOCUMENTED 04 s12.3 | KEEP |

### C. Cyclones

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| ENV-26 | Cyclone advisories layer | Storm markers, forecast track, lead-hour points, uncertainty cone | Same-origin provider merges status feed (no browser CORS) with GIS summary layers; 5-min snapshot; one native data source preserving polygon holes and seams | NOAA NHC / CPHC | DOCUMENTED 04 s13 | SKIP (basin); keep pattern for Arabian Sea |
| ENV-27 | Advisory consistency rule | Track/cone shown only if all parts match the status advisory; else "awaiting advisory N" | Match by advisory number; newer status shows position with geometry pending | n/a | DOCUMENTED 04 s13 | KEEP |
| ENV-28 | Storms list in card, auto-select, fly-to | Numbered list; first storm auto-selected | 1.4 s flight | n/a | DOCUMENTED 04 s13 | KEEP |
| ENV-29 | Cone/horizon rules | Cone is centre uncertainty, not size; all parts hidden beyond horizon | Copy shown in legend | n/a | DOCUMENTED 04 s13 | KEEP the copy |
| ENV-30 | Cyclone voice/automation query | "Active storms" optionally in an area | Reads same snapshot | NOAA | DOCUMENTED 08 | LATER |

### D. Earthquakes

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| ENV-31 | Earthquake discs (24 h, M2.5+) | Ground-clamped translucent discs; radius doubles per magnitude step (about 32 km at M5); colour by depth: under 70 km red, to 300 km orange, deeper yellow; M5+ more opaque with thicker outline | Static geometry rebuilt on each refresh from a fully validated snapshot; malformed feed keeps last good; unknown magnitude excluded | USGS daily feed (browser-direct, keyless) | DOCUMENTED 03 s11 | ADAPT (regional Zagros / Gulf of Oman view) |
| ENV-32 | Magnitude labels | "M4.6" tag above disc in depth colour | Shared label host: nearest-largest 96 cohort, 48 collision slots, priority = magnitude | n/a | DOCUMENTED 03 s11 | KEEP |
| ENV-33 | 60-s refresh, DEGRADED/UNAVAILABLE states | Row status changes | Abort on disable; late results ignored | n/a | DOCUMENTED 03 s11 | KEEP |
| ENV-34 | Earthquake analyst records and tool | Count/strongest by voice, up to 25 rows by default | On-demand snapshot | n/a | DOCUMENTED 03 s11, 08 | KEEP |
| ENV-35 | No click card | Display-only (confirmed) | n/a | n/a | DOCUMENTED 03 s11 | ADD a card in Kawn |

### E. Fire detections and perimeters

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| ENV-36 | Active fires layer | Heat cells from orbit; glow sprites close in | Four altitude bands (2 degree cells to 1,800; 1 degree to 3,600; 2,500 then 3,000 sprites) with 10% hysteresis, 650 ms check; top radiative power kept when clipping | NASA FIRMS (needs free key) via server merge of three VIIRS satellites plus MODIS, trailing 24 h | DOCUMENTED 03 s12 | ADAPT with industrial-flare filter |
| ENV-37 | Severity colour and size | Red/orange/yellow by power+confidence; core 8-28 px | Heat = square-root of power/150 weighted 0.85 plus confidence 0.15 | n/a | DOCUMENTED 03 s12 | KEEP |
| ENV-38 | Fire cards (ambient 18, selected, cell) | "FIRE 47 MW", confidence bucket, age, satellite short name, night flag | Greedy 150 px spacing, nearest hemisphere only | n/a | DOCUMENTED 03 s12 | KEEP |
| ENV-39 | Click to inspect and dive; nearest-camera hand-off | Camera dives to detection; tracked fire offers nearest live camera | Selection survives refresh by position + acquisition time | n/a | DOCUMENTED 03 s12 | ADAPT |
| ENV-40 | Row status/KEY REQUIRED | LIVE / STALE / KEY REQUIRED / unavailable | Proxy caches 30 min, serves stale on failure | n/a | DOCUMENTED 03 s12 | KEEP |
| ENV-41 | Fire perimeters | Containment-coloured polygons (red, orange, amber, green) with legend and counts | Paged official feed (5 x 2,000), simplified to ~100 m, 5-min refresh; unchanged snapshot keeps geometry | NIFC WFIGS | DOCUMENTED 03 s13 | SKIP (US only) |
| ENV-42 | Incident card with verified official-page link | Facts line, cost, personnel; link only if page verified | Catalogue 1 h cache; origin/date check, fail closed | InciWeb | DOCUMENTED 03 s13 | SKIP, keep verification idea |
| ENV-43 | Fire tools | Largest fire / perimeters query | Analyst records | n/a | DOCUMENTED 03, 08 | LATER |

### F. Submarine cables, data centres, dams, installations

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| ENV-44 | Submarine cables | 712 cable lines in dataset colours, 1,917 landing points | Bundled static data; lines clamped or visible per map stack; fully removed (not hidden) on disable | TeleGeography (non-commercial) | DOCUMENTED 04 s1 | ADAPT; replace licence |
| ENV-45 | Cable and landing labels | Nearest 160 within 9,000 km | Shared label host; no depth test | n/a | DOCUMENTED 04 s1 | KEEP |
| ENV-46 | Cable click | Flight only, no card | 6.5 km, 52 degrees, 1.35 s | n/a | DOCUMENTED 04 s1 | ADD real detail card |
| ENV-47 | Data centres (4,351) | Cyan footprints, stems, labels | Camera-height stem budget (80/200/420), priority scoring, ground sampling within 75 km | OSM extract (ODbL) | DOCUMENTED 04 s2 | ADAPT with authoritative UAE data |
| ENV-48 | Data-centre click/selection | Selection with 1-line card; flight to 5 km | Shared selected-context for voice | n/a | DOCUMENTED 04 s2 | ADAPT |
| ENV-49 | Dams (704) | Blue footprints, same behaviours | Same | OSM / Open Infrastructure Map | DOCUMENTED 04 s3 | SKIP (none in UAE) |
| ENV-50 | Mapped installations | Military-area polygons merged to markers, named lazily | Viewport tiles, 16-tile cap, bundled named index for wide views | OpenFreeMap + Overture/OSM names, optional Overpass/Places | DOCUMENTED 04 s4 | SKIP (policy) |
| ENV-51 | Installations around a contact | Mapped sites within 100 km of selected contact | Window follows subject; moves after 20 km | n/a | DOCUMENTED 04 s4 | SKIP |
| ENV-52 | Infrastructure/cable tools | Query by area/name | On-demand records | n/a | DOCUMENTED 08 | KEEP for cables/data centres |

### G. Regional extras

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| ENV-53 | Country/state/county outlines | "Outline Texas" draws real boundary; no network call | Bundled delta-encoded packs, lazy load, offline alias resolution, qualifier rules for ambiguous names | Natural Earth, US Census | DOCUMENTED 04 s15, 07 | ADAPT: emirates, municipalities |
| ENV-54 | Named physical/marine regions (e.g. a mountain range) | Navigation and annotation to a range | Bundled polygons; range resolved offline before network geocoders | Natural Earth | DOCUMENTED 04 s15 (brief) | KEEP (Hajar, Gulf) |
| ENV-55 | Neighbourhood polygons | Offline neighbourhood outlines (one city shipped) | Point-in-polygon and name lookup; no guessing | City open data | DOCUMENTED 04 s15 | ADAPT for Dubai communities |
| ENV-56 | Regional news (Local Info) | Headlines for the place under the aircraft/camera (cockpit page) | Region resolved offline; news search first, global event database as fallback; 0.1 degree cells, 5-min cache | Google News feed, GDELT | DOCUMENTED 04 s15 | ADAPT, licence review |
| ENV-57 | Local weather readout | Current conditions for a place | 0.1 degree cells, 5-min cache, stale to 30 min | Open-Meteo | DOCUMENTED 04 s15 | KEEP |
| ENV-58 | Cockpit weather effects (WX) | Clouds/rain look around the aircraft | Condition code selects effect family; bounded 520 x 320 pass at ~12 fps; opt-in; nothing synthetic if data missing | Open-Meteo | DOCUMENTED 04 s15 | LATER |
| ENV-59 | Regional brief tool | What/where a place is, weather, headlines | Combines the three above | n/a | DOCUMENTED 08 | KEEP |
| ENV-60 | Situation brief | Weather, quakes, fires, aircraft, ships, cyclones, section by section | Unavailable sections marked | n/a | DOCUMENTED 08 | KEEP |

### H. Director / scenes

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| ENV-61 | Scenes panel | Scene selector, NEW, DEL, CAPTURE SHOT, UPDATE SHOT, shot list, START, STOP, NEXT, EXPORT PRESETS, IMPORT, RUN LOG, progress, status | State owned by one director object; panel is a thin view | browser storage | DOCUMENTED 09 s1.1 | KEEP |
| ENV-62 | Capture shot | Saves live camera, visual look, all registered layer states | Defaults 4 s flight, 0.9 s hold | n/a | DOCUMENTED 09 s1.3 | KEEP |
| ENV-63 | Update shot | Overwrites selected shot with live state | Keeps move start and easing; anchors untouched | n/a | DOCUMENTED 09 s1.3 | KEEP |
| ENV-64 | Shot rename/delete, scene new/delete | Double-click rename; confirmations | n/a | n/a | DOCUMENTED 09 s1.1 | KEEP |
| ENV-65 | LOAD a shot | Applies one shot without running | Newest request wins; blocked during run | n/a | DOCUMENTED 09 s1.3 | KEEP |
| ENV-66 | NEXT | Loads next shot, wraps | Over the whole queue starting at the selected scene | n/a | DOCUMENTED 09 s1.3 | KEEP |
| ENV-67 | START / STOP / Escape | Whole-project run from selected scene; Escape stops | See explanation; START plays all scenes, wrapping | n/a | THIN (START scope) 09 s1.1 | KEEP |
| ENV-68 | Voice scene control | list, play (single scene), stop, next, status; scenes found by id or partial title | Same director | n/a | DOCUMENTED 08/09 | KEEP |
| ENV-69 | Run log | JSON of last run: start, estimate, counts, timestamped events | In-memory, downloaded locally | n/a | DOCUMENTED 09 s1.4 | KEEP local-only |
| ENV-70 | Playback presentation mode | Panels hidden, safe frame, scene-playing mode | See recording section | n/a | DOCUMENTED 09 s1.2 with errors, see Corrections | KEEP |
| ENV-71 | Ordinary shot flight | Eased flight to pose | Engine flight, cubic in/out, minimum 0.2 s, safety timeout | n/a | DOCUMENTED 09 s3.1 | KEEP |
| ENV-72 | Authored camera move | Start-to-end with easing | Per-animation-frame sampler, shortest arcs | n/a | DOCUMENTED 09 s3.2 | KEEP |
| ENV-73 | Anchors | Named scene positions | Up to 1,024; ellipsoid heights only | n/a | DOCUMENTED 09 s2.1 | KEEP |
| ENV-74 | Layer reconciliation | Only named layers touched; refused layers reported; tracking never re-established; isolating context mode left first | Abortable transitions | n/a | DOCUMENTED 09 s3.4; no-restore THIN | KEEP |
| ENV-75 | Seek / scrub | Scrubber in event panel jumps to any scene time | Pure timeline maths; newest wins | n/a | DOCUMENTED 09 s3.6; sampler difference THIN | KEEP |
| ENV-76 | Media-bounded holds | Hold waits for evidence clip, 20 s cap | Fail-closed stop | n/a | DOCUMENTED 09 s3.6 | KEEP |
| ENV-77 | Data packs (GeoJSON, PNG image, media) | Shapes/overlays/markers per shot with attribution cards | Source registrations only, caps, hashes, 15 s deadline | n/a | DOCUMENTED 09 s4 | ADAPT |
| ENV-78 | Declarative click actions | Card / focus / shot / layer buttons after load | 64 transitions cap | n/a | DOCUMENTED 09 s6 | KEEP |
| ENV-79 | Import with preview | Preview then Apply/Cancel | Validates before replacing | n/a | DOCUMENTED 09 s5.1 | KEEP |
| ENV-80 | Validation and migration | Limits table; versions 1-6 | Field-path errors | n/a | DOCUMENTED 09 s5.2-5.3 | KEEP |
| ENV-81 | Storage protection | Unreadable saved project is never overwritten | Temporary defaults + warning | browser storage | DOCUMENTED 09 s3.7 | KEEP |
| ENV-82 | Edit details | Validated draft editor, capture camera as anchor | n/a | n/a | DOCUMENTED 09 s5.4 | ADAPT with real editor |
| ENV-83 | Share scene / asset bundle | JSON of one scene; bundle with hashes | Re-import after reload | n/a | DOCUMENTED 09 s5.5 | KEEP |
| ENV-84 | Five generic showcase scenes | Flights Radar, Orbital Watch, Thermal Threat Board, City Overload, Omniscience Pullback | Recipes name only four layers | n/a | DOCUMENTED 09 s7.1 | ADAPT |
| ENV-85 | Nepal flood scene (25 shots) | Evidence-led story over Google 3D or Esri | Event panel, schematic flood path, evidence cards | Event pack (CC BY-NC) | DOCUMENTED 04 s14, 09 s7.2 | SKIP content, KEEP mechanism |
| ENV-86 | Event panel | Imagery split slider, reconstruction clock, beats, PLAY, PLAY SCENE, CINEMATIC, FULL STORY, OPEN ORIGINAL, LOWER GORGE, field reports | PLAY SCENE continues from the current shot without wrapping and without presentation mode | n/a | DOCUMENTED 04 s14; PLAY SCENE scope THIN | ADAPT |
| ENV-87 | Flood path reconstruction | Amber corridor revealed reach by reach | Schematic over a published centreline; prefix rebuilt on seek | CC BY-NC | DOCUMENTED 04 s14 | SKIP content |
| ENV-88 | Evidence cards and field reports | Cards with poster/embed or source link | Never bundled | n/a | DOCUMENTED 04 s14 | KEEP |
| ENV-89 | Flood tool | Evidence trail, path, imagery dates for assistants | n/a | n/a | DOCUMENTED 08 | SKIP |
| ENV-90 | Built-in scene install and upgrade | Seeded, upgraded idempotently, deleted ones not resurrected | Installed-id list | n/a | DOCUMENTED 09 s3.7 | KEEP |

### I. Annotations, draw, measure

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| ENV-91 | Voice annotate (pin, highlight, area, arrow, route, label) | Marks on the world | One engine; up to 24 per call; five colours | n/a | DOCUMENTED 07 s5.3 | KEEP |
| ENV-92 | Real outlines for areas | Actual boundary, not a circle | Bundled admin packs first, then geocoder + OSM outline with retry 8 s / 25 s; honest pin if none | OSM, Natural Earth | DOCUMENTED 07 s5.3, 04 s15 | ADAPT |
| ENV-93 | Routes | Street-following drive/walk/cycle with distance and time | Public routing; direct labelled line if unavailable | OSRM | DOCUMENTED 07 s5.3 | ADAPT |
| ENV-94 | Distance arrows (airport to airport) | Connector arrow, spoken distance, endpoints pinned | Both endpoints must resolve; great-circle distance appended; "the thing vs around the thing" intent; entity kind hints (building, compound, district, street, point feature) | n/a | DOCUMENTED 07 s6; intent field THIN | KEEP |
| ENV-95 | Persistence and caps | Marks stay until cleared; 120 max; non-persistent 22 s with 1.2 s fade | Geometry de-duplication | n/a | DOCUMENTED 07 s5.3 | KEEP |
| ENV-96 | Clear annotations | Voice or Draw Clear | Wipes all | n/a | DOCUMENTED 07 s5.2 | KEEP |
| ENV-97 | Draw tool (Area, Line, Pin) | Click vertices; double-click or Enter; Backspace; Esc | Pointer lease; 512 vertices; 0.5 m dedupe; closed ring; draped; unwrapped longitudes | n/a | DOCUMENTED 07 s5.2 | KEEP |
| ENV-98 | Live measure | Running length (m/km) or area (m2/ha/km2) | Great-circle length; planar area | n/a | DOCUMENTED 07 s6 | KEEP |
| ENV-99 | Camera assist for voice marks | Frames offscreen marks, 2.6 s stand-down | Region cap ~290 km | n/a | DOCUMENTED 07 s5.3 | KEEP |
| ENV-100 | Fly the route | Cinematic dolly along drawn route | One camera-motion slot | n/a | DOCUMENTED 06/08 | KEEP |
| ENV-101 | Import own files (GeoJSON, KML, KMZ, GPX) | Rows with show/hide, fly-to, remove | 12 files, 50 MiB, caps | n/a | DOCUMENTED 07 s5.4 | KEEP |
| ENV-102 | Annotations in links and embed | Up to 24 marks in 6,000 characters drawn after restore | Compact encoding | n/a | DOCUMENTED 02 C | KEEP |
| ENV-103 | Console test surface | Deterministic no-microphone demo/list/clear/count | n/a | n/a | MISSING (developer only) | SKIP |

### J. Recording, safe frame, share

| ID | Feature | What the user sees/does | How it works | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| ENV-104 | Recording mode | Chrome hidden except scene runtime; HUD set | Body class; pre-recording HUD snapshot once and restored | n/a | DOCUMENTED 07 s7 with errors | ADAPT |
| ENV-105 | Safe-frame overlay | 16:9 or 9:16 box with thirds guides | See gap G3 for geometry | n/a | THIN | ADAPT; add 4:5, A4 |
| ENV-106 | Clean view | Hides chrome, keeps credits | n/a | n/a | DOCUMENTED 07 s7, 02 | KEEP |
| ENV-107 | Share link copy | Button copies address plus copy time | Debounced hash state | n/a | DOCUMENTED 02 C | KEEP |
| ENV-108 | Share restore and follow | 1.5 s delay, 3 s flight, ACQUIRING window | Cancelled by user steering | n/a | DOCUMENTED 02 C | KEEP |
| ENV-109 | Open-in-app link tool | Link that opens app over an area with layers, style, annotations | Same view encoding | n/a | DOCUMENTED 08 | KEEP |
| ENV-110 | Embed mode | Globe-only page accepting view messages | n/a | n/a | DOCUMENTED 08, 02 | LATER |

Features listed: 110 (ENV-01 to ENV-110).

---

## Gaps found (filled in full)

### G1 - Wind particle technique (THIN: limits documented, mechanism not)
Written in full in "How the wind particle flow works" above. Points that were absent from 04 section 11 and are needed to rebuild it: equal-area golden-angle seeding; two-way half-hour integration, 16 steps each way (33 points); midpoint integration with a per-step distance cap; stopping rules (calm 0.15 m/s, polar 88.5 degrees, seam, missing data, fewer than 3 points); one-time bake per field, not per camera move; 30 degree cell grouping with per-frame horizon and frustum culling; animation by a single phase value with per-path golden-ratio offsets (head-and-tail comet shape, period about 6.25 s) and a ghost line (alpha 0.25, 0.34 paused); the canvas fallback (8-12 s life, camera-height speed scaling, trail half-life 0.65 s); scalar changes reuse existing baked geometry when model, times, grid and wind arrays are unchanged.

### G2 - START scope and PLAY SCENE (THIN)
START does not run only the selected scene: it queues the selected scene's shots then every other scene in project order, wrapping. Only voice "play" runs a single scene. In a fresh project that means START on the first scene plays all six built-ins back to back. The event panel's PLAY SCENE (Nepal) continues from the current shot to the end of that one scene, does not wrap, and does not turn on the recording presentation (panels stay visible, which is why the event panel stays usable). Both are runs that claim the camera and refuse to start while another is active. Kawn should expose "play this scene" and "play all" as separate controls.

### G3 - Safe-frame geometry and what actually drives it (THIN + error, see Corrections)
Geometry of the guide: a full-screen non-interactive layer above the map and below popovers; a 1 px outline at about 28% white with a 1 px dark halo; thirds guide lines at about 20% white; 16:9 box width is the smaller of 90% of viewport width and 90% of viewport height times 16/9; 9:16 box width is the smaller of 58% of viewport height and 76% of viewport width. Behaviour: HUD button state follows; the HUD mode snapshot before recording is taken once and restored on exit; recording hides chrome but keeps the scene-runtime readout and the credits line.

### G4 - No restore after playback (THIN)
Scene playback has no restore pass. Layers a shot names are left as the last shot set them; layers it does not name were never touched; only the scene's own release list is switched off at the end. The user's HUD mode and layout are restored; map-source and visual look are not described as restored. Captured shots name every registered layer, so a captured scene fully re-dresses the globe; recipe scenes name four (flights, satellites, earthquakes, traffic). Kawn should add an explicit "restore my view after the tour" option.

### G5 - Weather history depth per product (THIN)
Frames per product are capped at 13 (from at most 26 recent advertised times). At the sources' cadences this implies roughly: radar about 50 minutes, regional infrared about 65 minutes, lightning about 3 hours, global infrared about 12 hours of history (derived, UNVERIFIED as measured). The slider therefore spans the union and is dominated by the longest history, while short-history products hide for early ticks (and say "No frame within N min"). Kawn: choose history depth per product deliberately and display it.

### G6 - Seek versus live flight for ordinary shots (THIN, see Corrections)
Ordinary shots fly with the engine's own flight, which for long hops can arc upward; seek and scrub use a straight eased interpolation of position, so scrubbing and live playback show different in-between views for ordinary shots. Only authored moves share one sampler between live and seek. Endpoints agree.

### G7 - Scene clock and runtime readout (THIN)
A scene clock publishes elapsed time, scene duration and running/seeking/stopped state to subscribers (the event panel uses it for its scrubber). A separate progress timer drives the bar. Stop settles pending holds immediately and stale ticks cannot affect a replacement run. The total duration counts every shot's flight plus its effective hold, where effective hold is at least the media and path floors of that shot.

### G8 - Entity-kind and intent hints in voice annotations (THIN)
The annotate action accepts an intent ("the thing" or "around the thing") and an entity kind (building, compound, district, street, point feature), a footprint flag, a mode (walking, driving, cycling), screen-fraction coordinates and a persist flag. These steer which outline is sought (compound-sized for airports, hospitals, universities, parks, stadiums, campuses; envelope for districts) and let an annotation be placed by pointing. Airport-to-airport distance arrows rely on the compound-size outline centre.

---

## Corrections to existing docs

1. 09 section 1.2 and section 7.1 (City Overload "9:16 vertical"): the 9:16 frame is never applied in playback. Starting a preview run always requests the 16:9 frame with the full HUD mode; the recipe's "hide panels / HUD mode / safe frame" fields are not used for the recording chrome (only the HUD mode is converted into each shot's HUD visual setting, which then overrides the "full" request shot by shot). 9:16 exists in the overlay but nothing in the studied app asks for it.
2. 07 section 7 ("turned on by scene playback ... and by tools") and 07 section 9 (Share link row "recording state"): only scene preview runs turn recording mode on; no voice or automation tool does. The share link does not carry recording state; the recording flag appears only in the control-state readback.
3. 09 section 3.6 ("Direct load, forward/backward seek and replay all use the same camera sampler as live playback"): true for authored moves only; ordinary shots fly with the engine flight live but are scrubbed with straight eased interpolation (G6).
4. 09 section 1.1: START description should say it plays the selected scene then the remaining scenes, wrapping (G2).
5. 04 section 15 (Draw tool "Mentioned for completeness"): covered fully in 07 section 5.2; no conflict. 04 section 11 "Wind: newest run chosen by nearest cycle/forecast step" is correct; its "equal-area" technique was missing (G1).
6. No numerical errors found in 03 s11-13 or 04 s1-4, s11-14 spot checks (earthquake disc radius 2^magnitude km, depth bands, label cap 96, FIRMS bands and caps, perimeter bands, cable and data-centre budgets all agree with source).

## UNVERIFIED

- Native-GPU behaviour and frame cost of the wind shader (original's own docs say not verified).
- Real history span per weather product (G5 is derived).
- Whether the engine's flight arcs for the specific ordinary-shot distances used in bundled scenes (inferred from engine default behaviour).
- Fire detection card and perimeter card appearance on narrow screens; safe-frame look on non-16:9 screens (geometry only from style rules).
- Behaviour when two imported scenes collide with built-in ids (replace-all stated; no merge described).
- Accessibility focus trapping in the edit-details and import-preview dialogs.
- Exact total of Nepal shot holds at runtime (depends on media and path floors).
- No live run, no network call and no screenshots were made.

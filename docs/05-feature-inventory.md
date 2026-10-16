# Feature inventory: the studied app, with a verdict for Kawn

Verdict key: **KEEP** = build in Kawn as observed, **ADAPT** = build, reshaped for the UAE city-intelligence product, **LATER** = post-MVP, **SKIP** = do not build.
The Doc column points to the behaviour doc in reverse-engineering/ (number, then section where useful). Those docs are the source of truth;
this page is the checklist view. Layer-by-layer product decisions live in specs/02-layers; UX ideas in specs/01-product-ux ("Ideas to do differently").
Pass-2 corrections are already applied here (for example: the Display panel starts collapsed, Sharpen defaults on at 49 %, the original has no bottom sheets).

## A. Map, camera and navigation
| Feature | Verdict | Kawn note | Doc |
|---|---|---|---|
| 3D globe, keyless satellite basemap as default | ADAPT | City-first 2.5D map, not a globe; keep the keyless safety net | 05 s1 |
| Five map sources with fallback ladder and honest "unavailable" reasons (photoreal 3D, two Bing, Esri, OSM) | ADAPT | Photoreal 3D is the hero for the three emirates; drop Bing unless licensed; add Arabic labels | 05 s1 |
| Terrain (keyless open terrain, world terrain with a token) | LATER | Gulf is flat; imagery plus building footprints first, ground-height sampler for tower roofs | 05 s1.3 |
| Photoreal 3D tiles behaviour, sky and atmosphere | ADAPT | Metered, server-side key | 05 s1.4-1.5 |
| Camera model: one owner at a time, manual input always wins, refusal toasts | KEEP | | 06 s1 |
| Ground-collision guard (predict, land, measure, lift) | KEEP | Essential for towers over flat ground | 06 s1.7 |
| Fly-to framing per landmark (range, pitch, heading) | ADAPT | Curate 20-40 Dubai/Abu Dhabi/Sharjah landmarks | 06 s1.5 |
| Top action row: clear layers, copy link, tilt (about 89 vs 35 degrees), north up, reset | KEEP | Reset becomes "Reset to city" | 01 s2.3, 06 s1.6 |
| Slow orbit of a landmark | ADAPT | UI button, not a hotkey | 01 s2.7, 02 A.2 |
| Location tray: nine city pills, five landmark pills each, key hints | ADAPT | Dubai, Abu Dhabi, Sharjah, five landmarks each | 01 s4.1 |
| Search: coordinates and bundled names first, then remote geocoders, superseding searches | ADAPT | Bilingual AR/EN, transliteration-aware, offline gazetteer of districts | 01 s4.1, 04 s15 |
| Offline admin outlines (country/state/county) and neighbourhood polygons | ADAPT | Replace with emirate > area > community polygons | 04 s15 |
| Share link: camera, look, layers, options, tracked target, panel state, up to 24 marks | KEEP | Plus persona, shortlist, time-of-day (see section I) | 02 Part C |
| Share-link restore rules (link wins, fail closed, recipient never yanked) | KEEP | | 02 C.3-C.5 |
| Tracked-subject hand-off with expiry | ADAPT | Expiry only for live subjects | 02 C.4 |
| Session vs persistent preferences | ADAPT | Persist per profile, not only in the URL | 02 C.6 |
| Mouse/trackpad/touch gestures (engine defaults, 6 px / 400 ms click rule, pinch scaling) | ADAPT | Define a real touch gesture set | 02 Part B |
| Drag-and-drop file import | KEEP | See section E | 02 Part B, 07 s5.4 |
| Embed mode (globe only, posted-message control) | LATER | Partner embeds | 01 s12, 08 s9 |
| Clean view (hide chrome) | KEEP | | 01 s2.5, 05 s6 |
| FPS readout, idle render governor, hidden-tab pause | KEEP | Readout in developer menu only | 01 s2.2, 05 s8 |

## B. Live layers (what the layer panel offers)
Panel anatomy: six groups, ON/OFF state button, count, source-and-age meta line, sub-controls only while on, all layers off on a fresh profile (01 s5.1, 03 s0). Kawn regroups into Mobility, Real estate, Services, Environment, Events, Utilities.

| Layer | Verdict | Kawn note | Doc |
|---|---|---|---|
| Layer row states ON/LOADING/DEGRADED/STALE/PARTIAL/FALLBACK/UNAVAILABLE, "KEY REQUIRED" guidance | KEEP | Translate; keys stay server-side so users never see key prompts | 01 s5.1, 03 s0.2 |
| Honesty rules (estimated labels, "not an all-clear", insufficient data is not clean) | KEEP | Translate in full, do not shorten | 03 s0.2 |
| Civil aircraft (OpenSky, regional fallback, 30 s, 3-line card, tracked) | KEEP | DXB, DWC, AUH, SHJ; commercial feed needed (licence) | 03 s1 |
| Military aircraft | SKIP | | 03 s2 |
| Local ADS-B receiver feeds (own receivers, merge rules) | LATER | Operator-only; reuse implausible-jump filter | 03 s3 |
| GNSS integrity (anomaly cells, 250 nm circle) | LATER | Real Gulf interference; keep wording discipline | 03 s4 |
| Vessels (AIS, 60 s, ambient and selected cards) | ADAPT | Gulf box, Jebel Ali, Khalifa, Port Rashid, creeks, dhows | 03 s5 |
| Satellites with pass prediction (class card, dense Starlink chip) | LATER | Pass prediction had no UI in the original; could be a panel | 03 s6 |
| Launches / space missions (past 30 days only) | SKIP | Low UAE value | 03 s7 |
| Transit (operator realtime, 15 s, vehicle card; no filter UI in the original) | KEEP | RTA Metro, Tram, bus; Abu Dhabi ITC; Sharjah | 03 s8 |
| Bikeshare (station card) | LATER | | 03 s9 |
| Street traffic (OSM roads plus flow, simulated-vs-live honesty) | KEEP | Keep the honesty label | 03 s10 |
| Earthquakes (no click card in the original) | LATER | Low priority | 03 s11 |
| Fire detections and perimeters | SKIP | Not a UAE need | 03 s12-13 |
| Submarine cables | SKIP | Non-commercial dataset | 04 s1 |
| Data centres (16 UAE entries, noisy) | LATER | Authoritative local dataset and richer click panel | 04 s2 |
| Dams (0 in UAE) | SKIP | | 04 s3 |
| Installations (military and critical sites) | SKIP | | 04 s4 |
| Mapped ALPR cameras | SKIP | Plate surveillance | 04 s5 |
| CCTV / public camera network, calibration, coverage, auto-hop | SKIP | Privacy law; only explicitly licensed feeds if ever | 04 s6, 01 s6.2 |
| Recent imagery before/after (box, daily cards, swipe divider) | ADAPT | Construction progress, START HERE ranking | 04 s7, 01 s6.5 |
| Street-level photos (Mapillary, KartaView, Street View, one provider at a time) | LATER | Check UAE coverage first | 04 s8, 01 s6.3 |
| Radio (station directory, tuner, hand-off with voice) | LATER | UAE FM/AM streams | 04 s9 |
| Local RTL-SDR card (WebUSB) | SKIP | Hobbyist tier only | 04 s9.1 |
| Directions / routing with step list | ADAPT | Metro-aware, bilingual steps | 04 s10 |
| Wind (model, field overlay, units chips, inspect at centre) | KEEP | Shamal and dust emphasis | 04 s11 |
| Observed weather: rain radar, satellite clouds, lightning, one shared timeline | ADAPT | Keep the timeline pattern; US-only radar and lightning replaced by Gulf products | 04 s12 |
| Cyclones | SKIP | Basin not covered | 04 s13 |
| Flood event scene data (Nepal pack) | SKIP | Non-commercial, unrelated region; reuse the mechanism only | 04 s14, 09 s7.2 |
| Awareness / Contacts window (250 km, cohorts) | ADAPT | Default 5-15 km, expand to emirate; "nearby flights/vessels/transit" | 03 s0.4, 06 s3 |
| Local weather readout and regional news briefing | ADAPT | Bilingual headlines, heat/dust advisories | 04 s15, 06 s4.5 |
| Per-layer caps, altitude gating, hysteresis, declutter | KEEP | Tie budgets to device class | 03 s0.5, 05 s8 |

## C. Interaction, tracking and panels
| Feature | Verdict | Kawn note | Doc |
|---|---|---|---|
| Click-to-track: camera lock, fading trail, telemetry card, deselect in place | KEEP | | 06 s2 |
| Flight-plan estimate card with "estimated" label and plausibility check | KEEP | | 06 s2.5 |
| Contacts roster: previous/next, visited history, doubling-radius search, filters | ADAPT | UAE cohorts (metro, flights near airports, vessels at ports) | 06 s3 |
| Cockpit (first-person aircraft view, instruments, briefing carousel, weather effects) | LATER | Tourist ride-along (DXB approach, Palm helicopter); mirror tapes in RTL | 06 s4, 01 s8 |
| Camera-feed focus policies | SKIP | Goes with CCTV | 06 s5 |
| Panel system: collapsible, explicit vs automatic collapse, disclosure memory | KEEP | Mobile: bottom sheets | 01 s3 |
| Floating (portable) panels, drag and resize | SKIP | Mouse-only; bottom sheets on mobile | 01 s3.5, 02 Part B |
| Dock trays with hover-open (140 ms) and pin | ADAPT | Tap-to-open sheets on touch | 01 s3.7, s4 |
| Display panel (HUD, detect, 3D, scope, draw, import, bloom, sharpen; starts collapsed) | ADAPT | Reorganise for civic use | 01 s6.1 |
| Right-rail policy and Context panel (Contacts, Missions, Radio, SDR) | ADAPT | Contacts only | 01 s3.3, s6.6 |
| Loading chip (160 ms delay), toasts, guidance-not-error states | KEEP | | 01 s2.4, s14 |
| Brand block, style indicator, world-label accessibility list | KEEP | Brand from brand.json | 01 s2.1, s2.8 |
| First-run launcher with four tiles; never shown to link recipients | ADAPT | Persona entries (investor / visitor / government) | 01 s7.1 |
| Provider settings "POWER UP" panel and key chip | SKIP | Operator admin console instead | 01 s7.2, 11 s3 |
| Credits lightbox, always-visible attribution | KEEP | Bilingual, per-layer provenance | 01 s7.3, 07 s8 |
| Keyboard: looks 1-7, H, O, V, F, D, C (three meanings), backtick, Q-T landmarks, Escape tiers | ADAPT | Physical-key codes, modifier checks, in-app shortcut sheet, distinct gestures | 02 Part A |
| Hold-Space push-to-talk | ADAPT | Visible hold button plus key; test with Arabic input methods | 02 A.2, 08 s2 |
| Easter eggs (novelty reclassify button) | SKIP | | 06 |

## D. Looks and rendering
| Feature | Verdict | Kawn note | Doc |
|---|---|---|---|
| Seven looks: Normal, CRT, NVG, FLIR, Anime, Noir, Snow; 500 ms crossfade; per-look sliders | ADAPT | Day, Night city, Heat/Comfort, Dust/Haze, Blueprint/Print, Noir; drop CRT, NVG, FLIR, Snow, Anime | 05 s2 |
| Looks' side effects (auto HUD and dense detection on military looks) | SKIP | | 05 s2.3 |
| Bloom (off) and Sharpen (on at 49 %) | KEEP | | 05 s3 |
| Scope / keyhole mask with feather | KEEP | Signature visual, altitude-adaptive edge | 05 s4 |
| Celestial ring | SKIP | | 05 s5 |
| 3D aircraft models, proximity swap, hysteresis | ADAPT | Own licensed assets (A380/777, helicopters) | 05 s7 |
| Design language and tokens | ADAPT | See 06-design-system | 05 s9 |

## E. HUD, detection, annotations
| Feature | Verdict | Kawn note | Doc |
|---|---|---|---|
| HUD with four layouts (Tactical, Operator, Minimal, Cyber), telemetry fields, cadence | ADAPT | City status strip: place, time (UTC+4), heat, coordinates, scale | 07 s1 |
| Fake classification banners, MGRS, NIIRS, decorative edge text, REC dot | SKIP | | 07 s1.3 |
| Cyber theme and sonar | SKIP | | 01 s6.1, 07 s1.4 |
| Intel summary line (deterministic, optional AI, feed-state token) | ADAPT | Per-language length budget | 07 s2, 08 s6 |
| Detection overlay: budgets, density, allocation, collision grid, fade, leader lines | ADAPT | Label POIs/vehicles; tune for dense downtown and mobile | 07 s3 |
| Draw / whiteboard: area, line, pin, undo, measure | KEEP | Add buffer, corridor, hectares, saved boards | 07 s5.1-5.3 |
| Import GeoJSON/KML/KMZ/GPX (12 files, size caps, stays on device) | KEEP | | 07 s5.4, 01 s6.1 |
| Measurements (distance, area, route estimates) | ADAPT | Metric default, summer walking time, drive time | 07 s6 |
| Recording mode and safe frames (16:9, 9:16) | ADAPT | Present mode, share-as-image/PDF brief | 07 s7 |
| Accessible mirror list of visible targets | KEEP | | 07 s3, 01 s2.8 |

## F. Voice and AI
| Feature | Verdict | Kawn note | Doc |
|---|---|---|---|
| Realtime voice session lifecycle, barge-in, push-to-talk and open mic | LATER | Flagship v1: Arabic (Gulf dialects) + English, local or UAE-hosted stack | 08 s2 |
| About 30 voice actions (navigation, console, annotation) | ADAPT | Alias every layer, panel and place in AR and EN | 08 s4.1-4.4 |
| 25 world-query tools and analyst answers (count, filter, rank, scope words) | ADAPT | City objects: permits, transactions, transit, air quality | 08 s4.5, s5 |
| Point-and-ask, numbered references, visual grounding | KEEP | Privacy review for screenshots | 08 s2.8-2.10 |
| Radio hand-off ducking | LATER | | 08 s2.7 |
| Cost meter and server throttles | ADAPT | Compute budget instead of dollars if local | 08 s7 |
| Voice via ChatGPT-login reuse | SKIP | Account-terms risk | 08 s3 |
| MCP server (30 tools, 29 model-visible) | LATER | Expose city-intelligence queries | 08 s9, 10 s2.10 |
| Redacted diagnostics, privacy of audio | KEEP | Consent for microphone and screenshots | 08 s2.13, s8 |

## G. Director, scenes, sharing
| Feature | Verdict | Kawn note | Doc |
|---|---|---|---|
| Scenes panel: capture shot, shot list, start/stop/next, progress | ADAPT | "City stories" | 09 s1, 01 s5.2 |
| Shot model, anchors, ellipsoid heights, authored moves, seek-equals-playback | KEEP | | 09 s2-3 |
| Declarative interactions (version 6), 64-transition cap | KEEP | | 09 s6 |
| Data packs with attribution, licence, hashes, size caps | KEEP | Emirate GIS packs | 09 s4 |
| Import with preview, validation, migration; invalid import changes nothing | KEEP | | 09 s5 |
| Share a scene / bundle with hash check | KEEP | Add audience flags and approval stamp | 09 s5.5 |
| Five generic showcase scenes | ADAPT | Palm, Marina/Downtown, Expo City, Creek Harbour, Corniche, heritage Sharjah | 09 s7.1 |
| Incident scene template (Nepal flood pattern) | ADAPT | Flood, dust storm, road closure; "schematic" label | 09 s7.2 |
| Run log download | SKIP | | 09 s1.4 |
| Browser-only scene storage | ADAPT | Server-side with permissions | 09 s3.7 |

## H. Platform and ops
| Feature | Verdict | Kawn note | Doc |
|---|---|---|---|
| Server as key broker, short-lived tokens, same-origin browser, client-key allow-list | KEEP | | 10 s1, 11 s3 |
| About 40 upstream sources with licence notes | ADAPT | Replace with UAE sources; licence ledger as release gate | 10 s3, 11 s1 |
| Uniform record shapes, health states, serve-stale, single-flight, caps, timeouts | KEEP | | 10 s5-6 |
| No client-supplied URLs, per-hop redirect checks, DNS-pinned fetch | KEEP | | 10 s4 |
| Per-IP throttles, in-memory caches, local-file caches | ADAPT | Per-tenant quotas, shared cache | 10 s4-5, 11 s4 |
| Bundled local data (outlines, landmarks, models) | ADAPT | UAE equivalents | 10 s7 |
| Health and setup reporting | KEEP | | 10 s8 |
| Zero-key baseline with graceful degradation; key matrix | KEEP | Key registry per UAE provider | 11 s1-2 |
| Presence-only key status, atomic permission-first writes | ADAPT | Secrets manager and admin console | 11 s3 |
| Outbound destinations list, no telemetry found | KEEP | CSP allow-list, no analytics without consent | 11 s5-6 |
| Install modes (one-click installer, LAN sharing) | SKIP | Hosted service | 11 s8 |
| Testing strategy and CI gates (large unit suite, hermetic browser gates) | KEEP | Add RTL, accessibility, licence lint | 11 s9 |

## I. Kawn-only additions (not in the original)
Real estate layer (DLD transactions, prices per sqm, off-plan), developer/project pages, Arabic-first RTL, persona modes (investor / tourist / planner),
city switcher (Dubai, Abu Dhabi, Sharjah), city-growth timeline slider, city brief card per district (permits, prices, traffic, weather), shortlist of up to 4
places, signed short-lived links for government views, "nearby flights" idea from the Contacts roster. Detail: specs/01-product-ux.

## J. Not in the original: gaps Kawn must design
- **Mobile:** the original is the desktop layout squeezed into two scroll bands; no bottom sheets, swipe, long-press, or touch parity for keyboard-only features (01 s11, 02 Part B). Design bottom sheet (peek/half/full), 44 px targets, tap-to-open trays, touch gesture set.
- **Arabic and RTL:** no RTL, no Arabic, mono uppercase micro-labels, English-only copy (01 s13, 05 s9). Design mirrored rails and dock, bidi-safe IDs, numerals toggle, Arabic-capable type, per-language string budgets; compass, scope and pitch rails stay geographic.
- **Accessibility:** good focus and ARIA basics, but no screen-reader description of the map, tiny text, colour-leaning chips (01 s13). Design AA contrast, text alternatives for the map, shortcut sheet, no colour-only meaning.
- **Real estate:** none exists. Needs transaction/permit data model, parcel and project geometry, price layers, developer pages, provenance (official vs modelled), PDPL-aware ownership data.
- **Auth and roles:** no accounts; keys live in a local settings panel (11 s3, 10 s2.9). Design sign-in, roles (public / partner / government), audit log, per-dataset licence and redistribution flags, per-tenant quotas.
- **Also missing:** Gulf time zone logic (the original derives time from longitude; Abu Dhabi is mislabelled by its region label, so Kawn owns its region logic), data residency, and a privacy and consent notice.

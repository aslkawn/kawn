# 13b. Feature census and gap audit: aircraft, tracking, contacts, cockpit

## Purpose and method

Purpose: an independent list of every behaviour in the studied application's air domain, explained at technique level so a clean-room team can rebuild it, compared against our existing notes (00 to 12, mostly 03, 05, 06, 07, 08). Domain covered: civil flights, military flights, local ADS-B receivers and the in-browser radio decoder, GNSS integrity, aircraft classification and 3D models, route and type enrichment, trails (and the absence of replay), click-to-track and the follow camera, the contacts roster, cockpit mode, airport and runway ground handling, emergency detection, per-aircraft cards, the detection overlay on aircraft, and analyst/voice/MCP queries about flights.

Method: enumeration came first, from the studied project's own readme, known-issues note, current-state note, local-receiver guide, tool catalogue, change log, then the whole civil, military, local-receiver and GNSS layer folders, the data helpers (classification tables, emergency rules, motion maths, tracked-camera rules, height-datum chain, ground-floor and ground-snap services), the server proxies (state-vector, regional fallback, military feed, track backfill, adsbdb cache, local receivers, GNSS integrity), the radio decoder and its controller, and the cockpit modules. Test titles were used as statements of intended behaviour. The code-graph tool (GitNexus) was used for symbol location only; it returned no execution-flow processes for these concepts, so structure was confirmed by reading source directly. Then each feature was checked against our notes and labelled DOCUMENTED (note and section), THIN (present but missing mechanism, numbers, or limits) or MISSING.

Reading guide: "Kawn verdict" is KEEP, ADAPT, LATER or SKIP; "UAE" is relevance to Dubai, Abu Dhabi and the Gulf. Numbers are taken from source; statements about radio, avionics and geodesy standards are general domain knowledge and are marked as such. No code, identifiers or file names appear here by design.

Overall finding up front: our existing notes (03 sections 1 to 4, 06 sections 2 to 4, 08) already capture nearly all user-visible behaviour of this domain with real numbers. The gaps are almost entirely at the mechanism level: how a radio frame becomes a position, how the altitude datum chain works, how the ground floor tiers cooperate, the exact smoothing algorithm, which class tables exist, and several small cadence and limit values. Those are written out in full in section 4.

---

## 1. How positions are fetched, smoothed, extrapolated and rendered (plain words)

### 1.1 Where positions come from

There are four independent position streams in this domain. None needs a user key to run.

1. **Civil snapshot (public Flights layer).** The browser asks the app's own server for a "current picture" every 30 seconds. The server holds one worldwide state-vector snapshot from the OpenSky Network (a research network of volunteer ADS-B receivers; state vectors are per-aircraft rows: address, callsign, origin country, position time, last-contact time, longitude, latitude, barometric altitude, on-ground flag, speed, track, vertical rate, geometric altitude, squawk, emitter category). The snapshot is not filtered to the viewport; roughly 11,000+ rows. The browser sends its view centre only so the server can use it if it must switch to the regional fallback. The server caches the snapshot about 9 seconds and shares it among all browsers.
2. **Credit governor (server).** One worldwide call costs 4 of roughly 4,000 daily credits for an authenticated account. As the remaining budget shown in the upstream response headers thins, the server stretches how long it reuses a cached snapshot (9 s while more than 2,400 credits remain, 30 s above 1,200, 90 s above 400, then 5 min), so a browser left open all day does not exhaust the quota. On a rate-limit answer it stops calling upstream until the stated retry time (bounded 30 s to 30 min, 2 min if none is stated) and serves the last good body flagged stale. A cold start into a rate limit with nothing cached synthesises the 429 locally. Each upstream attempt has a 10 s deadline and one retry.
3. **Regional fallback.** If the snapshot call fails, or the snapshot's own timestamp is more than 2 minutes old, the server instead asks the adsb.lol community aggregator for aircraft within 250 nautical miles of the view centre (rounded to 0.25 degree so nearby views share a cache entry; 12 s cache; 80 anchors kept; response capped at 8 MB), converts them to the same row shape (adding the ADS-B emergency field the primary feed lacks), and labels the response with the source and coverage so the browser can show "regional fallback". About 250 aircraft near Dubai and about 830 near London were seen in the live walkthrough.
4. **Military feed.** A separate server route returns the adsb.lol "military" list (hundreds of aircraft worldwide, selected by the aggregator's address ranges and flags), cached 12 s, with a cooldown honoured on 429 or 5xx while the last body is served as stale with its age. The browser polls every 15 seconds.

**Local streams** (section 2.3) are a fifth and sixth: a browser-opened USB radio and user-run decoder feeds, polled once per second.

### 1.2 Normalisation, stickiness and removal

Every row is reduced to one common internal shape. Units are converted (knots, feet, feet per minute to metres per second and metres). Fields that a source intermittently drops (callsign, speed, track) are **sticky**: a later poll that lacks a field does not erase the last good value, but a later poll that changes it always wins. Emergency status is the opposite: it is per-poll and not sticky except for one source-switch omission, so a stale 7700 cannot linger. An aircraft is deleted after 3 consecutive polls without it, or after a single missed poll if its last fix was below about 150 m barometric and under about 45 knots (probably landed; the on-ground flag from the feed lags the real touchdown by a minute or so, which previously left phantom aircraft at airports). A partially admitted snapshot keeps absent aircraft up to 5 minutes.

### 1.3 The 30-second "render behind" trick and dead reckoning

The key smoothing idea: the fleet is drawn **one poll interval in the past** (30 s for civil, 15 s for military). With two real fixes on either side of the displayed instant, the position is a straight interpolation between known points, so there is no extrapolation error and no snap-back when the next poll lands. Each aircraft keeps its last 5 fixes.

- **Between two fixes:** the position is interpolated linearly in Earth-fixed 3D; the displayed course is a blend of the reported track (itself interpolated in time, so a slow turn advances continuously) and the direction of the chord between the two fixes, weighted by ground speed (at hover the chord is just GPS noise; helicopters always use the reported track).
- **Warm-up (newly seen, not enough history yet):** the displayed time predates the first fix, so the oldest fix is extrapolated **backwards** (up to 60 s) along its own velocity. As history accumulates the icon glides forward into normal interpolation with no freeze and no backward jump. The tracked trail draws nothing during this window.
- **Beyond the newest fix (feed paused):** the newest fix is extrapolated forward ("coasting") along a constant-rate turn arc if the aircraft has a measurable turn rate, otherwise a straight line; coast time is at least 60 s past the last position and extends while contact messages keep arriving, capped at 5 minutes. A coasting aircraft is drawn at 45% alpha (stale fade). This is local east-north-up geometry rather than repeated latitude/longitude trigonometry, to keep the per-frame cost low.
- **Course smoothing:** displayed course is rate-limited (maximum 60 degrees per second, scaled down at low speed; a stalled frame cannot spend more than 0.25 s of elapsed time in one step), which hides the step at each fix boundary.
- **Tick rates:** the whole fleet is advanced at about 12 Hz (every 80 ms), not per frame; rotation is recomputed only when the camera pose changes or once per second while idle. The tracked aircraft is advanced every frame, and one cached position per frame is shared by the icon, the rotation, the trail head, the follow camera and the card so they can never disagree.
- **Discontinuity absorber (tracked aircraft):** if the raw position jumps by more than a plausible per-frame move (real speed times frame time times four, plus 25 m), the jump is turned into an offset that decays to zero over 900 ms. Steady-state lag is zero because the offset is zero whenever motion is continuous.
- **Local receivers differ:** no render delay (fixes arrive about once per second), forward extrapolation from the newest fix with the same arc model and a 3 km "snap instead of slide" threshold (section 2.3).

### 1.4 Rendering

- **Far view: billboards.** One batched billboard collection per layer. Each billboard is a nose-up white silhouette (one of 10 class glyphs, drawn at 96-unit vector source, rasterised at twice that density, displayed at about 20 px, 24 px when tracked) tinted by colour: white civil, amber #FFB800 military, red #FF4444 emergency, cyan tracked, magenta local. Rotation is computed by projecting the course vector into **window coordinates** every time the camera moves, so the nose points the right way at any pitch, including when the camera orbits a tracked aircraft; near straight-down views a screen-aligned rule with hysteresis takes over. A billboard behind the planet's limb is hidden by an explicit horizon-occluder test (the globe is hidden in the photoreal look, so there is no depth test to hide far-side planes). Distant aircraft **recede**: between about 3,500 and 4,500 km camera height a limb-relative factor shrinks far icons to as little as 45% scale and 35% alpha (never zero) so a whole-globe view is not a white blizzard.
- **Near view: 3D models.** Below an 800 km camera altitude, and only for on-screen aircraft, billboards hand over to glTF models; see AIR-45 to AIR-49 for caps and radii. A billboard remains the visual until its model is loaded and placed, so there is never a gap or a double image.
- **Ground placement:** parked or taxiing aircraft are lifted onto a sampled ground height; see section 1.6.
- **Draw order and picking:** a registry decides which object wins a click when markers stack (military above civil above local, tracked on top); clicking a trail line is claimed as a no-op so it never reads as empty space and deselects.
- **Labels:** ambient aircraft have no persistent text; only the tracked aircraft, DETECT labels and cards carry text.

### 1.5 The altitude datum chain (why aircraft do not sink into runways)

The globe needs height above the reference ellipsoid. Aviation feeds give barometric altitude (referenced to a pressure datum, roughly mean sea level, with weather-dependent error) and, for some aircraft, geometric (GNSS) altitude. The render height is chosen in priority order:

1. geometric altitude from the feed, used as-is (it is already ellipsoidal);
2. barometric altitude plus the local **geoid undulation** (a bundled EGM96 grid, about 2.7 MB, loaded lazily; undulation ranges about -106 to +85 m worldwide), labelled in the source as a visual fallback and never claimed exact;
3. for a grounded aircraft with no altitude: the cached ellipsoidal ground height at its position (current cell first, last poll's cell second because a taxiing aircraft is always one cell ahead of the warm batch);
4. last resort for a brand-new grounded contact with no data at all: the geoid surface itself (right to within metres at sea-level airports such as the Gulf's coastal fields);
5. if none is available this poll, hold the previous render height (a one-poll baro dropout must not snap the plane down by the geoid offset and back).

The aviation altitude (barometric, feet or flight level) stays separate and is what labels, route-plausibility and camera range maths read.

### 1.6 Ground floor tiers (airports and runways)

Four cooperating mechanisms keep ground traffic on the ground:

1. **Coarse floor grid:** ground height is cached in cells of three decimal places of a degree (about 111 m), sourced from a terrain height service. Reads on the render path are synchronous cache lookups only; the poll loop warms cells in batches. A clamp lifts anything below floor plus 1.5 m up to that value, so a sprite sits on the surface rather than z-fighting it.
2. **Rendered-mesh floor (photoreal 3D look):** the visible world is photogrammetry that sits above the bare-earth terrain by about 17 m at one US airport apron, so terrain-floored sprites still buried themselves. A one-shot sampler measures the mesh height per coarse cell, with strict rationing: viewer within 15 km only, no sampling if the camera is above 25 km, at most 40 samples per poll, one-shot per cell per session, an undefined result (tiles not streamed yet) is not latched, and a sample is accepted only if it falls between 15 m below and 80 m above a warm terrain prior (a vertical probe can hit a rooftop or another aircraft). Mesh cells answer first, terrain second.
3. **Held floor and ease:** a contact standing on a borrowed neighbour floor keeps its last resolved floor while within about 1 km of where it was measured, probes adjacent cells at most every 500 ms, and a floor that drops under it is approached exponentially (time constant 360 ms, at most 22% of the remaining gap per tick); rises apply immediately because up is the safe direction. A retired floor seed stays reusable for 90 s because the feed's on-ground flag flaps through a take-off rotation and the height source switches at the same moment.
4. **Ground snap for 3D models:** a depth-tested model cannot use the billboard's "ignore depth" trick, so each grounded modelled aircraft gets one successful vertical sample of the tile skin, cached per aircraft, invalidated when it moves more than 50 m, retried with exponential backoff (maximum 30 s), and budgeted per window so a freshly modelled airport cluster cannot fire dozens of picks in one tick. Before the first sample resolves the model is withheld and the billboard stays; after it, a taxiing aircraft holds the last measurement while it resamples. A model's belly offset (distance from model centre to lowest vertex, measured per asset, 1.4 to 7.8 m) is added so wheels sit on the surface.
5. **Corridors:** for ground-moving or coasting aircraft within 25 km of the viewer, the cells along the path two poll intervals ahead (60 s) are pre-warmed, at most 64 new cells per poll and 4 per aircraft in the first pass, because rollout at 27 m/s covers about 810 m per 30 s poll.

Known residual (the product's own note): at a freshly visited airport grounded and low aircraft float low for one to two polls (30 to 60 s) while floors resolve. Cockpit mode waits up to 5 s for the surface under a confirmed ground contact before moving the camera.

---

## 2. Feature table

### 2.1 Civil flights (public layer)

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-01 | Worldwide civil snapshot | Turn on Flights: thousands of aircraft silhouettes; row shows count and source | Section 1.1; one shared worldwide snapshot, no viewport filtering, 30 s client poll, normalised and sticky-merged | OpenSky Network | DOCUMENTED (03 s1) | KEEP; UAE: use a UAE-centred regional feed as primary (OpenSky global is wasteful and rate-limited) |
| AIR-02 | Auth modes for the snapshot | None visible except source label and error text | Four server modes: OAuth client-credentials (default), HTTP basic, auto (OAuth then basic then anonymous), anonymous. Token refreshed one minute early, concurrent refreshes share one request; rejected-credential reasons map to short row messages ("OAuth client missing/invalid", "rate limited (anonymous)") | OpenSky | DOCUMENTED (03 s1) | ADAPT |
| AIR-03 | Credit governor and serve-stale | Layer keeps drawing during rate limits; STALE cue | Adaptive cache lifetime from remaining-credit header, bounded cooldown on 429, stale body served with status headers, synthetic 429 on cold start | OpenSky | DOCUMENTED (03 s1) | KEEP idea |
| AIR-04 | Regional fallback | Source label switches, coverage text names the radius | Section 1.1 item 3; also switched in when the primary snapshot is more than 2 minutes old; if both fail, last snapshot's true age is shown ("Source snapshot N min old", minimum 2) and every contact is marked STALE | adsb.lol | DOCUMENTED (03 s1) | KEEP; this is the natural UAE primary |
| AIR-05 | Render-behind interpolation | Smooth gliding icons | Section 1.3; 30 s display delay, 5 fixes retained | Derived | DOCUMENTED (03 s1 "display 30 s in the past") but mechanism THIN | KEEP |
| AIR-06 | Dead-reckoning, coast and warm-up | No visible stop-start; stale aircraft fade | Section 1.3: backward warm-up extrapolation, constant-rate-turn arc coast (60 s minimum to 5 min cap), 45% alpha stale fade | Derived | THIN: coast limits and 45% fade documented; warm-up backward extrapolation and arc model missing | KEEP |
| AIR-07 | Course blending and slew limit | Noses turn smoothly | Chord/track blend weighted by speed, helicopters track-only, 60 deg/s cap | Derived | MISSING | KEEP |
| AIR-08 | Fleet tick cadence | Invisible | 80 ms fleet tick; rotation on camera change or 1 s idle; tracked per frame with one cached position | Browser | DOCUMENTED (03 s1 cadence) | KEEP |
| AIR-09 | Sticky metadata | Callsign and heading do not flicker to blank or north | Section 1.2 | Derived | THIN (rule stated in 07 only for fields) | KEEP |
| AIR-10 | Removal and landed fast-cull | Aircraft vanish after landing instead of ghosting | 3 missed polls; 1 missed poll when last fix low (below ~150 m) and slow (below ~45 kt); 5 min retention for partial snapshots | Derived | DOCUMENTED (03 s1) | KEEP |
| AIR-11 | Source-backoff stale cue | STALE word in card and cockpit, 20 s retry | A failed poll backs the layer off 20 s (or the server's retry hint); every contact and the cockpit read STALE; cockpit holds its last position | Derived | DOCUMENTED (03 s1, 06 s2.2) | KEEP |
| AIR-12 | Billboard silhouettes and sizes | 10 class glyphs with relative sizes | Section 1.4; glyph set drawn as vector source and rasterised at double density; per-class scale table (glider 0.58 to four-engine 1.45); ground aircraft 0.8 | Derived | DOCUMENTED (03 s1, 05) | KEEP; UAE: add widebody-heavy bias is automatic (Emirates, Etihad) |
| AIR-13 | Limb recession | Far aircraft shrink and fade | Scale to 45% and alpha to 35% between ~3,500 and 4,500 km camera height, composed with the engine's own distance scaling, never culling | Derived | DOCUMENTED (03 s1) | KEEP |
| AIR-14 | Horizon cull | No planes showing through the planet | Explicit occluder test each fleet tick | Derived | DOCUMENTED (03 s0.3) | KEEP |
| AIR-15 | Screen-projected rotation | Nose correct at every camera angle | Section 1.4 | Derived | THIN (mentioned for military only) | KEEP |
| AIR-16 | Tint states | white, amber, red, cyan | Priority: emergency red, tracked cyan, known-military amber, default white; local magenta | Derived | DOCUMENTED (03 s1) | KEEP |
| AIR-17 | Military duplicate suppression | No doubled icons when both layers are on | Shared registry of known-military addresses (add-only, so a dropout never declassifies); when the military layer is active the civil layer drops those addresses (except the one currently tracked); toggling fires an immediate sweep, not a 30 s wait; when the military layer is off the registry self-refreshes every 60 s so those aircraft still draw amber in the civil layer | adsb.lol | DOCUMENTED (03 s2) | KEEP if military layer is kept; UAE: SKIP military |
| AIR-18 | No filters | none | Layer is on/off only; no type, airline, altitude or country filter in the UI (voice/analyst can filter) | n/a | DOCUMENTED (03 s1) | ADAPT: UAE needs at least airline/airport filters |
| AIR-19 | Layer status row | Count, source, fallback/stale words | Source label and coverage come from the server response headers; feed health words from poll outcome | Derived | DOCUMENTED (01, 03) | KEEP |

### 2.2 Military flights

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-20 | Military snapshot | Amber aircraft worldwide | Section 1.1 item 4; 15 s client poll, 12 s server cache, global list (not viewport filtered) | adsb.lol | DOCUMENTED (03 s2) | UAE: SKIP per plan; keep only for internal admin |
| AIR-21 | Cooldown and stale-with-age | Layer stays populated during rate limits | 429 honoured via the upstream's hint (bounded 5 s to 2 min; default 30 s for 429, 15 s for 5xx); cached body served with age; browser treats a relayed 429 as 45 s hold-off | adsb.lol | DOCUMENTED (03 s2) | n/a |
| AIR-22 | Ground aircraft | Grounded military drawn, not skipped | Ground flag is the string "ground" in the barometric altitude field (no separate boolean) | adsb.lol | DOCUMENTED (03 s2) | n/a |
| AIR-23 | Weight-class models | Heavies read as heavies, all amber | Same per-class models as civil; classes with no own asset (airliner, four-engine, glider) use the shared large airliner model; fast jets and unknowns use a dedicated jet model; amber tint must dominate any livery | Bundled | DOCUMENTED (03 s2, 05) | n/a |
| AIR-24 | Military trail backfill | Tracked military aircraft shows up to ~24 h of past track | Fetches the aggregator's per-aircraft trace file (address-keyed, one file per aircraft, undocumented but live, ODbL; no browser CORS so proxied), 60 s cache, 200 entries, 5 MB cap, 12 s timeout; attribution "adsb.lol (ODbL)" required | adsb.lol | DOCUMENTED (03 s2) but trace file naming/format THIN | n/a |
| AIR-25 | "Absence means nothing" notice | Honest copy | Many military aircraft do not broadcast; UI says so | n/a | DOCUMENTED (03 s2) | n/a |

### 2.3 Local ADS-B receivers and the in-browser radio decoder

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-26 | Local ADS-B layer | Magenta aircraft that your own antenna hears; row shows "N heard, X msg/s" or feed counts | Records from two inputs merged by aircraft address, newest position wins; markers expire after ~60 s without any message | User hardware | DOCUMENTED (03 s3, 04) | LATER; UAE: operator-owned receiver network is a strong differentiator for low-altitude Gulf coverage |
| AIR-27 | Browser radio (WebUSB) | Connect a USB RTL-SDR in desktop Chrome/Edge; tune 1090 MHz | Browser talks to the dongle over WebUSB; reads blocks of 32,768 samples; a read that does not finish in 5 s is treated as a stalled transfer and the session is torn down; graceful close bounded at 2 s, then the raw device is closed (twice that bound overall). Tuner at 1,090 MHz, 2.0 megasamples per second, fixed gain 28.0 dB by default (the WebUSB library maps dB to tuner stages differently from the common desktop library: 20.7 dB heard ~1 message/s, 28.0 dB ~9 on the same antenna) | RTL-SDR | DOCUMENTED (04 s Radio, gain) but stall/close timeouts MISSING | LATER |
| AIR-28 | Mode S frame detection | Invisible | Section 4 gap G-01: power per sample, 16-sample preamble with 4 pulses and 6 quiet gaps, contrast and ratio tests, pulse-position bits, CRC check, carry-over buffer | Radio | MISSING | LATER |
| AIR-29 | Message decode and position | Callsign, altitude, speed, track, vertical rate, category, position | Section 4 gap G-01: extended squitter types, CPR even/odd global decode, local decode against reference, surface CPR, speed-plausibility check | Radio | MISSING (03 s3 documents only the speed filter) | LATER |
| AIR-30 | Receiver location reference (LOCATE) | Faster position decoding after granting browser location | A single airborne frame can be decoded without a pair if a reference within half a zone is known; the receiver location is such a reference for airborne frames, never for surface frames (distance unknown from one frame) | Browser location | DOCUMENTED (04) | LATER |
| AIR-31 | Decoder feeds (dump1090/readsb/tar1090, 978 MHz UAT) | Row shows "2 feeds live" etc.; UAT-only aircraft get a thin lighter ring | Server reads each configured aircraft list about once a second (2 s deadline, 2 MiB body cap, 1 s cache, up to 8 feeds); address safety rule (loopback, private ranges, localhost, .local only; name resolved before each read and the connection pinned to validated addresses; redirects refused); feed health live, stale (its own clock older than 10 s), unreachable, invalid | Local decoder | DOCUMENTED (03 s3, local-receivers guide) | LATER |
| AIR-32 | Local motion | No delay, no snap | Section 1.3 last bullet; forward extrapolation, turn rate over 30 s window, coast 10 s minimum to 60 s maximum, 3 km snap, plausibility check (1.5 x speed + 50 kt, 500 m allowance) | Derived | DOCUMENTED (03 s3) | LATER |
| AIR-33 | Local trail (heard history) | Selected local aircraft shows what the receiver actually heard | 10 minutes, 600 fixes; never a click target | Radio | DOCUMENTED (03 s3) | LATER |
| AIR-34 | Local click card | Rich card (hex, callsign, class, airline/type, route, ALT/GS/TRK, V/S, position age, message count, receiver line) | Class from adsbdb type else emitter category; "evidence" flag stops the card presenting the classifier's default as fact; route only for airline-style callsigns | adsbdb | DOCUMENTED (03 s3) | LATER |
| AIR-35 | Tuner modes (FM vs ADS-B) | One tuner shared by FM radio and ADS-B | FM preset 98.5 MHz at 2.048 MS/s; gain remembered separately per mode | RTL-SDR | DOCUMENTED (04) | LATER |

### 2.4 GNSS integrity

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-36 | Navigation-integrity cells | Translucent green/amber/red squares draped on the ground | Section 4 gap G-04 states the algorithm in full; 0.5 degree cells (~55 km), at least 3 aircraft, 30-minute rolling window across visited views, score = 100 x max(0, bad - 1) / total, bands 2% and 10% | adsb.lol point queries | DOCUMENTED (03 s4) | KEEP; UAE: high value, the Gulf is a widely reported interference region (general knowledge, not measured here); keep the "not a detection" wording |
| AIR-37 | Regional sample, not worldwide | Row text and tooltip explain | 250 nm circle around the screen-centre ground point, snapped to whole degrees; server returns only integrity facts (no identities) | adsb.lol | DOCUMENTED (03 s4) | KEEP |
| AIR-38 | Camera-driven refresh | Cells appear after pan or zoom | 600 ms debounce, at least 2.5 s between reads, 60 s poll, at most 24 requests per minute; client retries 30 s doubling to 2 min; server cache 60 s, 64 anchors, 8 in flight, 10 s deadline, 8 MB cap; provider cooldown up to 1 h honoured by waking in bounded chunks | adsb.lol | DOCUMENTED (03 s4) | KEEP |
| AIR-39 | Honest wording | Never "jamming detected" | Primary state is "navigation-integrity anomaly"; "suspected interference" is a second unvalidated reading; per-aircraft thresholds declared the product's own | n/a | DOCUMENTED (03 s4) | KEEP |
| AIR-40 | Legend with live counts | Three bands with cell counts and blurb | See 03 s4 | n/a | DOCUMENTED | KEEP |

### 2.5 Classification, models, icons

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-41 | Type-aware classification | Silhouette and model match the type | Lookup order: ICAO type designator sets (fast jet, drone, helicopter, four-engine, widebody, turboprop, glider, business jet, light) then the "known type, not special" default of airliner; with no type, the live extended category integer (2 light, 3/4/5 airliner, 6 widebody, 7 fast jet, 8 rotorcraft, 9 glider) then the emitter-category letters (A1/A2 light, A3/A4 airliner, A5 widebody, A6 fast jet, A7 rotorcraft, B1 glider); otherwise airliner | adsbdb + feed | THIN: 03/05 name the classes and sizes; the lookup order and fallbacks are not written | KEEP; UAE: add widebody/A380/A388 and Gulf business-jet types to the sets |
| AIR-42 | Class labels | Card says "Widebody", "Fast jet", "Drone" etc. | Readable name per class (10) | Derived | THIN | KEEP |
| AIR-43 | Per-class 2D and 3D scale tables | Sizes | 2D 0.58 to 1.45; 3D clamped 0.75 to 1.45 while models are shared | Derived | DOCUMENTED (03 s1) | KEEP |
| AIR-44 | Bundled models | Realistic aircraft up close | One shared large-airliner model plus six real per-class models (helicopter, light, business jet, drone, widebody, turboprop) and a jet model for the military layer; all re-authored to real-world metres, nose toward local -X (so a 180 degree heading offset applies), origin at the bounding-box centre; per-model belly offset and bounding radius recorded; CC BY 4.0 attributions required | Sketchfab authors | DOCUMENTED (05, 03 s1) | KEEP; UAE: licence-check or commission models |
| AIR-45 | Fleet 3D admission | Models appear for nearby aircraft | Camera below 800 km, frustum-visible only, nearest-first within caps: proximity mode 150 models, add within 150 km, keep to 185 km; all mode 350 models, add 400 km, keep 450 km; hysteresis stops flicker; cockpit cap 60; minimum apparent size 24 px | Derived | DOCUMENTED (03 s1) | KEEP |
| AIR-46 | Model tint and light look | Models read as clean silhouettes | Colour-mix blend at 0.94 with the layer tint so the tint dominates the livery | Derived | DOCUMENTED (05) | KEEP |
| AIR-47 | Style-flip reload | Infrared look changes model material | Fleet models are dropped in batches of 40 per tick (each plane shows its billboard first) to avoid a render stall | Derived | THIN (05 mentions boost, not batching) | ADAPT |
| AIR-48 | Tracked model regime | Selected aircraft turns into a model when you zoom in | Independent of the fleet toggle; enters below 150 km camera altitude, leaves above 172.5 km (15% hysteresis); the billboard stays shown but transparent until the model is ready so camera framing is instant; tracked model rendered at real scale unless it would exceed 200 px (minimum 40 px) | Derived | DOCUMENTED (05 7.3, 06 s2.3) | KEEP |
| AIR-49 | Model load failure bound | Falls back to icon | Per selection: at most 3 failed loads with 1.5 s backoff, then the billboard only until another selection | Derived | DOCUMENTED (03 s1) | KEEP |
| AIR-50 | Easter-egg conversion | Triangle "TR-3B" replaces identity | Cosmetic override per contact, no 3D asset | n/a | DOCUMENTED (01, 03, 06) | SKIP |

### 2.6 Enrichment and route lookups

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-51 | Type and registration by address | Airline, type, tail number on the card | Community database lookup by address; best-effort, silent on failure | adsbdb | DOCUMENTED (03 s1) | ADAPT; UAE: add a UAE fleet table (local registry, A6- tails) |
| AIR-52 | Route by callsign | "DXB -> LHR" line | Scheduled route lookup by callsign, which can return the wrong leg | adsbdb | DOCUMENTED (03 s1) | ADAPT |
| AIR-53 | Route plausibility gate | Wrong-leg routes hidden | Plane within 130 km of an endpoint or within 200 km of the great-circle path; vertical trend anchored to the plane: low and climbing hard, origin within 150 km; low and descending hard, destination within 150 km | Derived | DOCUMENTED (03 s1) | KEEP |
| AIR-54 | Enrichment queue | Invisible | Tracked aircraft jumps the queue; ambient: on-screen aircraft not yet asked this session, nearest-to-camera first, at most 150 enqueues per poll, token bucket ceiling 300 refilling 150 per 5 minutes; 4 in flight, 200 ms apart; each key once per session | adsbdb | DOCUMENTED (03 s1) | KEEP |
| AIR-55 | Server cache | Repeat sessions resolve instantly | Disk-backed, 24 h time-to-live, negative answers cached, one in-flight request per key, 8 s upstream deadline | adsbdb | DOCUMENTED (03 s1) | KEEP |
| AIR-56 | Why enrichment exists | Most aircraft would be generic airliners | The live category field is empty ("no information") for about 94% of aircraft in field data | n/a | DOCUMENTED (03 s1) | KEEP |

### 2.7 Trails and replay

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-57 | Tracked-aircraft trail | Cyan (#00d4ff) civil, amber military, 2.5 px line behind the selected aircraft, uniform 85% alpha, dimmed to 40% where behind or below scenery so it never disappears | Exactly one polyline entity per trail (not a faded per-vertex primitive: the owner rejected tail fading, "show actual tracks"); consecutive points closer than a small distance are merged; depth-fail material draws the occluded part dimmed | Derived | DOCUMENTED (06 s2.4) | KEEP |
| AIR-58 | Trail body vs head | Trail never extends in front of the icon | Because the icon is drawn 30 s behind the newest fix, the body is every accumulated fix except the newest, and a per-frame head segment joins the last body point to the displayed (delayed) icon; head needs at least two accumulated points; on a parked aircraft the head is suppressed so no line runs through the fuselage | Derived | DOCUMENTED (06 s2.4) | KEEP |
| AIR-59 | Seeding at selection | Trail appears instantly | Seeded with the short retained fix history, but only fixes at or behind the delayed display time; newer fixes join as they age past the delay; during warm-up nothing is drawn | Derived | THIN (06 says "seeded with short dead-reckoning history") | KEEP |
| AIR-60 | Civil history backfill | Trail extends back in time | Fire-and-forget request for the aircraft's track from OpenSky's experimental per-aircraft track service (own credit bucket, about 4 credits per call, 60 s server cache, 200 entries); waypoints are time, latitude, longitude, barometric altitude, track, on-ground flag; waypoints strictly older than the oldest seeded fix are spliced ahead of the local segment; cap 400 vertices, newest kept; any failure (not found, rate limit, timeout, malformed) silently leaves the local trail | OpenSky | THIN: 06 documents splice and cap; endpoint class, credit cost and waypoint shape missing | ADAPT |
| AIR-61 | Backfill height handling | Trail does not dive underground | Track altitudes are barometric only, so render height is baro + geoid; a bounded (1.2 s) terrain-floor resolve floors every waypoint at local ground, cells warmed in the background for the next paint; a waypoint with no baro sits on the surface if the floor is known, else holds the previous waypoint's height; leading points with nothing to carry get a 10 km default | Derived | DOCUMENTED (06 s2.4) | KEEP |
| AIR-62 | Military history | Up to ~24 h of trace | See AIR-24 | adsb.lol | DOCUMENTED (03 s2) | n/a |
| AIR-63 | Grounded-aircraft trails | Taxi history shows on select | Ground aircraft draw no trail while untracked ("a taxi path is noise"); a tracked grounded aircraft gets one, and a trail starts when a tracked grounded plane takes off | Derived | DOCUMENTED (06 s2.4) | KEEP |
| AIR-64 | Local-receiver trail | Only what your antenna heard | AIR-33 | Radio | DOCUMENTED | LATER |
| AIR-65 | Trail click ownership | Clicking a trail does nothing | The trail's id namespace is claimed by the pick registry as "owned by another layer", so no layer treats it as empty space | Derived | DOCUMENTED (06 s2.1) | KEEP |
| AIR-66 | Replay / time scrubbing for aircraft | Not present | The product has a retained "report-time playback" engine (48-byte fixes, shared memory budget of 32 MiB, lag reservoir, seek, rate, gap markers, outlier quarantine) but only the transit layers use it; aircraft have trails and an analyst track tool (200 thinned points) and nothing else | Derived | DOCUMENTED as absent only implicitly | LATER; UAE: flight replay for incident review is high value |

### 2.8 Click-to-track, follow camera, share restore

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-67 | Select by click | Click an aircraft, camera starts following | A click counts as a selection when pointer travel stays within 6 px (duration ignored, so a stationary long press can select); a deselecting click on empty space needs both travel within 6 px and press duration within 400 ms; travel is accumulated segment by segment so an orbit nudge that returns to the start pixel cannot pass as a click | Browser | MISSING (06 s2.1 lists what deselects but not the gesture rule) | KEEP |
| AIR-68 | Pick rules | Right thing is selected | Pick registry resolves heterogeneous ids (hex strings, numeric satellite ids, vessel objects); a pick that belongs to a sibling layer is not empty space; clicking the aircraft already tracked, via its 2D entity, its standalone model, or any pick carrying its address, is a no-op; a tool that owns the pointer (drawing) blocks track requests | Browser | DOCUMENTED (06 s2.1) | KEEP |
| AIR-69 | Initial follow framing | Aircraft centred behind-and-above | Any camera flight is cancelled first (otherwise voice-started tracking mid-flight would not frame); camera placed once at a computed offset: range = clamp(altitude x 1.1 + 2,500 m, 3,000 to 30,000 m), 0.8 x range behind and 0.55 x range above in an east-north-up frame; default altitude 1,500 m if unknown | Derived | DOCUMENTED (06 s2.2) | KEEP |
| AIR-70 | Single camera writer and minimum range | No jitter at close range | After first framing the engine's own entity-follow is the only writer (two writers produced sub-pixel jitter); the guard enforces a 150 m minimum range, turns the camera back if a programmatic zoom crosses the target, disables zoom inertia while tracking and raises the zoom controller's minimum distance to 150 m, restoring both on release; framing logic runs before render, not after, because after-render changes made camera and icon disagree for a frame (visible front/back oscillation); the follow frame is fixed to east-north-up because an automatic frame alternated with the model's frame | Derived | THIN: 06 states 150 m and one-writer; zoom-inertia override, pre-render timing and fixed frame missing | KEEP |
| AIR-71 | Shared display position | Label and card do not swing around the icon | The camera, label, rotation and trail head all read one per-frame cached display position; a separate "visual position" (the translation the 3D model really renders with, including ground-snap offset of up to ~100 m inland) anchors the card so it sits on the aircraft you see | Derived | THIN (06 s2.2 states the first half only) | KEEP |
| AIR-72 | Deselect in place | Camera stays where it is | Billboard restored with the last rendered rotation (otherwise it reappears with a stale nose for up to a second), ground/military-aware restore, entity removed, no fly-to | Derived | DOCUMENTED (06 s2.1) | KEEP |
| AIR-73 | Eviction vs deselect | Cockpit card keeps last values when the aircraft leaves the feed | Eviction (aged out) is distinguished from deliberate clear; consumers like the cockpit contact panel hold last-known data after an eviction | Derived | DOCUMENTED (06 s3.3 "contact lost") | KEEP |
| AIR-74 | Cross-layer handoff | Tracking a military aircraft drops the civil track | When another layer takes the follow camera, this layer tears down its state but leaves the viewer's tracked entity alone | Derived | DOCUMENTED (06) | KEEP |
| AIR-75 | Share-link restore | Opening a link follows the same aircraft | Deferred latch retries until the feed shows the target; a known-military address stays restorable by the civil layer because suppression exempts a latched target; explicit user selections cancel a pending restore; recipient gestures cancel it | Derived | DOCUMENTED (06 s2.1, 02) | KEEP |
| AIR-76 | Escape deselect | Escape releases the tracked aircraft | One of the layered Escape owners | Browser | DOCUMENTED (06 s2.1) | KEEP |
| AIR-77 | Context slot | Voice and cockpit read the selected aircraft | The selected contact's live descriptor (not a selection-time snapshot) is published, with the display label chain callsign, registration, address, and an honesty flag when it is coasting | Derived | DOCUMENTED (07, 08) | KEEP |

### 2.9 Contacts roster

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-78 | Contacts window | 250 km picture around the tracked aircraft, vessel or installation | Proximity engine over loaded layers; 750 ms parked, 175 ms while camera moves, back to slow after 250 ms still; re-evaluated when the subject moves 250 m | Layers | DOCUMENTED (06 s3) | ADAPT |
| AIR-79 | What counts as "nearby" | Counts match what is on screen | An aircraft is included only if it is tracked, or its billboard is shown, or its 3D model is genuinely rendering (not merely loaded or hidden); suppressed duplicates and horizon-hidden aircraft are excluded unless a hidden-inclusive query is requested | Derived | MISSING | KEEP |
| AIR-80 | Cohort honesty | "?" for unknown, "at least N" at cap | Feed down means unknown, never zero | Derived | DOCUMENTED (06 s3.2) | KEEP |
| AIR-81 | PREVIOUS / FOCUS / NEXT | Step through nearest contacts | History walk, nearest unvisited, radius doubling up to whole Earth, reset | Derived | DOCUMENTED (06 s3.5) | KEEP |
| AIR-82 | Pages and compass arrows | Three nearest per page, 10 s rotation, rim arrows | Documented | Derived | DOCUMENTED (06 s3.3) | ADAPT |
| AIR-83 | Filters (voice/tool only) | Civil only, military only, helicopters only | Substring match on class, type code or type name | Derived | DOCUMENTED (06 s3.4) | LATER |
| AIR-84 | Detection tie-in | DETECT turns on at tactical preset | Scope is Contacts, not Cockpit: activation forces detection on, cockpit enter/exit and third-person tracking never touch it, manual off holds until next activation, deactivation restores the full pre-state (mode and density) | Derived | DOCUMENTED (06 s3.6) | LATER |

### 2.10 Cockpit mode

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-85 | Entry gate and transaction | COCKPIT button, key C, voice | Multi-step entry (adopt the target, enter, roll back to the previous tracker on any failure form; the explicit attempted selection is undone through the same persistence path so share state does not keep the failed target) | Derived | DOCUMENTED (06 s4.1); rollback THIN | LATER |
| AIR-86 | Eye camera | First-person view | Eye 7 m ahead and 2.6 m above the aircraft reference, -4 degree pitch, heading slewed at most 28 deg/s, advanced inertially every 50 ms from course and speed and pulled toward the displayed position at the larger of 0.75 m/s and 22% of ground speed; stale feed holds exactly | Derived | DOCUMENTED (06 s4.2) | LATER |
| AIR-87 | Ground rules | Camera not buried at the airport | 12 m clearance above sampled surface; confirmed ground contact waits up to 5 s (probe every 500 ms) for the photoreal surface, meta line "acquiring surface" then "surface fallback" | Derived | DOCUMENTED (06 s4.2) | LATER |
| AIR-88 | Own aircraft hidden, traffic as pips | You are not seen; far traffic is dots | Tracked aircraft's own model is not drawn in cockpit (7 m behind its origin the airframe would fill the visor); near band uses the same add/keep radii as 3D models (150/185 km proximity, 400/450 km all) so an aircraft stays a silhouette even when 3D is off, loading or capped; far contacts become a shared 16 px ring-and-centre-dot pip (6 px, tinted civil #DCEEFF-ish cyan-white or military amber) whose single image identity is shared across hundreds of billboards (a fresh texture per billboard stalled entry); entry never destroys models synchronously (browser became unresponsive) | Derived | THIN: 05/06 name silhouettes vs pips; near-band rule and shared-pip reason missing | LATER |
| AIR-89 | Instruments, tapes, readouts | Roll arc, pitch rails, speed/altitude tapes, compass | Documented with tick spacing rules (speed 10/20/25 kt; altitude 100/250/500 ft) | Derived | DOCUMENTED (06 s4.3) | LATER |
| AIR-90 | Estimated flight plan and destination arrow | FROM/TO and DEST arrow | Uses the plausible-route result; bearing relative clamped to +/-120 degrees | adsbdb | DOCUMENTED (06 s4.3) | LATER |
| AIR-91 | Vision chips | Normal/CRT/NVG/FLIR/Anime/Noir/Snow | Cockpit-only stage intensities without touching shader parameters; settles pending crossfades on entry | Derived | DOCUMENTED (06 s4.3) | LATER |
| AIR-92 | Contact panel and briefing carousel | Counts, nearest object arrow, SIG/NEWS/LOCAL pages | Documented | RSS, Open-Meteo | DOCUMENTED (06 s4.4-4.5) | LATER; UAE: Arabic/English local news |
| AIR-93 | Cloud and weather effects | Optional clouds, rain, fog on the visor | Ray-marched noise at ~12 fps, 520 x 320 buffer, 5 min or 25 km refresh, altitude fades | Open-Meteo | DOCUMENTED (06 s4.6) | LATER |
| AIR-94 | Exit rules and grace | Esc, C, button, reset; auto-exit if aircraft vanishes | 1.5 s grace after a contact hop | Derived | DOCUMENTED (06 s4.1) | LATER |
| AIR-95 | Narrow-window layout | Tapes and side windows hide at 760 px or less | Documented | Browser | DOCUMENTED (06 s4.1) | LATER |
| AIR-96 | Camera tools refused | Voice cannot fly the camera in cockpit | Documented by design | Derived | DOCUMENTED (08) | LATER |

### 2.11 Airport and runway ground handling

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-97 | Ground traffic is drawn | Taxiing and parked aircraft visible, clickable, trackable | Same silhouette and rotation pipeline at 0.8 scale, full-strength colour (a muted grey was rejected: unreadable); on-ground flip restyles the same icon in place, so landing and take-off are transitions not removals | Feed flags | DOCUMENTED (03 s1) | KEEP; UAE: DXB/AUH/DWC ground view is a headline use |
| AIR-98 | Four-tier ground floor | Aircraft sit on the apron | Section 1.6 tiers 1 to 5 | Terrain service, rendered mesh | THIN: 03 documents radius, corridors, ease and seed; coarse grid, lift, mesh acceptance window, rationing, ground snap and belly offset missing | KEEP |
| AIR-99 | Geoid correction | Coastal airports at negative ellipsoid height not underwater | Section 1.5 (a US coastal ramp sits near -30 m ellipsoidal) | EGM96 | THIN (07 only for camera altitude) | KEEP; UAE: Gulf fields sit near sea level |
| AIR-100 | Model on ground | Wheels on the tarmac | AIR-45 plus ground snap and belly offset | Derived | THIN | KEEP |
| AIR-101 | Floor flap resilience | No pop below runway through rotation | Section 1.6 tier 3, 90 s seed grace | Derived | DOCUMENTED (03 s1) | KEEP |
| AIR-102 | Runway/taxiway geometry | None | The app draws no runway, taxiway, gate or airport layer of its own; airports exist only as places to search and as route endpoints; there are no airport markers, no runway-aware logic (no landing-queue, no go-around detection) | n/a | DOCUMENTED implicitly | ADAPT; UAE: add airport and runway reference layer |

### 2.12 Emergency detection

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-103 | Squawk rule | 7500, 7600, 7700 turn the aircraft red | Mode A code must be four octal digits; 7500 unlawful interference, 7600 radio failure ("nordo"), 7700 general; international standards | OpenSky squawk | DOCUMENTED (03 s1) | KEEP |
| AIR-104 | ADS-B emergency field | Same red tint | Aggregator reports general, lifeguard, minimum fuel, nordo, unlawful, downed, reserved; superset of squawk codes | adsb.lol | DOCUMENTED (03 s1) | KEEP |
| AIR-105 | Severity and tie-breaking | Priority statuses are not red | Emergency severity: general, unlawful, nordo, downed; priority severity: minimum fuel, lifeguard. Highest severity wins; on a tie the ADS-B field wins; a 7x00 squawk counts even when ADS-B says none | Derived | DOCUMENTED (03 s1, 08) | KEEP |
| AIR-106 | Provenance wording | "broadcasting squawk 7700 (general emergency)" or "ADS-B reports minimum fuel" | Whether the Mode A code itself names the status decides the phrase; never "has declared an emergency" | Derived | DOCUMENTED (03 s1, 08) | KEEP |
| AIR-107 | Latest-broadcast semantics | Status clears when the crew resets the code | Not sticky across polls except one source-switch omission; an explicit "none" or a changed squawk clears immediately; contact time stays the last general contact | Derived | DOCUMENTED (03 s1) | KEEP |
| AIR-108 | Primary vs fallback difference | Primary can only see squawk-based emergencies | OpenSky rows carry squawk but no emergency field; fallback rows carry both; provenance differs but kind agrees | Derived | DOCUMENTED (03 s1) | KEEP |
| AIR-109 | Where it shows | Red icon or model, cockpit meta tag, analyst filter by source | Red also on military layer; no alert list, no sound, no notification, no history of emergencies | Derived | THIN (no "no alerting" statement) | ADAPT; UAE: an alert feed for the operator is a cheap high-value addition |
| AIR-110 | Local receivers and emergencies | Not available | The radio decoder reads extended squitter only (no squawk), and the local card shows no squawk or emergency; decoder-feed records could carry them but the card does not show them | Radio | MISSING | note |

### 2.13 Per-aircraft cards

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-111 | Tracked card (civil and military) | Three lines: callsign - FL - kt (STALE), airline - type, origin -> destination | Presentation model of title, details and accent, rendered by the shared overlay host in a protected lane composited above other cards; anchor is the visual position; fed live from the same record the context slot uses; accent #39d0ff civil | Derived | DOCUMENTED (06 s2.5) | KEEP |
| AIR-112 | Altitude format | Feet below FL180, FLnnn above | Standard transition altitude convention at 18,000 ft | Derived | DOCUMENTED (06 s2.5) | KEEP; UAE: transition altitude is 13,000 ft in the UAE (general aviation knowledge) so FL is shown from there |
| AIR-113 | Label identity chain | Callsign, else registration, else hex, never blank | Whitespace-only callsign counts as blank | Derived | DOCUMENTED (06) | KEEP |
| AIR-114 | Local card | See AIR-34 | | | DOCUMENTED | LATER |
| AIR-115 | Ambient hover text | None | Aircraft show no hover cards | n/a | DOCUMENTED implicitly | n/a |

### 2.14 Detection overlay on aircraft

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-116 | DETECT labels on aircraft | Boxed labels on visible aircraft | Aircraft layers feed the shared detection drawer with frustum and horizon-visible contacts; military weighted highest; labels single-line | Derived | DOCUMENTED (07) | ADAPT |
| AIR-117 | Cockpit subject exclusion | Your own aircraft gets no label | In cockpit, detection omits only the active cockpit subject; a subject present in both civil and military feeds is suppressed in both | Derived | THIN | LATER |
| AIR-118 | Colour/tint priority | Normal, priority, emergency, tracked, cockpit dot | Tint priority table covered by tests | Derived | DOCUMENTED (03) | KEEP |

### 2.15 Analyst, voice and MCP access to flights

| ID | Feature | What the user sees / does | How it works (technique) | Data source | Documented in | Kawn verdict / UAE |
|---|---|---|---|---|---|---|
| AIR-119 | Analyst query over loaded layers | "How many flights over Texas", "anything above 40,000 ft" | Filter/sort/scope over loaded rows (up to 20,000 per aircraft layer); "at least N" when the cap is hit; answers carry feed state words | Layers | DOCUMENTED (08) | KEEP |
| AIR-120 | Row fields | Altitude m, speed m/s, vertical rate, heading, callsign, hex, origin country, operator, class, military flag, ground flag, squawk, emergency kind/source/phrase, last contact, route (civil, only if plausible) | Same record used by the context slot | Layers | DOCUMENTED (03 s1, 08) | KEEP |
| AIR-121 | Emergency filter by source | "Any 7700 squawks?" | Analyst can filter emergencies by provenance | Layers | DOCUMENTED (08 s emergency wording) | KEEP |
| AIR-122 | World-query tools | Aircraft in an area, find by callsign/hex/registration, track, type/route lookup | Aircraft in area (nearest first, military switch), find aircraft anywhere, track (200 thinned points), info (adsbdb type, registration, route with no-copy notice) | Public APIs | DOCUMENTED (08 s tools) | ADAPT |
| AIR-123 | Frame what is overhead | Pull back to show nearby aircraft | Default radius 150 km | Layers | DOCUMENTED (08) | KEEP |
| AIR-124 | Select nearest aircraft near a place | One voice turn: fly, wait, load layer, exclude on-ground, follow | Atomic step | Layers | DOCUMENTED (08) | KEEP |
| AIR-125 | Follow from a shared link | Link carries one aircraft | Cockpit cannot be carried by a link | Share | DOCUMENTED (02, 09) | KEEP |
| AIR-126 | Situation brief and military awareness | Aircraft section in a brief | 250 km military awareness, section by section | Layers | DOCUMENTED (08) | UAE: SKIP military |
| AIR-127 | Identity narration honesty | Voice admits missing enrichment | Test-pinned wording | Derived | DOCUMENTED (08) | KEEP |

---

## 3. Build notes for Kawn (aircraft)

- Primary position source for the UAE should be a regional aggregator query centred on the UAE (the product itself treats this as its resilient path), not the worldwide state-vector snapshot; keep the freshness switch (prefer the regional feed when the worldwide snapshot is more than 2 minutes old) and the "never say just now" rule.
- The 30-second render delay is a product decision tied to a 30-second poll. If Kawn polls a regional feed every 5 to 10 seconds, reduce the delay to about one poll interval; the principle (interpolate between two real fixes) is what matters, and "delay equals one poll" is the rule.
- Keep sticky metadata, the landed fast-cull and the eviction counts exactly: they were each added after a field defect.
- UAE airports are at or near sea level; geoid handling and the ground floor matter more than inland. Plan a bundled geoid grid and a coarse floor cache; photogrammetry offsets are only relevant if a photoreal tileset is used.
- GNSS integrity is the most Gulf-relevant item in this domain; keep wording and thresholds, but disclose that the per-aircraft rule is borrowed from a US avionics minimum and is not validated against the region's events.
- Local ADS-B maps to an operator-owned receiver network (fixed receivers reporting to a server) rather than the browser dongle; the merge, plausibility and staleness rules transfer unchanged.

---

## 4. Gaps found (missing or thin behaviours, written out in full)

**G-01 Radio decoder pipeline (AIR-27 to AIR-30) - MISSING in all notes.**
Input: unsigned 8-bit interleaved I/Q samples at 2.0 megasamples per second from the tuner at 1,090 MHz. Step 1, magnitude: each sample pair is centred (subtract 127.5), squared and summed, giving a power value per sample (two samples per microsecond; one Mode S bit lasts 1 microsecond = 2 samples). Step 2, preamble: a Mode S frame starts with 4 pulses at sample offsets 0, 2, 7 and 9 of a 16-sample preamble; the six gaps between them (offsets 1, 3, 4, 5, 6, 8) establish a local noise floor (offsets 10 to 15 are deliberately not used because the tail of the last pulse leaks into them on real 2 Msps hardware). A candidate is accepted only if pulse-minus-quiet contrast is at least 120 and the pulse mean is at least 1.8 times the quiet mean; the bit decision level is quiet mean + 0.35 x contrast. Step 3, bits: 112 bits follow the preamble, each as two samples; a bit is 1 if the first half-sample is stronger than the second; a bit whose two halves differ by less than max(20, 2% of contrast) is rejected as too weak and the frame is dropped. Step 4, validity: downlink format must be 17 (ADS-B extended squitter) or 18 (non-transponder/ground relay); the 24-bit Mode S parity (polynomial 0xFFF409 expressed as the standard Mode S cyclic redundancy check) must leave a zero remainder; frames that fail are discarded (no error correction is attempted, unlike some desktop decoders). Step 5, stream continuity: enough samples (one full frame minus one) are carried over between USB reads so a frame split across a block boundary is recovered. Step 6, fields: aircraft address from bytes 2 to 4; type code (5 bits) selects content. Types 1 to 4: 8-character callsign in a 6-bit character set, plus emitter category letter from the type code (A from 4, B from 3, C from 2, D from 1) and 3-bit category (A7 rotorcraft, A1 light). Types 5 to 8: surface position with a 7-bit movement field mapped by bands to speeds (nonlinear: 0.125 kt steps near zero up to 5 kt steps near 100 kt; 124 means 175 kt or more), optional ground track in 360/128 degree steps, no altitude. Types 9 to 18: airborne position with barometric altitude (Q-bit set: 25 ft steps minus 1,000 ft). Type 19 subtypes 1 and 2 (ground-speed velocity, with x4 supersonic multiplier): east-west and north-south components give speed and heading; vertical rate in 64 ft/min steps. Not decoded: GNSS-height position types, airspeed velocity subtypes, aircraft status (so no squawk, no emergency), operational status/quality (so no NIC/NACp locally), short Mode S replies. Step 7, position: CPR (compact position reporting) packs latitude and longitude into 17 bits each, alternating "even" and "odd" frames with different zone sizes (15 latitude zones, 131,072 scale). A global decode needs an even and an odd frame within 10 s (25 s on the surface, where movement is slow); a single frame is decoded locally against a reference known to be within half a zone (airborne: the aircraft's own last accepted position if under 10 minutes old, else the receiver location; surface zones are a quarter of the globe so the reference must be within 45 nautical miles, only the aircraft's own recent position qualifies and the receiver location never does; a surface pair has four candidate answers 90 degrees apart and the one nearest the reference wins, no reference means no answer). Airborne and surface frames never pair. Step 8, sanity: every new fix must pass a speed check against the last accepted fix (limit 1.5 x reported speed + 50 kt, 1,000 kt when unknown, 350 kt for light, glider and rotorcraft categories, 500 m allowance); a failing fix is counted and dropped; from the third consecutive rejection a globally decoded fix becomes the new reference so one bad accepted fix cannot freeze a track. A surface message sets on-ground and clears altitude and climb; the next airborne position clears it. Step 9: tracks with no valid message for 60 s are removed. Real-world tuning: gain matters enormously (AIR-27) and an antenna tuned for 1090 MHz matters more than software.

**G-02 Altitude datum chain and ground tiers (AIR-98, 99, 100).** Written in sections 1.5 and 1.6 above. Rebuild rule: never place an aircraft below the local ground; lift by 1.5 m; floors are an estimate, so every tier must fail toward "slightly high" rather than "buried".

**G-03 Smoothing algorithm (AIR-05 to AIR-07).** Written in section 1.3. Rebuild rule: three position regimes (interpolate between fixes, extrapolate backwards at warm-up, extrapolate forwards when stale), one cached position per frame for the tracked aircraft, an absorber for discontinuities, and a rate-limited course.

**G-04 GNSS cell algorithm in one place (AIR-36 to AIR-40).** Per-aircraft: airborne, position from direct ADS-B or multilateration, broadcast version 1 or 2, position age 0 to 60 s, derived quality values ignored. Degraded if accuracy category below 8, or integrity category below 7 (except integrity 0 with accuracy 10 or higher, treated as unknown), or the aggregator's GPS-lost flag is set (evidence then sits at the last good position). Per-cell: key is the 0.5 degree square; an aircraft counts once per cell per 30-minute window; presence and degraded evidence age separately (a degraded report keeps that aircraft degraded for a full window after the report, however many healthy reports follow, and a recovered aircraft is not held degraded forever). Rows are dated by when the proxy observed the snapshot, so replaying a cache never restarts a window and a snapshot older than the window adds nothing. Score = 100 x max(0, degraded - 1) / total. Hide cells with fewer than 3 aircraft. Low under 2%, medium 2 to 10%, high over 10%. Fill alpha 0.30, 0.42, 0.55. Cells are non-pickable so overlapping ground items stay clickable (a ground primitive with picking disabled still erases picks of overlapping geometry, so a render-only fill is used).

**G-05 Classification order (AIR-41, 42).** Written in the AIR-41 row. Add to Kawn: a Gulf-aware type table and an explicit "unknown" class so the card never presents a default as fact (the local card already does this with an evidence flag; the public card does not).

**G-06 Click and deselect gesture (AIR-67).** Written in the AIR-67 row.

**G-07 Tracked camera guard details (AIR-70, 71).** Written in the rows.

**G-08 "Nearby" definition (AIR-79).** An aircraft counts toward nearby counts, the contacts roster and analyst proximity only if it is tracked, drawn as a visible billboard, or really rendering a 3D model; a model that exists but is hidden, unplaced or loading does not count. A rebuild that counts all records will disagree with what the user sees.

**G-09 Feed-state contract between server and browser.** The browser learns the source name, coverage text, count, auth mode and reason, cache status and cache age from response metadata rather than from the body. Fallback responses name themselves (source, coverage such as "250 nm regional fallback", count). Freshness is taken from the source snapshot's own time, not from when the browser received a cached 200; a snapshot older than 2 minutes or without a time is stale, the layer backs off, shows "Source snapshot N min old" (minimum 2) and marks every contact STALE.

**G-10 No emergency alerting and no airport layer (AIR-102, 109).** State this explicitly in the Kawn spec so it is a decision, not an omission.

**G-11 Local-receiver emergencies (AIR-110).** The radio decoder cannot detect emergencies; decoder feeds can supply squawk but the card ignores it. Kawn's receiver network should carry squawk.

**G-12 Playback engine scope (AIR-66).** Playback exists as infrastructure (bounded memory per track, shared budget, seek, rate, gap markers, outlier quarantine) used only by transit. A UAE flight replay would reuse that design with the local ADS-B history as source.

**G-13 Trail backfill mechanics (AIR-60, 24).** Written in the rows (endpoint class, credit cost, waypoint shape, trace files per aircraft).

**G-14 Fleet-wide model style flip batching (AIR-47).** Dropping 350 GPU-backed models synchronously in a style handler stalls the render thread; release 40 per tick, billboard first. Same concern in cockpit entry (never destroy synchronously).

---

## 5. Corrections to existing docs

- **03 section 2 "Landing rule ... below about 500 ft".** The civil rule is below about 150 m (about 500 ft) barometric and 23 m/s; for the military layer the shared record policy uses the same constants (landed limit 1 poll, grace 3 polls, back-off 20 s), so "500 ft" is consistent but the doc implies a separate military rule; they are one rule.
- **03 section 1 "Billboards: 20 px".** Not stated; the billboard base is 20 px (24 px tracked) drawn from a 96-unit source rasterised at about 192 px, so 2D sizes in 03/05 should be read as multipliers of that.
- **06 section 2.2 "the aircraft icon keeps drawing the display position".** Incomplete: the card anchors to the separate visual position (AIR-71), which differs from the display position by the ground-snap offset when a grounded model is shown.
- **06 section 2.1** lists deselect triggers but not the 6 px / 400 ms gesture rule (AIR-67).
- **03 section 3 "Local aircraft ... same class silhouettes".** True, but the class is only as good as the emitter category (types 1 to 4 messages) and lookups; the decoder cannot supply squawk or quality fields (AIR-110, G-01).
- **03 section 1 "Trails: Only the tracked aircraft has a trail".** Correct, but add that a trail is suppressed on a parked contact and that grounded tracked aircraft do get one.
- **06 section 4.2 / 05 near-band language.** The silhouette-versus-pip split in cockpit is driven by the same add/keep radii as 3D model admission (150/185 km and 400/450 km), independent of whether models are enabled (AIR-88).
- **Aircraft replay.** Notes mentioning "replay/playback" in 03 refer to transit; there is no aircraft replay (AIR-66).

---

## 6. UNVERIFIED

- Whether OpenSky's "tracks" service remains available to anonymous or free accounts at the time of Kawn's build; it is labelled experimental by its provider.
- The adsb.lol per-aircraft trace file layout and its 24-hour depth are undocumented upstream; behaviour inferred from the proxy and tests.
- Accuracy of ground-floor mesh sampling at Gulf airports (the tuning figures come from US airports: a 17 m terrain-to-mesh gap, a 21 m grade spread across one field); no UAE measurement exists.
- The UAE transition altitude (13,000 ft) and reported GNSS interference in the Gulf are general knowledge, not checked in this pass.
- Whether airport traffic around Dubai and Abu Dhabi is well covered by community receivers: only a single walkthrough count (about 250 aircraft near Dubai in the fallback) exists.
- Live behaviour of the cockpit on touch devices and the exact pixel layout of tapes were not exercised (carried over from 06).
- Whether cockpit instruments need per-aircraft calibration (eye offset is fixed at 7 m / 2.6 m for every class).
- Frame-time cost of the 12 Hz fleet tick with the full 11,000-aircraft snapshot (a performance note in the product's docs claims acceptable cost; not measured).

---

## 7. Counts

Features listed: 127 (AIR-01 to AIR-127). THIN: 16 rows (plus partially thin DOCUMENTED rows noted in the text). MISSING: 6 rows (AIR-07, 28, 29, 67, 79, 110), plus 14 written-out gaps G-01 to G-14 in section 4.

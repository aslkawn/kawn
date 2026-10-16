# Product Spec 02: Data Layers (behavioural study)

Scope: what each live or bundled data layer in the studied globe app does, in product terms. Written for a re-implementation targeting UAE cities. Numbers are approximate and rounded; treat them as sizing hints, not requirements. Detail: reverse-engineering notes 03 (movement and events), 04 (infrastructure, cameras, weather), 06 (contacts).

## Cross-cutting behaviour (applies to almost every layer)

- **Layer panel.** One list in six fixed groups: Movement, Cameras, Infrastructure, Events, Weather, Utilities. Every layer is off by default; a first-run mission picker enables sets only after an explicit choice. State is saved locally and carried in share links.
- **Health words.** An enabled row shows ON, LOADING, DEGRADED, STALE, PARTIAL, FALLBACK or UNAVAILABLE plus a one-line reason. An error with prior data reads DEGRADED (old data stays drawn), with none UNAVAILABLE; a declared fallback source reads FALLBACK; guidance ("zoom in") reads as normal ON; a missing free key reads UNAVAILABLE with a key hint (Kawn keeps keys server-side). A bundled dataset that fails to load is UNAVAILABLE, never a green ON over an empty globe.
- **Altitude gating.** Dense layers wake only below a layer-specific camera height and idle above it, with hysteresis so a slow zoom does not flicker.
- **Debounced, viewport-driven fetching.** Wait for the camera to settle (a few hundred ms), request a padded window, skip if the view mostly overlaps the previous one.
- **Hard caps.** Every layer has a ceiling on drawn items; when hit, the UI says coverage is limited. Bundled sets show fewer items when zoomed out.
- **Staleness honesty.** A dead feed must never look alive. Old items are removed or visibly faded, not frozen; freshness uses the source's own timestamp; counts name their scope; a missing broadcast or map entry is never evidence of absence.
- **Declutter.** Labels and cards are chosen greedily by priority and screen spacing (about 150 px) with a per-layer cap; the selected item always gets a protected card.
- **Ground/sea anchoring.** Items sit slightly above terrain or sea (vessels 3 m, bikeshare 2 m) via a height lookup warmed only at close range; at national zoom everything rides the plain ellipsoid.
- **Attribution.** Each layer registers its credit into one expandable panel, visible in clean-view and recording modes.
- **Select / track.** Click selects and opens a card. Only aircraft, military aircraft and satellites are camera-tracked (standoff follow, one tracked subject at a time). Vessels, transit vehicles, local ADS-B aircraft, launches, fires and bikeshare stations are selectable but the camera does not follow them. Escape or empty-globe click clears.
- **Smooth motion.** Moving layers render slightly in the past, interpolating between two real fixes, then coast briefly, go stale and are removed. Local receiver feeds are the exception (no delay).

## Moving-object layers

### Aircraft (civil flights)
- **Purpose:** worldwide live air picture.
- **User sees:** ten class-shaped silhouettes rotated to heading: white by default, amber if a known military identity, red if the transponder broadcasts an emergency, cyan when tracked. Far icons near the globe limb shrink and fade. Grounded aircraft are smaller, no trail; landing and take-off restyle the same icon. Below 800 km camera height on-screen aircraft become 3D models (default on; nearby mode up to 150, all mode up to 350).
- **Interactions:** click selects and tracks; only the tracked plane has a trail (up to 400 points). Card has three lines: callsign, altitude, speed and a STALE cue; airline and type; origin to destination only when judged plausible. Click-to-enrich fetches type, registration, airline and route; ambient enrichment trickles at low concurrency. No type or airline filter.
- **Data:** OpenSky worldwide snapshot is primary; credentials are optional and OAuth is the default mode (anonymous only when none are set, rate-limited). adsb.lol point queries (250 nautical-mile circle) are the bounded fallback, also used when the snapshot is over 2 minutes old. Client polls every 30 s, displays about one poll behind; server cache stretches as the daily credit budget thins. Emergency detection on the primary feed is squawk-only. Metadata from adsbdb.
- **LOD:** icons at altitude, models only close and on screen; ground-floor lookups only below about 4.5 km.
- **Edge cases:** removed after 3 missed polls (1 if low and slow); a failed poll backs off 20 s and marks contacts STALE; failed model loads retry 3 times then fall back to the icon; an old snapshot reports its true age. Most ambient aircraft start as a generic airliner until enriched.
- **Licence:** OpenSky is non-commercial; adsb.lol is ODbL; route data cannot be republished.

### Military aircraft
- **Purpose:** same, filtered to military hex ranges/flags.
- **Sees:** amber silhouettes, same ground and 3D treatment; tracked trail amber, backfilled up to about 24 hours.
- **Interactions:** select, track; same card but never a route line.
- **Data:** adsb.lol military endpoint, polled every 15 s, hundreds worldwide. The civil layer reads it in the background so known military hexes render amber there, and suppresses them while this layer is on.
- **Edge:** many militaries do not broadcast, so absence is meaningless. Kawn: skip.

### Local ADS-B receiver feeds
- **Purpose:** show aircraft from a user's own receiver(s): browser USB radio (desktop Chrome/Edge) and/or decoder feeds, including the 978 MHz UAT band.
- **Sees:** magenta glyphs distinct from the global layer, a status line naming each feed and its health, a heard-position trail for the selected aircraft. Never camera-followed.
- **Data:** server-configured feeds only (up to 8, private-network addresses only), read about once a second, scene merged every ~200 ms.
- **Edge:** positions over about 60 s dropped; feeds merge by aircraft id; an implausible-jump filter rejects speed-impossible fixes; one feed down does not hide the others. Kawn: operator-only admin feature at most.

### GNSS integrity
- **Purpose:** reveal areas of possible GPS jamming/spoofing from aircraft-reported navigation accuracy. Presented as statistics, never a located emitter.
- **Sees:** 0.5 degree ground cells draped on terrain, non-blocking. Low cells (under 2 percent degraded) are drawn on purpose so "covered but clean" differs from "no data"; medium (2 to 10) and high (over 10) are anomalies. Legend with live counts and a caveat. No click card.
- **Data:** adsb.lol point queries (250 nautical-mile circle) driven by the camera plus a 60 s poll, server-cached about a minute; evidence accumulates over a rolling 30 minutes; a cell needs at least 3 aircraft.
- **Edge:** sparse airspace yields no cells, which must read as "insufficient data", not "clean". Very relevant to the Gulf.

### Vessels (AIS)
- **Purpose:** live ship picture.
- **Sees:** chevrons rotated to heading and **coloured by ship type**: tankers amber, cargo/container cyan (default), passenger/ferry pink, fishing green, tug/service pale yellow. Up to about 12,000 drawn, about 900 labels. Selected vessel gets a trail (up to 400 points, only after about 25 m of movement) above the sea surface.
- **Interactions:** select (not camera-followed), view-area query when below about 450 km height. Ambient card: name, type, speed, heading; selected card adds destination, MMSI, last-position time and a STALE mark if it vanishes from refreshes.
- **Data:** AISStream websocket held by the server, browser polls its cache every 60 s; free server-side key. Short server history backfills trails.
- **Edge:** a watchdog classifies the feed (connecting, live, stale, reconnecting, down, auth-failed, missing-key, idle) and the chip shows the reason even with cached vessels on screen; 30 s first-position grace; retained rows never make a dead feed look healthy. Terrestrial AIS is sparse in the Gulf; satellite AIS is usually paid.

### Satellites
- **Purpose:** orbital objects overhead.
- **Sees:** about 840 points in six groups (stations, bright visual, GPS, GLONASS, Galileo, geostationary) in five visual classes with a legend; the ISS is a larger red labelled dot with its path shown by default. A DENSE chip adds about 10,000 unlabelled Starlink points.
- **Interactions:** click shows the orbit path and tracks, standoff scaled up for high orbits; card has class and altitude/catalogue lines.
- **Pass prediction has no on-globe UI.** It exists only through voice and assistant tools (next pass over a place for the ISS or any satellite, minimum elevation, visible-only option; "visible" means any part of the pass sunlit while the observer's Sun is below minus 6 degrees; a sibling lists satellites overhead). Kawn could add a real panel.
- **Data:** CelesTrak element sets via proxy (cached about 6 hours, client refresh 5 minutes), propagated locally every second.
- **Edge:** a satellite in several groups keeps the most specific tag; partial group failure reads DEGRADED, total outage keeps last-good points but UNAVAILABLE.

### Launches / space missions
- **Purpose:** past launches from the **trailing 30 days only** (no upcoming launches), plus replay. Row title carries a 30-day suffix.
- **Sees:** a pad dot per mission coloured by operator group, a roster (newest first), and for a selected mission a transfer/orbit line, stage re-entry annotations, live or estimated payload readout and a detail panel. Replay: countdown, compressed ascent, one orbital lap, speed 0.25x to 4x.
- **Data:** Launch Library 2 (at most 100 fetched; pad-less launches dropped), client refresh ~5 minutes, server cache 15 minutes. Payload elements from CelesTrak.
- **Edge:** elements can lag launch, so one delayed retry; ascent without upstream samples is labelled a reconstructed estimate. Enabling enters a Space Missions mode that isolates other layers and enables satellites.

### Transit (buses, trains, trams)
- **Purpose:** live public-transport vehicles.
- **Sees:** mode-coloured icons, played back smoothly behind real time between two real fixes (delay adapts 25 to 120 s); trail for the selected vehicle only.
- **Interactions:** select for a vehicle card. **There is no filter UI** (no feed, mode or route filter, single toggle); a route filter exists only in the assistant lookup.
- **Data:** GTFS-Realtime per agency, polled every ~15 s, server-side registered list; active for feeds near the camera and below ~3,000 km; cap ~15,000. Seven keyless regions (Boston, Austin, Minneapolis, Netherlands, Norway, South East Queensland, Helsinki); none in the UAE.
- **Edge:** dropped after two missed polls or ~10 minutes; feed stale after 90 s, evicted after 5 min; fixes implying an impossible speed for the mode are refused.

### Bikeshare
- **Sees:** points sized by capacity (about 4 to 12 px), coloured by bikes share (green over 60 percent, amber, red under 30, grey no data, dark offline); station card with bikes/docks.
- **Data:** GBFS feeds (32 systems, all US), status polled each minute; active below ~50 km; cap ~8,000. No known UAE feed.

### Street Traffic
- **Purpose:** road activity and congestion.
- **Sees:** small moving dots along roads, sized and paced by road class. **Dots are a simulation, not real vehicles**; with live flow, real speeds drive pace, density and colour (green/amber/red, directional); roads without flow stay white.
- **Interactions:** road-source chips (TomTom, OSM, Hybrid); keyless default is OSM simulation. No per-dot selection.
- **Data:** open vector road tiles plus optional TomTom flow (bring-your-own key, server-side with a daily tile budget); fetched on camera settle, only below ~8 km, major roads only above ~4.5 km.
- **Performance:** dot cap ~6,000 scaled down with height; long polylines sub-sampled; fetch skipped unless the view moved.
- **Edge:** UI must say dots are not real vehicles; attribution only in live mode.

## Hazard and environment layers

### Earthquakes
- **Sees:** **static** ground-clamped translucent discs (not pulsing), radius 2 to the power of magnitude in km, coloured by depth (under 70 km red, 70 to 300 orange, deeper yellow), with magnitude labels (cap 96). Row title carries a 24-hour suffix.
- **Interactions:** **none, no click or card.** Details only in assistant records. Kawn should add a card if used.
- **Data:** USGS past-day feed, browser-direct, magnitude 2.5+, refreshed every 60 s; tens to a few hundred events.
- **Edge:** a malformed snapshot is rejected and the last good kept (DEGRADED). Low UAE relevance.

### Fire detections (satellite thermal)
- **Sees:** far away, aggregated heat cells; nearer, glow sprites sized and coloured by radiative power; capped nearest-hemisphere cards; click selects and the camera dives to it.
- **Data:** NASA FIRMS via proxy (three VIIRS satellites plus MODIS, trailing 24 hours); client poll ~10 minutes, proxy cache ~30 minutes; free server-side key.
- **LOD:** four altitude bands with 10 percent hysteresis; view padded 30 percent.
- **Edge:** no key reads KEY REQUIRED with setup guidance; detections can be industrial flares, which in the Gulf means many persistent false "fires".

### Fire perimeters
- **Sees:** incident polygons coloured by containment, a card with facts and a verified link to the official incident page.
- **Data:** US interagency feeds only. Skip.

### Tropical cyclones
- **Sees:** storm position, forecast track, lead labels (within ~4,000 km), uncertainty cone; first storm auto-selected.
- **Data:** NOAA advisories, Atlantic and eastern/central Pacific only, five-minute cache. Arabian Sea storms never appear and the UI says so; low UAE value unless another source is added.
- **Edge:** track and cone render only when they match the status advisory; "no active systems" differs from "unavailable".

### Wind
- **Sees:** animated GPU streamlines, optional colour field (speed, pressure, temperature), model chips (GFS, ECMWF), unit chips, inspect-at-map-centre reading.
- **Data:** NOAA GFS or ECMWF IFS open data (10 m wind), partial byte-range reads of large GRIB files, **resampled to a roughly 1 degree display grid** (source 0.25 degree), cached an hour. The animation shows one fixed forecast and does not follow the observed-history timeline.
- **LOD:** curves fade below 60 km camera height, vanish at 15 km; reduced-motion shows a static flow.
- **Edge:** newest run unpublished falls back to the previous; failure names the source. A 1 degree grid is too coarse for a city; Kawn needs a finer regional model (shamal, dust).

### Weather imagery (radar, cloud, lightning)
- **Sees:** time-aware overlays: rain radar (contiguous US only), satellite clouds (regional North America by default, or a global hourly mosaic with 2 to 3 hour latency), lightning density (Americas and Pacific only). Opacity is two chips (Soft, Vivid); clouds have a Clouds-only (default) or Full treatment; each product has a coverage badge.
- **Timeline:** one shared observed clock: each product shows its nearest frame at or before the requested time, never a future or interpolated one, with an eligibility gap (30 minutes; 3 hours for global infrared); share links open at latest.
- **Data:** NOAA nowCOAST WMS, keyless; metadata refreshed every 2 to 10 minutes; exact-time images immutable for 24 hours.
- **Edge:** outside coverage shows nothing and says so. **Only the global infrared mosaic applies to the UAE.**

### Recent imagery
- **Sees:** a 30-day strip of true-colour days: 30 m Sentinel-2/Landsat (default) or 250 m VIIRS overview; thumbnails with cloud cover; a drawn box (up to 1,000 km a side) or the current view; image, versus-basemap and A/B swipe modes; PNG export; a START HERE ranking.
- **Data:** NASA GIBS tiles and CMR catalogue, keyless; daily cadence. Useful for UAE construction and coastline change.
- **Edge:** cloud-free dates are rare in some regions; empty catalogue shows "no scene"; empty days are never draped.

## Infrastructure and context layers

### Submarine cables
- **Sees:** cable routes in dataset colours, landing points with stem markers, labels in the shared overlay (cap ~160 nearest, within ~9,000 km).
- **Interactions:** **click only flies the camera**; no card or selection.
- **Data:** bundled static snapshot: 712 cables, 1,917 landing points (about 2,600 references), each with only id, name and colour: **no owner, length, capacity or service year**. Non-commercial licence: replace or license for commercial use.
- **Perf:** label sweep only on camera settle plus a slow motion probe; disabling removes the sources.
- **Relevance:** UAE landings include Fujairah, Dubai, Abu Dhabi, Sharjah, Kalba, Das Island. Kawn should add a real detail card with licensed data.

### Data centres
- **Sees:** cyan footprints with a stem and label (name first, else operator); stem budget by camera height.
- **Interactions:** click selects and flies in; **no detail panel**, only a title plus one operator/capacity line.
- **Data:** bundled OSM-derived extract, 4,351 features worldwide, static, ODbL. **The UAE has 16**, with noisy tags (several unnamed, one mis-tagged typing centre); a curated local dataset would be better.

### Dams
- **Sees:** blue footprints and labels, same rules as data centres; one river line.
- **Data:** bundled OSM-derived extract, 704 dams. **The UAE has 0 dams**, so the layer is empty there. The studied row mislabels its source; do not copy.

### Installations (military and critical sites)
- **Sees:** named markers coloured by class, up to ~700; touching parcels merge into one area marker; click gives name, class, member names; a SEARCH NEARBY SITES button runs a one-shot places search.
- **Data:** open vector-tile military land-use polygons, names from a separately licensed pack, optional places provider. Views too wide for 16 tiles show "zoom in"; views over ~10 degrees use a bundled named-point index.
- **Edge:** incomplete by nature, and labelled so. With a Contacts subject it loads a 100 km window. Kawn: skip.

### Awareness / contacts
- **Purpose:** a situational picture around a subject (tracked aircraft, selected vessel or site), in the Context panel (a mode beside Space Missions, not a data layer).
- **Sees:** a 250 km window with per-cohort counts (civil, military, vessels, installations), 3 nearest entries per page rotating every 10 s, a compass ring with up to three bearing arrows, PREVIOUS / FOCUS / NEXT.
- **Interactions:** NEXT walks history, then the nearest unvisited contact, widening the radius when all are visited; military-only and class filters arrive only by voice/tools. Entering turns on detection and dependency layers; exiting restores the prior set. Detail: note 06 section 3.
- **Data:** composed from other layers; re-evaluated every 750 ms parked, 175 ms while the camera moves.
- **Edge:** a cohort with a down feed shows "?" not zero; "at least N" when capped; losing the subject keeps last-known data with a "contact lost" cue; the panel says it is not an all-clear.

### Radio
- **Purpose:** geolocated internet radio stations to tune in.
- **Sees:** markers coloured by category (news, talk, weather/emergency, safety, aviation/marine, traffic, music, other), clustered by category and zoom with strict cohort caps (~64 labels), horizon culling.
- **Interactions:** select to listen; an analog tuner snapping across up to 750 stations with simulated static between; transport, volume, tag filter; enabling never autoplays. A local RTL-SDR card (FM and ADS-B) sits in the same panel.
- **Data:** Radio Browser directory: nine bounded queries (one broad popularity plus eight specialist tag queries) merged into 750 working HTTPS stations, cached 45 minutes, last good served up to 7 days. The browser connects to broadcasters directly; audio is never relayed or recorded.
- **Edge:** streams not playing within ~12 s give up; bad-coordinate stations dropped. **UAE coverage is not guaranteed** (worldwide popularity cut); Kawn should curate a UAE list.

### CCTV
- **Purpose:** public traffic and weather cameras with live frames or HLS video.
- **Sees:** camera icons, a coverage pyramid (default on) and optional viewshed, and the live frame projected onto a monitor plane in the 3D scene (1080p, default on) like a hologram; small thumbnail cards for nearby stills.
- **Interactions:** select, nearest/previous/next, focus. **Auto-hop is off by default; interval 18 s** (8 to 90 s, settable only by voice/tool; deselecting suspends it). A calibration gizmo (seven offsets, saved locally) corrects estimated poses.
- **Data:** per-agency public packs (about 3,900 cameras claimed; availability decides): US, London, Canada, Finland, Norway, Estonia, Australia, one German webcam. Frames ~10 s active, 60 s idle; health ~7 s. If no source is reachable an 18-camera seed set is used, **including two hand-set Dubai cameras**.
- **Perf:** geometry in tiny nearest-first batches; unchanged frames detected by a tiny signature.
- **Edge:** frame fallback is upstream picture, then a street-view still, then a placeholder; cameras with uncertain heading are drawn with reduced confidence.
- **UAE note:** no public catalogues; see ranking. Skip.

### ALPR cameras
- **Sees:** community-mapped plate-reader locations as badges with an optional illustrative facing wedge; explicit note these are mapped locations, not footage or events; a SHOW NEAREST chip.
- **Data:** hourly OpenStreetMap extract, US and Canada only; city-scale (view up to 3 degrees), up to ~1,500 records, view snapped to a coarse grid.
- **UAE note:** skip entirely.

### Street level
- **Sees:** a panel with provider chips, **one provider lit at a time** (Mapillary on by default, Google Street View, KartaView); coverage lines for the two crowdsourced providers, nearest-image lookup on click, an embedded viewer, a position marker/cone on the globe.
- **Filters:** IMAGERY (all/360/flat) and SINCE apply to Mapillary and KartaView; Google panoramas are all 360 with no coverage lines, so filters and legend hide when Google is lit. Google appears only on the Google 3D base map, by licence, billed per panorama.
- **Data:** Mapillary needs a free token, KartaView is keyless; coverage tiles cached up to a day.
- **Edge:** missing or rejected keys, outages or no coverage give explicit messages; credits always visible.

### Directions
- **Sees:** a street-following route as flowing dashes draped on terrain, a dot per manoeuvre, a fly-along camera, a turn-by-turn list (cap 200 steps).
- **Interactions:** SET A then B by clicking the globe; DRIVE (default), WALK, BIKE; SWAP, CLEAR, FLY; click a dot for the instruction.
- **Data:** public OSRM community servers via a rate-limited proxy, not for production load; Kawn needs a self-hosted router (scale, Arabic names).
- **Edge:** if no route, it says so; it never draws a straight line as a fake route.

### Regional extras (not layers, but shared)
Boundaries, a single bundled neighbourhood set (San Francisco only), place search, a cockpit-only opt-in local weather readout (Open-Meteo) and regional news are context, not feeds. A bundled non-commercial Nepal flood case study (imagery pair, river line, sixteen evidence records, a schematic corridor rather than a model) demonstrates an "event reconstruction" and data-pack pattern: reuse the pattern for "city stories", not the data. Detail: note 04 sections 14 and 15.

## Relevance ranking for UAE (Dubai, Abu Dhabi, Sharjah)

| Layer | UAE relevance | Data availability in UAE | Privacy / legal stance |
|---|---|---|---|
| Aircraft | Very high (DXB, DWC, AUH, SHJ airspace) | Good, global feeds cover it | Public broadcast; fine; primary feed is non-commercial |
| Vessels | Very high (Jebel Ali, Khalifa Port, Fujairah anchorage) | Fair; terrestrial AIS sparse, satellite gaps | Public broadcast; fine; avoid military names |
| Street traffic | High | Needs provider key; synthetic fallback | Fine if flagged as simulated |
| Directions | High | Works with global map data | Fine; self-host for Arabic names |
| Weather: wind | High | Global model works; grid too coarse for a city | Fine; add a regional model |
| Weather: cloud / dust imagery | High (dust, fog) | Global infrared only; no regional radar | Fine; add local sources later |
| Recent imagery | High | Global | Fine |
| Satellites | Medium | Global | Fine; add a pass-prediction panel |
| GNSS integrity | High (regional jamming) | Derived from aircraft | Sensitive framing; present as statistics |
| Submarine cables | Medium | Static bundle, no owner data | Licence limits commercial use |
| Data centres | Medium | 16 noisy OSM entries | Fine; use a curated dataset |
| Fire detections | Medium (flares, desert burns) | Global | Fine; explain industrial false hits |
| Transit | High potential | None in current feeds | Needs RTA/ITC operator data agreement |
| Bikeshare | Low-medium | Need local operator feed | Fine with operator consent |
| Radio | Low-medium | UAE coverage not guaranteed | Fine; curate a UAE list |
| Street level | Medium | Mapillary partial (unverified); Google billed, map-restricted | Faces/plates blurred by providers; licence |
| Local ADS-B | Medium | Own receiver | Fine; operator-only |
| Earthquakes | Low | Global | Fine |
| Cyclones | Low | Basin not covered | Fine |
| Launches | Low | Global (past 30 days only) | Fine |
| Military aircraft | Low-medium | Partial | Skip or restrict: regional security sensitivity |
| Installations (military) | Skip for UAE | Would map defence sites | Skip: security and legal risk |
| Awareness / contacts | Medium | Composed | Skip military contacts |
| Rain radar / lightning | Skip | US / Americas only | Not applicable |
| Dams | Skip | 0 in the UAE | Not applicable |
| Fire perimeters | Skip | US only | Not applicable |
| CCTV | Skip | No public feeds (two Dubai seed cameras only) | Skip: cameras are government-controlled, surveillance law and the personal data protection law apply |
| ALPR | Skip | None | Skip: plate surveillance, privacy and legal risk |

Summary: build first aircraft, vessels, wind, cloud, imagery, directions, synthetic or licensed traffic, then transit and GNSS statistics with local data agreements. Never ship CCTV, plate readers or military site mapping for UAE.

## Spec revision log

- Vessels are coloured by ship type (not one teal family); vessels, transit, bikeshare, fires and launches are select-only, with only aircraft, military and satellites camera-tracked (note 03 sections 0.3, 5).
- Earthquakes are static discs with no click card; transit has no filter UI; satellite pass prediction is voice/tool only; launches cover the trailing 30 days only; aircraft auth defaults to OAuth (note 03 sections 1, 6, 7, 8, 11).
- Added aircraft/GNSS/local-receiver specifics (3D thresholds, fallback rules, score bands, feed limits) and fire, wind (1 degree display grid), cyclone and street-traffic details (note 03 sections 1 to 4, 10, 12).
- Cables: click only flies, data has no owner/length; added data centres (16 UAE entries) and dams (0 in the UAE); installations wide-view behaviour (note 04 sections 1 to 4).
- CCTV auto-hop is off by default at 18 s, with Dubai seed cameras noted; radio UAE coverage not guaranteed; street level is three providers, one lit at a time, with per-provider filters (note 04 sections 6, 8, 9).
- Weather: opacity chips, regional-by-default clouds, shared observed timeline, coverage limits; recent imagery modes (note 04 sections 7, 12).
- Contacts section filled from the roster study (note 06 section 3); ranking table adds data centres, dams, radar/lightning.

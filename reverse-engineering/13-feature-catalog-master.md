# 13 - Master feature catalogue (merged from censuses 13a-13f)

662 features, one row each: ID, feature, documentation status in our notes, Kawn verdict, source census. Details, mechanisms and gap write-ups live in the census file named in the last column.

Note: 662 of about 687 census rows were machine-merged; the rest (irregular layouts) live only in their census file; 19 rows show "?" where no verdict marker was parseable.

Status: DOCUMENTED = covered by docs 01-12/specs; THIN = mentioned but missing detail (see census gaps); MISSING = not in our docs before the census.

Totals: {'DOCUMENTED': 541, 'THIN': 74, 'MISSING': 47}. Verdicts: {'KEEP': 421, 'LATER': 97, 'ADAPT': 80, 'SKIP': 45, '?': 19}.

| ID | Feature | Status | Kawn | Census |
|---|---|---|---|---|
| NAV-01 | Free camera: drag, wheel, right-drag, middle-drag | DOCUMENTED | KEEP | 13a |
| NAV-02 | Trackpad pinch relay | THIN | KEEP | 13a |
| NAV-03 | Zoom limits | DOCUMENTED | KEEP | 13a |
| NAV-04 | Keyboard zoom | THIN | LATER | 13a |
| NAV-05 | Voice relative zoom | MISSING | KEEP | 13a |
| NAV-06 | Voice fly-to | THIN | KEEP | 13a |
| NAV-07 | Voice return to globe | DOCUMENTED | KEEP | 13a |
| NAV-08 | Search box (tray) | DOCUMENTED | KEEP | 13a |
| NAV-09 | Search chain orchestrator | THIN | KEEP | 13a |
| NAV-10 | Coordinate search | THIN | KEEP | 13a |
| NAV-11 | Bundled-name search | DOCUMENTED | KEEP | 13a |
| NAV-12 | Alps shortcut | THIN | ADAPT | 13a |
| NAV-13 | Google geocoding | DOCUMENTED | ADAPT | 13a |
| NAV-14 | Photon geocoding | THIN | KEEP | 13a |
| NAV-15 | Nominatim proxy | DOCUMENTED | KEEP | 13a |
| NAV-16 | Nearby-place recovery | THIN | ADAPT | 13a |
| NAV-17 | Framing-mode classification | THIN | KEEP | 13a |
| NAV-18 | Viewport sanity gate | MISSING | KEEP | 13a |
| NAV-19 | Natural-region swath | THIN | KEEP | 13a |
| NAV-20 | Building-footprint framing | THIN | KEEP | 13a |
| NAV-21 | Curated landmark reuse | DOCUMENTED | KEEP | 13a |
| NAV-22 | Ground-height resolution for arrival | DOCUMENTED | KEEP | 13a |
| NAV-23 | Post-arrival ground guard | DOCUMENTED | KEEP | 13a |
| NAV-24 | Location tray | DOCUMENTED | KEEP | 13a |
| NAV-25 | City pill | DOCUMENTED | KEEP | 13a |
| NAV-26 | Landmark pill and Q/W/E/R/T | DOCUMENTED | KEEP | 13a |
| NAV-27 | Inter-city world jump | DOCUMENTED | KEEP | 13a |
| NAV-28 | Orbit (key O) | DOCUMENTED | KEEP | 13a |
| NAV-29 | Camera verbs (voice) | DOCUMENTED | KEEP | 13a |
| NAV-30 | World-click focus | DOCUMENTED | KEEP | 13a |
| NAV-31 | Navigation authority | DOCUMENTED | KEEP | 13a |
| NAV-32 | Reset globe | DOCUMENTED | KEEP | 13a |
| NAV-33 | Tilt toggle | DOCUMENTED | KEEP | 13a |
| NAV-34 | North-up compass | DOCUMENTED | KEEP | 13a |
| NAV-35 | Startup camera | DOCUMENTED | ADAPT | 13a |
| NAV-36 | Share-link restore | DOCUMENTED | KEEP | 13a |
| NAV-37 | Share-link writing | THIN | KEEP | 13a |
| NAV-38 | Copy link | DOCUMENTED | KEEP | 13a |
| NAV-39 | Ground-floor service | DOCUMENTED | KEEP | 13a |
| NAV-40 | Terrain on 2D stacks | DOCUMENTED | KEEP | 13a |
| NAV-41 | Terrain tile retry | DOCUMENTED | KEEP | 13a |
| NAV-42 | Map-source chips | DOCUMENTED | KEEP | 13a |
| NAV-43 | Startup tile route | DOCUMENTED | KEEP | 13a |
| NAV-44 | Google 3D tile tuning | DOCUMENTED | KEEP | 13a |
| NAV-45 | Google token source | DOCUMENTED | KEEP | 13a |
| NAV-46 | ion route | DOCUMENTED | ADAPT | 13a |
| NAV-47 | Esri World Imagery | DOCUMENTED | KEEP | 13a |
| NAV-48 | OSM tiles | DOCUMENTED | KEEP | 13a |
| NAV-49 | Bing Aerial / Labels via ion | DOCUMENTED | LATER | 13a |
| NAV-50 | Map controller | DOCUMENTED | KEEP | 13a |
| NAV-51 | Stack-changed broadcast | DOCUMENTED | KEEP | 13a |
| NAV-52 | Voice map switching | DOCUMENTED | KEEP | 13a |
| NAV-53 | Map source in share link | DOCUMENTED | KEEP | 13a |
| NAV-54 | Attribution | DOCUMENTED | KEEP | 13a |
| NAV-55 | Imagery comparison lease | DOCUMENTED | LATER | 13a |
| NAV-56 | Viewer construction | THIN | KEEP | 13a |
| NAV-57 | Idle render governor | THIN | KEEP | 13a |
| NAV-58 | One-frame render requests | DOCUMENTED | KEEP | 13a |
| NAV-59 | Hidden-tab pause | DOCUMENTED | KEEP | 13a |
| NAV-60 | Apple Metal atmosphere workaround | DOCUMENTED | KEEP | 13a |
| NAV-61 | Sky atmosphere | DOCUMENTED | KEEP | 13a |
| NAV-62 | Fog policy | DOCUMENTED | KEEP | 13a |
| NAV-63 | FPS readout | DOCUMENTED | KEEP | 13a |
| NAV-64 | Celestial ring | DOCUMENTED | ADAPT | 13a |
| NAV-65 | Sun/moon ephemeris | DOCUMENTED | KEEP | 13a |
| NAV-66 | No day/night lighting | MISSING | LATER | 13a |
| NAV-67 | Clean view | DOCUMENTED | KEEP | 13a |
| NAV-68 | Loader and first-run reveal | DOCUMENTED | KEEP | 13a |
| NAV-69 | Status strip | DOCUMENTED | KEEP | 13a |
| NAV-70 | Camera button semantics | DOCUMENTED | KEEP | 13a |
| NAV-71 | Measured baseline | DOCUMENTED | KEEP | 13a |
| AIR-01 | Worldwide civil snapshot | DOCUMENTED | KEEP | 13b |
| AIR-02 | Auth modes for the snapshot | DOCUMENTED | ADAPT | 13b |
| AIR-03 | Credit governor and serve-stale | DOCUMENTED | KEEP | 13b |
| AIR-04 | Regional fallback | DOCUMENTED | KEEP | 13b |
| AIR-05 | Render-behind interpolation | THIN | KEEP | 13b |
| AIR-06 | Dead-reckoning, coast and warm-up | MISSING | KEEP | 13b |
| AIR-07 | Course blending and slew limit | MISSING | KEEP | 13b |
| AIR-08 | Fleet tick cadence | DOCUMENTED | KEEP | 13b |
| AIR-09 | Sticky metadata | THIN | KEEP | 13b |
| AIR-10 | Removal and landed fast-cull | DOCUMENTED | KEEP | 13b |
| AIR-11 | Source-backoff stale cue | DOCUMENTED | KEEP | 13b |
| AIR-12 | Billboard silhouettes and sizes | DOCUMENTED | KEEP | 13b |
| AIR-13 | Limb recession | DOCUMENTED | KEEP | 13b |
| AIR-14 | Horizon cull | DOCUMENTED | KEEP | 13b |
| AIR-15 | Screen-projected rotation | THIN | KEEP | 13b |
| AIR-16 | Tint states | DOCUMENTED | KEEP | 13b |
| AIR-17 | Military duplicate suppression | DOCUMENTED | KEEP | 13b |
| AIR-18 | No filters | DOCUMENTED | ADAPT | 13b |
| AIR-19 | Layer status row | DOCUMENTED | KEEP | 13b |
| AIR-20 | Military snapshot | DOCUMENTED | SKIP | 13b |
| AIR-21 | Cooldown and stale-with-age | DOCUMENTED | ? | 13b |
| AIR-22 | Ground aircraft | DOCUMENTED | ? | 13b |
| AIR-23 | Weight-class models | DOCUMENTED | ? | 13b |
| AIR-24 | Military trail backfill | THIN | ? | 13b |
| AIR-25 | "Absence means nothing" notice | DOCUMENTED | ? | 13b |
| AIR-26 | Local ADS-B layer | DOCUMENTED | LATER | 13b |
| AIR-27 | Browser radio (WebUSB) | MISSING | LATER | 13b |
| AIR-28 | Mode S frame detection | MISSING | LATER | 13b |
| AIR-29 | Message decode and position | MISSING | LATER | 13b |
| AIR-30 | Receiver location reference (LOCATE) | DOCUMENTED | LATER | 13b |
| AIR-31 | Decoder feeds (dump1090/readsb/tar1090, 978 MHz UAT) | DOCUMENTED | LATER | 13b |
| AIR-32 | Local motion | DOCUMENTED | LATER | 13b |
| AIR-33 | Local trail (heard history) | DOCUMENTED | LATER | 13b |
| AIR-34 | Local click card | DOCUMENTED | LATER | 13b |
| AIR-35 | Tuner modes (FM vs ADS-B) | DOCUMENTED | LATER | 13b |
| AIR-36 | Navigation-integrity cells | DOCUMENTED | KEEP | 13b |
| AIR-37 | Regional sample, not worldwide | DOCUMENTED | KEEP | 13b |
| AIR-38 | Camera-driven refresh | DOCUMENTED | KEEP | 13b |
| AIR-39 | Honest wording | DOCUMENTED | KEEP | 13b |
| AIR-40 | Legend with live counts | DOCUMENTED | KEEP | 13b |
| AIR-41 | Type-aware classification | THIN | KEEP | 13b |
| AIR-42 | Class labels | THIN | KEEP | 13b |
| AIR-43 | Per-class 2D and 3D scale tables | DOCUMENTED | KEEP | 13b |
| AIR-44 | Bundled models | DOCUMENTED | KEEP | 13b |
| AIR-45 | Fleet 3D admission | DOCUMENTED | KEEP | 13b |
| AIR-46 | Model tint and light look | DOCUMENTED | KEEP | 13b |
| AIR-47 | Style-flip reload | THIN | ADAPT | 13b |
| AIR-48 | Tracked model regime | DOCUMENTED | KEEP | 13b |
| AIR-49 | Model load failure bound | DOCUMENTED | KEEP | 13b |
| AIR-50 | Easter-egg conversion | DOCUMENTED | SKIP | 13b |
| AIR-51 | Type and registration by address | DOCUMENTED | ADAPT | 13b |
| AIR-52 | Route by callsign | DOCUMENTED | ADAPT | 13b |
| AIR-53 | Route plausibility gate | DOCUMENTED | KEEP | 13b |
| AIR-54 | Enrichment queue | DOCUMENTED | KEEP | 13b |
| AIR-55 | Server cache | DOCUMENTED | KEEP | 13b |
| AIR-56 | Why enrichment exists | DOCUMENTED | KEEP | 13b |
| AIR-57 | Tracked-aircraft trail | DOCUMENTED | KEEP | 13b |
| AIR-58 | Trail body vs head | DOCUMENTED | KEEP | 13b |
| AIR-59 | Seeding at selection | THIN | KEEP | 13b |
| AIR-60 | Civil history backfill | MISSING | ADAPT | 13b |
| AIR-61 | Backfill height handling | DOCUMENTED | KEEP | 13b |
| AIR-62 | Military history | DOCUMENTED | ? | 13b |
| AIR-63 | Grounded-aircraft trails | DOCUMENTED | KEEP | 13b |
| AIR-64 | Local-receiver trail | DOCUMENTED | LATER | 13b |
| AIR-65 | Trail click ownership | DOCUMENTED | KEEP | 13b |
| AIR-66 | Replay / time scrubbing for aircraft | DOCUMENTED | LATER | 13b |
| AIR-67 | Select by click | MISSING | KEEP | 13b |
| AIR-68 | Pick rules | DOCUMENTED | KEEP | 13b |
| AIR-69 | Initial follow framing | DOCUMENTED | KEEP | 13b |
| AIR-70 | Single camera writer and minimum range | MISSING | KEEP | 13b |
| AIR-71 | Shared display position | THIN | KEEP | 13b |
| AIR-72 | Deselect in place | DOCUMENTED | KEEP | 13b |
| AIR-73 | Eviction vs deselect | DOCUMENTED | KEEP | 13b |
| AIR-74 | Cross-layer handoff | DOCUMENTED | KEEP | 13b |
| AIR-75 | Share-link restore | DOCUMENTED | KEEP | 13b |
| AIR-76 | Escape deselect | DOCUMENTED | KEEP | 13b |
| AIR-77 | Context slot | DOCUMENTED | KEEP | 13b |
| AIR-78 | Contacts window | DOCUMENTED | ADAPT | 13b |
| AIR-79 | What counts as "nearby" | MISSING | KEEP | 13b |
| AIR-80 | Cohort honesty | DOCUMENTED | KEEP | 13b |
| AIR-81 | PREVIOUS / FOCUS / NEXT | DOCUMENTED | KEEP | 13b |
| AIR-82 | Pages and compass arrows | DOCUMENTED | ADAPT | 13b |
| AIR-83 | Filters (voice/tool only) | DOCUMENTED | LATER | 13b |
| AIR-84 | Detection tie-in | DOCUMENTED | LATER | 13b |
| AIR-85 | Entry gate and transaction | THIN | LATER | 13b |
| AIR-86 | Eye camera | DOCUMENTED | LATER | 13b |
| AIR-87 | Ground rules | DOCUMENTED | LATER | 13b |
| AIR-88 | Own aircraft hidden, traffic as pips | MISSING | LATER | 13b |
| AIR-89 | Instruments, tapes, readouts | DOCUMENTED | LATER | 13b |
| AIR-90 | Estimated flight plan and destination arrow | DOCUMENTED | LATER | 13b |
| AIR-91 | Vision chips | DOCUMENTED | LATER | 13b |
| AIR-92 | Contact panel and briefing carousel | DOCUMENTED | LATER | 13b |
| AIR-93 | Cloud and weather effects | DOCUMENTED | LATER | 13b |
| AIR-94 | Exit rules and grace | DOCUMENTED | LATER | 13b |
| AIR-95 | Narrow-window layout | DOCUMENTED | LATER | 13b |
| AIR-96 | Camera tools refused | DOCUMENTED | LATER | 13b |
| AIR-97 | Ground traffic is drawn | DOCUMENTED | KEEP | 13b |
| AIR-98 | Four-tier ground floor | MISSING | KEEP | 13b |
| AIR-99 | Geoid correction | THIN | KEEP | 13b |
| AIR-100 | Model on ground | THIN | KEEP | 13b |
| AIR-101 | Floor flap resilience | DOCUMENTED | KEEP | 13b |
| AIR-102 | Runway/taxiway geometry | DOCUMENTED | ADAPT | 13b |
| AIR-103 | Squawk rule | DOCUMENTED | KEEP | 13b |
| AIR-104 | ADS-B emergency field | DOCUMENTED | KEEP | 13b |
| AIR-105 | Severity and tie-breaking | DOCUMENTED | KEEP | 13b |
| AIR-106 | Provenance wording | DOCUMENTED | KEEP | 13b |
| AIR-107 | Latest-broadcast semantics | DOCUMENTED | KEEP | 13b |
| AIR-108 | Primary vs fallback difference | DOCUMENTED | KEEP | 13b |
| AIR-109 | Where it shows | THIN | ADAPT | 13b |
| AIR-110 | Local receivers and emergencies | MISSING | ? | 13b |
| AIR-111 | Tracked card (civil and military) | DOCUMENTED | KEEP | 13b |
| AIR-112 | Altitude format | DOCUMENTED | KEEP | 13b |
| AIR-113 | Label identity chain | DOCUMENTED | KEEP | 13b |
| AIR-114 | Local card | DOCUMENTED | LATER | 13b |
| AIR-115 | Ambient hover text | DOCUMENTED | ? | 13b |
| AIR-116 | DETECT labels on aircraft | DOCUMENTED | ADAPT | 13b |
| AIR-117 | Cockpit subject exclusion | THIN | LATER | 13b |
| AIR-118 | Colour/tint priority | DOCUMENTED | KEEP | 13b |
| AIR-119 | Analyst query over loaded layers | DOCUMENTED | KEEP | 13b |
| AIR-120 | Row fields | DOCUMENTED | KEEP | 13b |
| AIR-121 | Emergency filter by source | DOCUMENTED | KEEP | 13b |
| AIR-122 | World-query tools | DOCUMENTED | ADAPT | 13b |
| AIR-123 | Frame what is overhead | DOCUMENTED | KEEP | 13b |
| AIR-124 | Select nearest aircraft near a place | DOCUMENTED | KEEP | 13b |
| AIR-125 | Follow from a shared link | DOCUMENTED | KEEP | 13b |
| AIR-126 | Situation brief and military awareness | DOCUMENTED | SKIP | 13b |
| AIR-127 | Identity narration honesty | DOCUMENTED | KEEP | 13b |
| SAT-01 | Core catalogue | DOCUMENTED | KEEP | 13c |
| SAT-02 | Element proxy and cache | DOCUMENTED | KEEP | 13c |
| SAT-03 | Refresh and rebuild | DOCUMENTED | KEEP | 13c |
| SAT-04 | Position computation | THIN | KEEP | 13c |
| SAT-05 | Update cadence tiers | MISSING | KEEP | 13c |
| SAT-06 | DENSE chip (Starlink shell) | DOCUMENTED | LATER | 13c |
| SAT-07 | Dense update budget | DOCUMENTED | LATER | 13c |
| SAT-08 | Dense failure handling | DOCUMENTED | LATER | 13c |
| SAT-09 | Satellite classes and icon colours | DOCUMENTED | KEEP | 13c |
| SAT-10 | Dot size per group | MISSING | KEEP | 13c |
| SAT-11 | Row legend with live counts | DOCUMENTED | KEEP | 13c |
| SAT-12 | ISS hero marker | DOCUMENTED | KEEP | 13c |
| SAT-13 | Orbit ring construction | MISSING | KEEP | 13c |
| SAT-14 | Ring locked to dot | DOCUMENTED | KEEP | 13c |
| SAT-15 | Ring style | MISSING | KEEP | 13c |
| SAT-16 | Show points / show orbits switches | THIN | ADAPT | 13c |
| SAT-17 | Click to select and follow | DOCUMENTED | KEEP | 13c |
| SAT-18 | Tracked card | DOCUMENTED | KEEP | 13c |
| SAT-19 | Docked-vehicle consolidation | DOCUMENTED | LATER | 13c |
| SAT-20 | Deselect paths | DOCUMENTED | KEEP | 13c |
| SAT-21 | Restore from share link / voice | DOCUMENTED | KEEP | 13c |
| SAT-22 | Voice and tool lookup of a satellite | THIN | ADAPT | 13c |
| SAT-23 | Context record for voice | DOCUMENTED | KEEP | 13c |
| SAT-24 | Analyst records | DOCUMENTED | KEEP | 13c |
| SAT-25 | DETECT overlay membership | DOCUMENTED | LATER | 13c |
| SAT-26 | Focus fade around tracked target | THIN | LATER | 13c |
| SAT-27 | Layer health words | DOCUMENTED | KEEP | 13c |
| SAT-28 | Pass prediction engine | DOCUMENTED | KEEP | 13c |
| SAT-29 | Visibility maths | DOCUMENTED | KEEP | 13c |
| SAT-30 | Voice pass actions | DOCUMENTED | KEEP | 13c |
| SAT-31 | External-tool pass lookup | DOCUMENTED | KEEP | 13c |
| SAT-32 | Satellites overhead | DOCUMENTED | KEEP | 13c |
| SAT-33 | Name resolution for passes | DOCUMENTED | KEEP | 13c |
| SAT-34 | Orbital Watch scene | DOCUMENTED | LATER | 13c |
| SAT-35 | Celestial ring: look | DOCUMENTED | SKIP | 13c |
| SAT-36 | Celestial ring: direction maths | THIN | LATER | 13c |
| SAT-37 | Celestial ring: lifecycle | DOCUMENTED | LATER | 13c |
| SAT-38 | Celestial ring: cost control | DOCUMENTED | LATER | 13c |
| SAT-39 | Sun, moon and stars in the 3D scene | MISSING | KEEP | 13c |
| SAT-40 | Real time only | MISSING | ADAPT | 13c |
| SAT-41 | Ground track and footprint | MISSING | LATER | 13c |
| SAT-42 | Pass-prediction UI | DOCUMENTED | ADAPT | 13c |
| SAT-43 | Satellite basemap confusion rule | THIN | KEEP | 13c |
| SEA-01 | Launch feed | DOCUMENTED | LATER | 13c |
| SEA-02 | Record normalisation | DOCUMENTED | LATER | 13c |
| SEA-03 | Marker colour by operator | DOCUMENTED | LATER | 13c |
| SEA-04 | Roster and hover preview | DOCUMENTED | LATER | 13c |
| SEA-05 | Select, isolate, step | DOCUMENTED | LATER | 13c |
| SEA-06 | Detail panel | DOCUMENTED | LATER | 13c |
| SEA-07 | Match launch to a real satellite | MISSING | LATER | 13c |
| SEA-08 | Approximate orbit when no match | THIN | LATER | 13c |
| SEA-09 | Failed-launch rule | DOCUMENTED | LATER | 13c |
| SEA-10 | Insertion time | MISSING | LATER | 13c |
| SEA-11 | Reconstructed ascent | MISSING | LATER | 13c |
| SEA-12 | Supplied trajectory | DOCUMENTED | LATER | 13c |
| SEA-13 | Stage recovery endpoints | DOCUMENTED | SKIP | 13c |
| SEA-14 | Replay transport | DOCUMENTED | SKIP | 13c |
| SEA-15 | Replay time compression | MISSING | SKIP | 13c |
| SEA-16 | Replay camera | DOCUMENTED | SKIP | 13c |
| SEA-17 | Replay vehicle glyph | DOCUMENTED | SKIP | 13c |
| SEA-18 | Launch-zone ring | DOCUMENTED | SKIP | 13c |
| SEA-19 | Live payload telemetry | DOCUMENTED | LATER | 13c |
| SEA-20 | Satellite dependency and mode exclusivity | DOCUMENTED | LATER | 13c |
| SEA-21 | Analyst and tool access | DOCUMENTED | LATER | 13c |
| SEA-22 | Upcoming launches | DOCUMENTED | ADAPT | 13c |
| SEA-23 | Live ship layer | DOCUMENTED | KEEP | 13c |
| SEA-24 | Subscription | DOCUMENTED | KEEP | 13c |
| SEA-25 | Sentinel clean-up | MISSING | KEEP | 13c |
| SEA-26 | Static-data merge | MISSING | KEEP | 13c |
| SEA-27 | Server cache | MISSING | KEEP | 13c |
| SEA-28 | Feed health words and recovery | DOCUMENTED | KEEP | 13c |
| SEA-29 | View-area request | THIN | ADAPT | 13c |
| SEA-30 | Record reconciliation | DOCUMENTED | KEEP | 13c |
| SEA-31 | Chevron markers | MISSING | KEEP | 13c |
| SEA-32 | Type decoding | DOCUMENTED | KEEP | 13c |
| SEA-33 | Ambient cards | DOCUMENTED | KEEP | 13c |
| SEA-34 | Selected card and HUD line | DOCUMENTED | KEEP | 13c |
| SEA-35 | Trail | DOCUMENTED | KEEP | 13c |
| SEA-36 | Motion between polls | MISSING | ADAPT | 13c |
| SEA-37 | Sea-surface height | DOCUMENTED | KEEP | 13c |
| SEA-38 | 3D ship model | DOCUMENTED | LATER | 13c |
| SEA-39 | Vessel tools | DOCUMENTED | KEEP | 13c |
| SEA-40 | Follow a vessel by voice | DOCUMENTED | KEEP | 13c |
| SEA-41 | Nearest camera cross-link | DOCUMENTED | ADAPT | 13c |
| SEA-42 | Ports | DOCUMENTED | ADAPT | 13c |
| SEA-43 | Context / DETECT participation | DOCUMENTED | KEEP | 13c |
| SEA-44 | Cable layer data | DOCUMENTED | ADAPT | 13c |
| SEA-45 | Cable lines | DOCUMENTED | ADAPT | 13c |
| SEA-46 | Landing points | DOCUMENTED | ADAPT | 13c |
| SEA-47 | Reference labels and stems | DOCUMENTED | ADAPT | 13c |
| SEA-48 | Click behaviour | DOCUMENTED | ADAPT | 13c |
| SEA-49 | Cable lookup tool | DOCUMENTED | ADAPT | 13c |
| SEA-50 | Cable to vessel ties | MISSING | LATER | 13c |
| SEA-51 | Teardown discipline | DOCUMENTED | KEEP | 13c |
| GND-01 | Street Traffic layer toggle (off by default) | DOCUMENTED | ADAPT | 13d |
| GND-02 | Road source chips: TomTom / OSM / Hybrid | DOCUMENTED | KEEP | 13d |
| GND-03 | Two-pass road load (major then full) | DOCUMENTED | KEEP | 13d |
| GND-04 | Viewport targeting / horizon-gaze pull-back | DOCUMENTED | KEEP | 13d |
| GND-05 | Debounce, overlap and shift thresholds | DOCUMENTED | KEEP | 13d |
| GND-06 | Live flow colouring (green/amber/red) | DOCUMENTED | ADAPT | 13d |
| GND-07 | Flow -> OSM road snapping (35 m, 30 degrees, 7 samples, median) | MISSING | KEEP | 13d |
| GND-08 | Directional flow (per travel direction) | THIN | KEEP | 13d |
| GND-09 | Hybrid de-duplication | MISSING | KEEP | 13d |
| GND-10 | Road-class filtering of flow segments | MISSING | KEEP | 13d |
| GND-11 | Closure handling | THIN | KEEP | 13d |
| GND-12 | Uncovered-roads policy: simulate (default) or hide | MISSING | KEEP | 13d |
| GND-13 | Dot spawn count by altitude and class | THIN | KEEP | 13d |
| GND-14 | Dot speed model and noise | DOCUMENTED | KEEP | 13d |
| GND-15 | One-way roads | MISSING | KEEP | 13d |
| GND-16 | Traffic-light pauses | MISSING | ADAPT | 13d |
| GND-17 | Fade in/out and recycle at road end | THIN | KEEP | 13d |
| GND-18 | Frame-time cap | MISSING | KEEP | 13d |
| GND-19 | Dot budget and fair allocation | DOCUMENTED | KEEP | 13d |
| GND-20 | Surface alignment | THIN | KEEP | 13d |
| GND-21 | Jam density mode: platoons + stop-and-go | THIN | KEEP | 13d |
| GND-22 | Heat-line corridors (jam pulsing, slow faint, 400 cap) | DOCUMENTED | ADAPT | 13d |
| GND-23 | Jam-visualisation mode param: none / density / heatline / both | DOCUMENTED | LATER | 13d |
| GND-24 | Style-preset dot treatment (mono / retro / normal) | THIN | KEEP | 13d |
| GND-25 | Sync chip and status text | DOCUMENTED | KEEP | 13d |
| GND-26 | Failure honesty | DOCUMENTED | KEEP | 13d |
| GND-27 | TomTom proxy + budget governor | DOCUMENTED | ADAPT | 13d |
| GND-28 | Flow tool for assistants | DOCUMENTED | KEEP | 13d |
| GND-29 | DETECT overlay feed from dots | DOCUMENTED | LATER | 13d |
| GND-30 | Attribution | DOCUMENTED | KEEP | 13d |
| GND-31 | Known issue: late fill in dense cores | DOCUMENTED | ? | 13d |
| GND-32 | Dev-only timing diagnostics | MISSING | SKIP | 13d |
| GND-33 | Transit layer toggle | DOCUMENTED | LATER | 13d |
| GND-34 | Feed registry (7 regions) | DOCUMENTED | ADAPT | 13d |
| GND-35 | Mode inference from route id | THIN | KEEP | 13d |
| GND-36 | 15 s polling, nearest feeds only | DOCUMENTED | KEEP | 13d |
| GND-37 | Server snapshot cache, conditional requests, back-off | DOCUMENTED | KEEP | 13d |
| GND-38 | Delayed playback (interpolation between real reports) | DOCUMENTED | KEEP | 13d |
| GND-39 | Plausibility filter | DOCUMENTED | KEEP | 13d |
| GND-40 | Course smoothing and STOPPED logic | DOCUMENTED | KEEP | 13d |
| GND-41 | Time-source flags | MISSING | KEEP | 13d |
| GND-42 | Stale/evict policy | DOCUMENTED | KEEP | 13d |
| GND-43 | Fleet cap 15,000 | DOCUMENTED | KEEP | 13d |
| GND-44 | Off-view vehicles skip animation | DOCUMENTED | KEEP | 13d |
| GND-45 | Ground anchoring | DOCUMENTED | KEEP | 13d |
| GND-46 | Selected-vehicle trail | DOCUMENTED | KEEP | 13d |
| GND-47 | Server trail history (Boston only) | DOCUMENTED | LATER | 13d |
| GND-48 | Selection card | DOCUMENTED | KEEP | 13d |
| GND-49 | DETECT labels for transit | DOCUMENTED | LATER | 13d |
| GND-50 | Mode icons and look variants | DOCUMENTED | KEEP | 13d |
| GND-51 | Transit tool for assistants | DOCUMENTED | LATER | 13d |
| GND-52 | Operator credits while in use | DOCUMENTED | KEEP | 13d |
| GND-53 | Bikeshare layer | DOCUMENTED | SKIP | 13d |
| GND-54 | 32-system registry | DOCUMENTED | ADAPT | 13d |
| GND-55 | Info once, status every 60 s | DOCUMENTED | KEEP | 13d |
| GND-56 | Colour/size encoding | DOCUMENTED | KEEP | 13d |
| GND-57 | Selected card, warnings, DETECT, 8,000 cap | DOCUMENTED | KEEP | 13d |
| GND-58 | Proxy hardening | DOCUMENTED | KEEP | 13d |
| GND-59 | Bike-share tool | DOCUMENTED | LATER | 13d |
| GND-60 | CCTV layer toggle and enable choreography | DOCUMENTED | ADAPT | 13d |
| GND-61 | Camera catalogue assembled from "packs" | DOCUMENTED | ADAPT | 13d |
| GND-62 | Per-pack rules (caps, anchors, status filters, heading source) | DOCUMENTED | ADAPT | 13d |
| GND-63 | Operator overrides: local file and inline JSON sources; curated packs | THIN | LATER | 13d |
| GND-64 | Opt-in agreement-gated pack (New York) | DOCUMENTED | SKIP | 13d |
| GND-65 | Seed cameras fallback | DOCUMENTED | ? | 13d |
| GND-66 | Camera pose model ("raw prior") | DOCUMENTED | KEEP | 13d |
| GND-67 | Heading provenance: ESTIMATED tag and dashed wires | DOCUMENTED | KEEP | 13d |
| GND-68 | Camera icons | DOCUMENTED | KEEP | 13d |
| GND-69 | Click / tap to activate | DOCUMENTED | KEEP | 13d |
| GND-70 | World-click focus request | DOCUMENTED | KEEP | 13d |
| GND-71 | NEAREST | THIN | KEEP | 13d |
| GND-72 | PREV / NEXT / dropdown / FOCUS | DOCUMENTED | KEEP | 13d |
| GND-73 | Focus framing | DOCUMENTED | KEEP | 13d |
| GND-74 | Hand-off from a tracked target | THIN | KEEP | 13d |
| GND-75 | Auto hop | DOCUMENTED | LATER | 13d |
| GND-76 | Coverage modes OFF / ON / VIEWSHED | DOCUMENTED | KEEP | 13d |
| GND-77 | Coverage neighbours | DOCUMENTED | KEEP | 13d |
| GND-78 | Viewshed colours | DOCUMENTED | KEEP | 13d |
| GND-79 | Projection (monitor plane) toggle | DOCUMENTED | KEEP | 13d |
| GND-80 | Obstruction probe and footprint lift | THIN | KEEP | 13d |
| GND-81 | Ground resolution per surface regime | THIN | KEEP | 13d |
| GND-82 | Shipped precomputed ground heights | THIN | LATER | 13d |
| GND-83 | Snapshot frames | DOCUMENTED | KEEP | 13d |
| GND-84 | Live video (HLS) | THIN | LATER | 13d |
| GND-85 | Server HLS relay limits | DOCUMENTED | ? | 13d |
| GND-86 | Media proxy hardening | THIN | ? | 13d |
| GND-87 | Health tracking | DOCUMENTED | KEEP | 13d |
| GND-88 | Source badge | DOCUMENTED | KEEP | 13d |
| GND-89 | Ambient thumbnail cards | DOCUMENTED | LATER | 13d |
| GND-90 | Hover-summoned card | DOCUMENTED | LATER | 13d |
| GND-91 | Optional protected thumbnail for the active camera | MISSING | SKIP | 13d |
| GND-92 | Calibration gizmo (ADJUST) | DOCUMENTED | KEEP | 13d |
| GND-93 | Click-to-type pose readout | DOCUMENTED | KEEP | 13d |
| GND-94 | Save-gated calibration persistence | DOCUMENTED | KEEP | 13d |
| GND-95 | Gizmo commit grounding | DOCUMENTED | KEEP | 13d |
| GND-96 | Scene summary typing line | DOCUMENTED | LATER | 13d |
| GND-97 | Panel auto-expand on active change | DOCUMENTED | KEEP | 13d |
| GND-98 | Voice/assistant control of cameras | DOCUMENTED | KEEP | 13d |
| GND-99 | Camera tools for external assistants | DOCUMENTED | ? | 13d |
| GND-100 | Content stance | DOCUMENTED | ? | 13d |
| GND-101 | Credits / attribution per pack | DOCUMENTED | ? | 13d |
| GND-102 | ALPR layer | DOCUMENTED | SKIP | 13d |
| GND-103 | Viewport snapping and reuse | DOCUMENTED | SKIP | 13d |
| GND-104 | Caps and "coverage limited" | DOCUMENTED | SKIP | 13d |
| GND-105 | Badge markers, scale by altitude, ground placement | DOCUMENTED | SKIP | 13d |
| GND-106 | Facing wedge | DOCUMENTED | SKIP | 13d |
| GND-107 | Click card | DOCUMENTED | SKIP | 13d |
| GND-108 | SHOW NEAREST chip | DOCUMENTED | SKIP | 13d |
| GND-109 | Row status strings and attribution timing | DOCUMENTED | SKIP | 13d |
| GND-110 | Assistant tool for ALPR | MISSING | SKIP | 13d |
| GND-111 | Provider-neutral core + registry | DOCUMENTED | ADAPT | 13d |
| GND-112 | One provider lit at a time | DOCUMENTED | KEEP | 13d |
| GND-113 | Mapillary coverage + photos | DOCUMENTED | ADAPT | 13d |
| GND-114 | KartaView coverage + photos | DOCUMENTED | ADAPT | 13d |
| GND-115 | Google Street View (Google 3D map only) | DOCUMENTED | LATER | 13d |
| GND-116 | IMAGERY (ALL/360/FLAT) and SINCE filters | DOCUMENTED | KEEP | 13d |
| GND-117 | Embedded viewer: EXPAND, FIT/FILL, FOLLOW, close; 360 WebGL | DOCUMENTED | KEEP | 13d |
| GND-118 | Sequence stepping + keyboard | DOCUMENTED | KEEP | 13d |
| GND-119 | Surface mode | DOCUMENTED | KEEP | 13d |
| GND-120 | Position marker, cones, credits, key states, floating panel, share state | DOCUMENTED | KEEP | 13d |
| GND-121 | Directions A->B (drive/walk/bike) | DOCUMENTED | KEEP | 13d |
| GND-122 | Pointer ownership message | MISSING | KEEP | 13d |
| GND-123 | Maneuver dot height | DOCUMENTED | KEEP | 13d |
| GND-124 | Active-step highlight during FLY | DOCUMENTED | KEEP | 13d |
| GND-125 | Route flight cinematics | THIN | KEEP | 13d |
| GND-126 | Voice: draw route then "fly it" | THIN | KEEP | 13d |
| GND-127 | Assistant route tool | DOCUMENTED | LATER | 13d |
| GND-128 | 30-day day strip, boxes, modes IMAGE / VS BASEMAP / A-B, swipe, export, START HERE ranking, share | DOCUMENTED | KEEP | 13d |
| GND-129 | Assistant tool for latest clear image | DOCUMENTED | KEEP | 13d |
| GND-130 | Radio layer (markers by category, clusters) | THIN | KEEP | 13d |
| GND-131 | Analog tuner | DOCUMENTED | KEEP | 13d |
| GND-132 | Playback controls, volume, voice hand-off | DOCUMENTED | KEEP | 13d |
| GND-133 | Voice radio control with place/category/country/name | DOCUMENTED | KEEP | 13d |
| GND-134 | Local RTL-SDR FM | DOCUMENTED | LATER | 13d |
| GND-135 | Local ADS-B decode and decoder feeds | THIN | LATER | 13d |
| GND-136 | Device selection memory | MISSING | LATER | 13d |
| GND-137 | Installations layer | DOCUMENTED | SKIP | 13d |
| GND-138 | Contacts coupling | DOCUMENTED | SKIP | 13d |
| ENV-01 | Wind layer on/off, row and WEATHER card | DOCUMENTED | ADAPT | 13e |
| ENV-02 | Animated wind trails | THIN | ADAPT | 13e |
| ENV-03 | Model choice (GFS / ECMWF) | DOCUMENTED | KEEP | 13e |
| ENV-04 | Colour field (None / Speed / Pressure / Temperature) | DOCUMENTED | ADAPT | 13e |
| ENV-05 | Wind units chips | DOCUMENTED | KEEP | 13e |
| ENV-06 | Inspect centre / reading | DOCUMENTED | KEEP | 13e |
| ENV-07 | Pause / Resume, reduced motion, hidden-tab suspend | DOCUMENTED | KEEP | 13e |
| ENV-08 | Relief shading helper | DOCUMENTED | LATER | 13e |
| ENV-09 | Canvas particle fallback | THIN | KEEP | 13e |
| ENV-10 | Wind query by voice/automation | DOCUMENTED | KEEP | 13e |
| ENV-11 | Wind share state | DOCUMENTED | KEEP | 13e |
| ENV-12 | Rain radar layer | DOCUMENTED | SKIP | 13e |
| ENV-13 | Satellite clouds, regional (GOES) | DOCUMENTED | SKIP | 13e |
| ENV-14 | Satellite clouds, global | DOCUMENTED | KEEP | 13e |
| ENV-15 | Clouds-only vs Full image | DOCUMENTED | KEEP | 13e |
| ENV-16 | Lightning density | DOCUMENTED | SKIP | 13e |
| ENV-17 | Per-product controls | DOCUMENTED | KEEP | 13e |
| ENV-18 | Observed-history timeline (Earlier, Later, slider, Latest, Play) | DOCUMENTED | KEEP | 13e |
| ENV-19 | Delayed/stale flags | DOCUMENTED | KEEP | 13e |
| ENV-20 | Frame prefetch and failure retention | DOCUMENTED | KEEP | 13e |
| ENV-21 | Raised translucent shells on photoreal 3D map | DOCUMENTED | KEEP | 13e |
| ENV-22 | Globe tile hosting and imagery order | DOCUMENTED | KEEP | 13e |
| ENV-23 | WEATHER panel | DOCUMENTED | KEEP | 13e |
| ENV-24 | Weather map image tool | THIN | ADAPT | 13e |
| ENV-25 | Recording/clean view hides weather chrome | DOCUMENTED | KEEP | 13e |
| ENV-26 | Cyclone advisories layer | DOCUMENTED | SKIP | 13e |
| ENV-27 | Advisory consistency rule | DOCUMENTED | KEEP | 13e |
| ENV-28 | Storms list in card, auto-select, fly-to | DOCUMENTED | KEEP | 13e |
| ENV-29 | Cone/horizon rules | DOCUMENTED | KEEP | 13e |
| ENV-30 | Cyclone voice/automation query | DOCUMENTED | LATER | 13e |
| ENV-31 | Earthquake discs (24 h, M2.5+) | DOCUMENTED | ADAPT | 13e |
| ENV-32 | Magnitude labels | DOCUMENTED | KEEP | 13e |
| ENV-33 | 60-s refresh, DEGRADED/UNAVAILABLE states | DOCUMENTED | KEEP | 13e |
| ENV-34 | Earthquake analyst records and tool | DOCUMENTED | KEEP | 13e |
| ENV-35 | No click card | DOCUMENTED | ? | 13e |
| ENV-36 | Active fires layer | DOCUMENTED | ADAPT | 13e |
| ENV-37 | Severity colour and size | DOCUMENTED | KEEP | 13e |
| ENV-38 | Fire cards (ambient 18, selected, cell) | DOCUMENTED | KEEP | 13e |
| ENV-39 | Click to inspect and dive; nearest-camera hand-off | DOCUMENTED | ADAPT | 13e |
| ENV-40 | Row status/KEY REQUIRED | DOCUMENTED | KEEP | 13e |
| ENV-41 | Fire perimeters | DOCUMENTED | SKIP | 13e |
| ENV-42 | Incident card with verified official-page link | DOCUMENTED | SKIP | 13e |
| ENV-43 | Fire tools | DOCUMENTED | LATER | 13e |
| ENV-44 | Submarine cables | DOCUMENTED | ADAPT | 13e |
| ENV-45 | Cable and landing labels | DOCUMENTED | KEEP | 13e |
| ENV-46 | Cable click | DOCUMENTED | ? | 13e |
| ENV-47 | Data centres (4,351) | DOCUMENTED | ADAPT | 13e |
| ENV-48 | Data-centre click/selection | DOCUMENTED | ADAPT | 13e |
| ENV-49 | Dams (704) | DOCUMENTED | SKIP | 13e |
| ENV-50 | Mapped installations | DOCUMENTED | SKIP | 13e |
| ENV-51 | Installations around a contact | DOCUMENTED | SKIP | 13e |
| ENV-52 | Infrastructure/cable tools | DOCUMENTED | KEEP | 13e |
| ENV-53 | Country/state/county outlines | DOCUMENTED | ADAPT | 13e |
| ENV-54 | Named physical/marine regions (e.g. a mountain range) | DOCUMENTED | KEEP | 13e |
| ENV-55 | Neighbourhood polygons | DOCUMENTED | ADAPT | 13e |
| ENV-56 | Regional news (Local Info) | DOCUMENTED | ADAPT | 13e |
| ENV-57 | Local weather readout | DOCUMENTED | KEEP | 13e |
| ENV-58 | Cockpit weather effects (WX) | DOCUMENTED | LATER | 13e |
| ENV-59 | Regional brief tool | DOCUMENTED | KEEP | 13e |
| ENV-60 | Situation brief | DOCUMENTED | KEEP | 13e |
| ENV-61 | Scenes panel | DOCUMENTED | KEEP | 13e |
| ENV-62 | Capture shot | DOCUMENTED | KEEP | 13e |
| ENV-63 | Update shot | DOCUMENTED | KEEP | 13e |
| ENV-64 | Shot rename/delete, scene new/delete | DOCUMENTED | KEEP | 13e |
| ENV-65 | LOAD a shot | DOCUMENTED | KEEP | 13e |
| ENV-66 | NEXT | DOCUMENTED | KEEP | 13e |
| ENV-67 | START / STOP / Escape | THIN | KEEP | 13e |
| ENV-68 | Voice scene control | DOCUMENTED | KEEP | 13e |
| ENV-69 | Run log | DOCUMENTED | KEEP | 13e |
| ENV-70 | Playback presentation mode | DOCUMENTED | KEEP | 13e |
| ENV-71 | Ordinary shot flight | DOCUMENTED | KEEP | 13e |
| ENV-72 | Authored camera move | DOCUMENTED | KEEP | 13e |
| ENV-73 | Anchors | DOCUMENTED | KEEP | 13e |
| ENV-74 | Layer reconciliation | THIN | KEEP | 13e |
| ENV-75 | Seek / scrub | THIN | KEEP | 13e |
| ENV-76 | Media-bounded holds | DOCUMENTED | KEEP | 13e |
| ENV-77 | Data packs (GeoJSON, PNG image, media) | DOCUMENTED | ADAPT | 13e |
| ENV-78 | Declarative click actions | DOCUMENTED | KEEP | 13e |
| ENV-79 | Import with preview | DOCUMENTED | KEEP | 13e |
| ENV-80 | Validation and migration | DOCUMENTED | KEEP | 13e |
| ENV-81 | Storage protection | DOCUMENTED | KEEP | 13e |
| ENV-82 | Edit details | DOCUMENTED | ADAPT | 13e |
| ENV-83 | Share scene / asset bundle | DOCUMENTED | KEEP | 13e |
| ENV-84 | Five generic showcase scenes | DOCUMENTED | ADAPT | 13e |
| ENV-85 | Nepal flood scene (25 shots) | DOCUMENTED | SKIP | 13e |
| ENV-86 | Event panel | THIN | ADAPT | 13e |
| ENV-87 | Flood path reconstruction | DOCUMENTED | SKIP | 13e |
| ENV-88 | Evidence cards and field reports | DOCUMENTED | KEEP | 13e |
| ENV-89 | Flood tool | DOCUMENTED | SKIP | 13e |
| ENV-90 | Built-in scene install and upgrade | DOCUMENTED | KEEP | 13e |
| ENV-91 | Voice annotate (pin, highlight, area, arrow, route, label) | DOCUMENTED | KEEP | 13e |
| ENV-92 | Real outlines for areas | DOCUMENTED | ADAPT | 13e |
| ENV-93 | Routes | DOCUMENTED | ADAPT | 13e |
| ENV-94 | Distance arrows (airport to airport) | THIN | KEEP | 13e |
| ENV-95 | Persistence and caps | DOCUMENTED | KEEP | 13e |
| ENV-96 | Clear annotations | DOCUMENTED | KEEP | 13e |
| ENV-97 | Draw tool (Area, Line, Pin) | DOCUMENTED | KEEP | 13e |
| ENV-98 | Live measure | DOCUMENTED | KEEP | 13e |
| ENV-99 | Camera assist for voice marks | DOCUMENTED | KEEP | 13e |
| ENV-100 | Fly the route | DOCUMENTED | KEEP | 13e |
| ENV-101 | Import own files (GeoJSON, KML, KMZ, GPX) | DOCUMENTED | KEEP | 13e |
| ENV-102 | Annotations in links and embed | DOCUMENTED | KEEP | 13e |
| ENV-103 | Console test surface | MISSING | SKIP | 13e |
| ENV-104 | Recording mode | DOCUMENTED | ADAPT | 13e |
| ENV-105 | Safe-frame overlay | THIN | ADAPT | 13e |
| ENV-106 | Clean view | DOCUMENTED | KEEP | 13e |
| ENV-107 | Share link copy | DOCUMENTED | KEEP | 13e |
| ENV-108 | Share restore and follow | DOCUMENTED | KEEP | 13e |
| ENV-109 | Open-in-app link tool | DOCUMENTED | KEEP | 13e |
| ENV-110 | Embed mode | DOCUMENTED | LATER | 13e |
| SYS-001 | Voice capsule and status states | DOCUMENTED | KEEP | 13f |
| SYS-002 | Two ways to talk: open-mic and push-to-talk | DOCUMENTED | KEEP | 13f |
| SYS-003 | Cloud realtime connection | DOCUMENTED | ADAPT | 13f |
| SYS-004 | Session defaults | DOCUMENTED | KEEP | 13f |
| SYS-005 | Model tier toggle (STD / MINI) | DOCUMENTED | KEEP | 13f |
| SYS-006 | Live spend readout | DOCUMENTED | KEEP | 13f |
| SYS-007 | Soft warning and hard cap | DOCUMENTED | KEEP | 13f |
| SYS-008 | Cost meter is bound to the negotiated model | THIN | KEEP | 13f |
| SYS-009 | ChatGPT/Codex sign-in alternative (experimental) | DOCUMENTED | SKIP | 13f |
| SYS-010 | Typed command path | THIN | KEEP | 13f |
| SYS-011 | Tool-call reliability | DOCUMENTED | KEEP | 13f |
| SYS-012 | Interruption and barge-in | DOCUMENTED | KEEP | 13f |
| SYS-013 | Progress narration and earcon | DOCUMENTED | KEEP | 13f |
| SYS-014 | Voice card | DOCUMENTED | KEEP | 13f |
| SYS-015 | Speech lint | THIN | LATER | 13f |
| SYS-016 | Per-turn latency spans | DOCUMENTED | KEEP | 13f |
| SYS-017 | Debug log sink | DOCUMENTED | ADAPT | 13f |
| SYS-018 | Visual grounding | DOCUMENTED | KEEP | 13f |
| SYS-019 | Scene context assembly | DOCUMENTED | KEEP | 13f |
| SYS-020 | Tracked-subject context slot | THIN | KEEP | 13f |
| SYS-021 | Named-view shorthand | DOCUMENTED | KEEP | 13f |
| SYS-022 | Disambiguation rules | DOCUMENTED | KEEP | 13f |
| SYS-023 | Voice benchmark harness (developer) | THIN | KEEP | 13f |
| SYS-024 | Backend seam | DOCUMENTED | KEEP | 13f |
| SYS-030 | Pointer tracking and dwell pick | DOCUMENTED | KEEP | 13f |
| SYS-031 | Snapshot timing | DOCUMENTED | KEEP | 13f |
| SYS-032 | Reticle | DOCUMENTED | KEEP | 13f |
| SYS-033 | Deictic sentinel substitution | DOCUMENTED | KEEP | 13f |
| SYS-034 | Pointer crop image | DOCUMENTED | KEEP | 13f |
| SYS-035 | Numbered referents | DOCUMENTED | KEEP | 13f |
| SYS-036 | Centre-of-query precedence | DOCUMENTED | KEEP | 13f |
| SYS-070 | Analyst engine | DOCUMENTED | KEEP | 13f |
| SYS-071 | Honest counting contract | DOCUMENTED | KEEP | 13f |
| SYS-072 | Result hand-off keys | DOCUMENTED | KEEP | 13f |
| SYS-073 | Loaded-data caveat | DOCUMENTED | KEEP | 13f |
| SYS-074 | Satellite pass caveats | DOCUMENTED | KEEP | 13f |
| SYS-080 | Define-once tool catalogue | DOCUMENTED | KEEP | 13f |
| SYS-081 | Surfaces (per-consumer exposure) | DOCUMENTED | KEEP | 13f |
| SYS-082 | Shared area argument | DOCUMENTED | KEEP | 13f |
| SYS-083 | Query tool families (40 definitions) | DOCUMENTED | KEEP | 13f |
| SYS-084 | Query: hazards and events | DOCUMENTED | KEEP | 13f |
| SYS-085 | Query: aircraft and ships | DOCUMENTED | KEEP | 13f |
| SYS-086 | Query: space | DOCUMENTED | KEEP | 13f |
| SYS-087 | Query: cameras and street | DOCUMENTED | ADAPT | 13f |
| SYS-088 | Query: places and movement | DOCUMENTED | ADAPT | 13f |
| SYS-089 | Query: environment | DOCUMENTED | KEEP | 13f |
| SYS-090 | Query: infrastructure and context | DOCUMENTED | ADAPT | 13f |
| SYS-091 | Show-in-app tool and view object | DOCUMENTED | KEEP | 13f |
| SYS-092 | MCP transports | DOCUMENTED | ADAPT | 13f |
| SYS-093 | MCP Apps globe panel | DOCUMENTED | LATER | 13f |
| SYS-094 | Panel-only relay tool | THIN | LATER | 13f |
| SYS-095 | Shared panel key across stdio processes | THIN | LATER | 13f |
| SYS-096 | Tool error classes and images | DOCUMENTED | KEEP | 13f |
| SYS-100 | Four HUD layouts | DOCUMENTED | KEEP | 13f |
| SYS-101 | HUD field set | DOCUMENTED | KEEP | 13f |
| SYS-102 | Altitude datum honesty | DOCUMENTED | KEEP | 13f |
| SYS-103 | Deterministic intel summary | DOCUMENTED | ADAPT | 13f |
| SYS-104 | AI summary line | DOCUMENTED | ADAPT | 13f |
| SYS-105 | Cyber skin | DOCUMENTED | LATER | 13f |
| SYS-106 | Cyber sonar | DOCUMENTED | LATER | 13f |
| SYS-107 | Logo gaze | DOCUMENTED | SKIP | 13f |
| SYS-108 | Split-flap status chips | DOCUMENTED | LATER | 13f |
| SYS-109 | Narrow-window HUD reflow | DOCUMENTED | KEEP | 13f |
| SYS-120 | Detection overlay | DOCUMENTED | KEEP | 13f |
| SYS-121 | Modes and density | DOCUMENTED | KEEP | 13f |
| SYS-122 | Allocation: ELASTIC vs WEIGHTED | DOCUMENTED | KEEP | 13f |
| SYS-123 | Fade and OUTSIDE opacity | DOCUMENTED | KEEP | 13f |
| SYS-124 | Label arbiter | DOCUMENTED | KEEP | 13f |
| SYS-125 | Per-object drawing | DOCUMENTED | KEEP | 13f |
| SYS-126 | Tracked readout callout | DOCUMENTED | KEEP | 13f |
| SYS-127 | Zero idle cost | THIN | KEEP | 13f |
| SYS-128 | Detection mode banner | DOCUMENTED | SKIP | 13f |
| SYS-140 | Scope (circular keyhole viewport) | DOCUMENTED | KEEP | 13f |
| SYS-141 | Feather | DOCUMENTED | KEEP | 13f |
| SYS-142 | Altitude-adaptive edge terminus | DOCUMENTED | KEEP | 13f |
| SYS-143 | Celestial ring | DOCUMENTED | LATER | 13f |
| SYS-160 | Seven looks | DOCUMENTED | KEEP | 13f |
| SYS-161 | CRT | DOCUMENTED | LATER | 13f |
| SYS-162 | NVG | DOCUMENTED | LATER | 13f |
| SYS-163 | FLIR | DOCUMENTED | LATER | 13f |
| SYS-164 | Anime / Noir / Snow | DOCUMENTED | SKIP | 13f |
| SYS-165 | Starting params | DOCUMENTED | KEEP | 13f |
| SYS-166 | Look side effects | DOCUMENTED | KEEP | 13f |
| SYS-167 | Bloom | DOCUMENTED | KEEP | 13f |
| SYS-168 | Sharpen | DOCUMENTED | KEEP | 13f |
| SYS-169 | Idle render governor | DOCUMENTED | KEEP | 13f |
| SYS-170 | Frame-rate readout | DOCUMENTED | KEEP | 13f |
| SYS-171 | Photoreal horizon blend | DOCUMENTED | KEEP | 13f |
| SYS-172 | Model atmosphere workaround on Apple Metal | THIN | KEEP | 13f |
| SYS-180 | First-run mission launcher | DOCUMENTED | ADAPT | 13f |
| SYS-181 | POWER UP / Provider Settings | DOCUMENTED | ADAPT | 13f |
| SYS-182 | Setup doctor | DOCUMENTED | KEEP | 13f |
| SYS-183 | Credits and attribution | DOCUMENTED | KEEP | 13f |
| SYS-184 | Embed mode | DOCUMENTED | KEEP | 13f |
| SYS-185 | Framing policy | DOCUMENTED | KEEP | 13f |
| SYS-186 | Loading and status notices | DOCUMENTED | KEEP | 13f |
| SYS-187 | Honest feed states | DOCUMENTED | KEEP | 13f |
| SYS-188 | Layer row key messages | DOCUMENTED | KEEP | 13f |
| SYS-189 | Location mini-status | DOCUMENTED | KEEP | 13f |
| SYS-190 | Keyless search | DOCUMENTED | ADAPT | 13f |
| SYS-191 | Cost metering beyond voice | DOCUMENTED | KEEP | 13f |
| SYS-200 | Local broker/proxy | DOCUMENTED | ADAPT | 13f |
| SYS-201 | Request gating | DOCUMENTED | KEEP | 13f |
| SYS-202 | Content security policy | DOCUMENTED | KEEP | 13f |
| SYS-203 | Caching and staleness | DOCUMENTED | KEEP | 13f |
| SYS-204 | Tool-key hygiene | DOCUMENTED | KEEP | 13f |
| SYS-220 | Unit and contract tests | DOCUMENTED | KEEP | 13f |
| SYS-221 | Tracking regression suite | DOCUMENTED | KEEP | 13f |
| SYS-222 | Headless browser QA harnesses (97 scripts) | DOCUMENTED | KEEP | 13f |
| SYS-223 | Recorded-audio voice checks | DOCUMENTED | KEEP | 13f |
| SYS-224 | Boundary and token checkers | DOCUMENTED | KEEP | 13f |
| SYS-225 | CI | DOCUMENTED | KEEP | 13f |
| SYS-226 | Manual field-test script | DOCUMENTED | LATER | 13f |
| SYS-227 | Offline imagery tools | MISSING | SKIP | 13f |
| SYS-228 | Reviewer workflow | DOCUMENTED | SKIP | 13f |
| SYS-240 | Vite build and dev/preview | DOCUMENTED | ADAPT | 13f |
| SYS-241 | Panel build | DOCUMENTED | LATER | 13f |
| SYS-242 | Launcher scripts | DOCUMENTED | SKIP | 13f |
| SYS-243 | One-click installer (Pinokio) | DOCUMENTED | ? | 13f |
| SYS-244 | Preview vs production | DOCUMENTED | KEEP | 13f |
| SYS-245 | Node/dependency baseline | DOCUMENTED | KEEP | 13f |
| SYS-246 | Local receivers | DOCUMENTED | ? | 13f |
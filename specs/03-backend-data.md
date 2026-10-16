# 03 - Backend and Data-Source Architecture (clean-room study notes)

Scope (revised against reverse-engineering docs 10 and 11): a read-only study of the reference web app's server layer, security documentation, data-source documentation, environment template and bundled configuration packs. Everything below is paraphrased description of intent and behaviour. No code, identifiers, routes or comments are reproduced. Licence statements are reported as the reference docs state them; they are not independently verified.

---

## 1. Overall architecture

The reference product is a browser-based 3D globe that overlays many live public feeds (aircraft, ships, satellites, traffic, transit, cameras, weather, fires, street-level imagery, radio, news). It is "local-first": one development or preview web server serves the front-end bundle and, in the same process, hosts a family of thin server-side adapters, one per upstream provider.

**Browser responsibilities.** Rendering the globe, interpolating movement between updates, drawing layers, showing attribution, and talking only to its own origin for anything that needs a credential, a cache or protection of a third party's quota. A small number of keys are deliberately handed to the browser because the vendor designs them for client use (a maps key, a globe-streaming token, a street-imagery client token). These are expected to be restricted at the vendor console by referrer, allowed API list or URL scope. The build step uses an explicit allowlist of what is injected into the client bundle, so nothing else can leak by accident.

**Server responsibilities (the "key broker" role).**
1. Hold every secret-bearing credential (voice/LLM key, ship-feed key, flight OAuth client secret, fire-data key, traffic key, server-side maps key) and never forward it to the browser. Where the browser needs something derived from a secret, the server mints a short-lived, narrowly scoped token instead.
2. Maintain long-lived upstream connections the browser should not own, for example a single persistent socket to the ship-position feed whose contents are kept in a server cache and exposed to the browser as a simple polled snapshot.
3. Normalise heterogeneous upstream formats into a small number of uniform JSON shapes: binary transit protocol buffers decoded server-side, binary gridded weather files read by byte range and resampled to a coarse display grid, ArcGIS/Socrata/GeoJSON/DATEX camera catalogues unified into one camera record shape, and so on.
4. Protect upstream providers and the operator's quota: cache, coalesce identical in-flight requests, rate-limit per client, back off after failures, and serve stale data during outages.
5. Enforce a request-gating policy so the server cannot be turned into an open relay (see section 2).
6. Present a unified "source health" story: each feed reports whether it is live, degraded, stale, empty or unavailable, and these states stay distinct in the UI.

**Why keys stay server-side.** The server is described as a spendable-quota broker: whoever can reach it can consume the operator's paid or rate-limited allowances. Putting secrets in the browser would let any visitor extract and abuse them, so the browser only ever sees same-origin endpoints. Credentials can come from process environment, a git-ignored local environment file, the operating-system keychain through the launcher, or a local settings panel that writes the ignored file with owner-only permissions and restarts the server. Credentials set externally are shown as "configured elsewhere" and never overwritten. The settings registry holds ten provider entries; nine appear in the panel and the tenth (the separate server-side maps key) is hidden advanced configuration, not counted as missing. Only three keys are client-visible by design (maps, globe-streaming token, street-imagery token). Detail: reverse-engineering 11 sections 2-3.

**Graceful degradation as a design principle.** Every keyed feature has a keyless fallback or an explicit "key required / empty" state: satellite basemap falls back to a keyless imagery service then to open map tiles; flight data falls back from the primary community network to a second community aggregator; traffic falls back to a built-in simulation; place search falls from the commercial geocoder to two open geocoders; news falls from one provider to another. The app starts with zero keys.

**Secondary surfaces.** The same tool set is exposed to external assistant clients over a local stdio channel and a local-only HTTP endpoint, with an embeddable "panel" build. These are loopback-only transports, not authenticated APIs.

**Operator-supplied configuration packs.** Camera sources are assembled from a registry of "packs" (one per city or road agency), each individually switchable and capped by count, plus small curated JSON packs for municipal webcams (several cities in Europe, Japan and the US are shipped). A catalogue-wide ceiling shares capacity round-robin across packs. Optional inline or file-based overrides let an operator replace the whole set.

---

## 2. Security model

### 2.1 Request gating
- **Local-only by default.** The server binds to the loopback interface. Exposure to the LAN is an explicit opt-in with a loud launcher warning, and it still keeps the host check below.
- **Host-header allowlist applied before any route.** A dedicated early check rejects unexpected host names in every mode, which blunts DNS-rebinding. When bound to all interfaces it accepts IP addresses, the loopback name and names the operator explicitly listed; wildcard and suffix entries are ignored by design. Listed names are trusted like the app itself.
- **Cross-site gate on only some endpoints**: the voice session credential, the short HUD summary, the sign-in status and start, the voice debug-log sink, the two Google places lookups, and the street-imagery status and coverage tiles. Requests are refused when the Origin is foreign or opaque, when the fetch-site metadata says anything other than same-origin or none (this catches images and navigations that carry no Origin), or when reverse-proxy/CDN forwarding headers are present. Non-browser local tools without an Origin still work. This is defence in depth behind the host check. Flights, ships, traffic tiles, fires, routing, geocoding, cameras, transit and the other proxies are NOT gated: a hostile page in the user's browser can make the local server spend quota (it cannot read the replies without CORS), so the loopback bind and host check are the only protection there. A rebuild should gate every quota-bearing endpoint. Detail: reverse-engineering 10 section 4.
- **Stricter loopback-only admission** for the credential-setting panel and for the ChatGPT-sign-in-based voice path (status, login and token endpoints answer only loopback connections; none returns the access token).
- **Assistant-protocol endpoint** answers only direct local requests: loopback connection naming a loopback host and its own port, matching Origin if any, no forwarding headers, sharing disabled, JSON bodies only. The docs call this transport safety, not authentication. The panel relay is refused access to settings, credential, model and internal routes.
- **Framing control.** Pages cannot be framed by default; embed mode can be opened to specific ancestors by configuration. A full content-security policy is sent, with documented, accepted weaknesses (an eval allowance required by a bundled UI library; a trusted maps script host that has legacy callback endpoints).
- **Launcher sharing refused.** The packaged-app launcher forces its tunnel/LAN sharing controls off so the key broker cannot be accidentally published.

### 2.2 Host and destination checks (SSRF defence)
- Clients almost never supply an upstream URL. Camera frames, video streams, transit feeds and local receiver feeds are resolved from server-side registries by identifier only; other proxies target fixed hosts, and the satellite group name and ship id are validated strings placed into fixed addresses. Two narrow exceptions: the bike-share relay accepts a full https address from the browser, but only for an allow-listed set of operator hosts (five named operators plus subdomains of one hosted bike-system domain) and only for the two station documents (information and status), refusing redirects and capping the body at 5 MB; and the detailed-map-feature relay accepts a query body, but only against operator-configured upstream servers with size, area and selector limits.
- Redirects are refused or followed manually with each hop validated against the registered origin (no downgrade to plain http, no off-origin hop).
- The radio-directory proxy resolves names itself, rejects any answer in private, loopback, link-local, metadata or other non-global ranges (v4 and v6), allowlists directory hosts and paths, and pins the TLS connection to the validated address. It returns only public HTTPS stream targets; the browser then connects directly to the broadcaster after an explicit user action, so the server never relays audio and the broadcaster sees the listener's address.
- Local ADS-B receiver feeds accept only operator-configured addresses limited to loopback, private ranges and local names, requiring a specific file-name ending and refusing credentials, queries and fragments. Invalid entries are logged at start-up and never fetched.
- Live video (HLS) relays are origin-pinned, reject off-origin playlist references, bound playlist and segment sizes, hold only a few sessions in memory, expire idle sessions in seconds, and never write to disk. Direct RTMP or external transcoder execution is not enabled.
- Request headers are not passed upstream verbatim. The only forwarded header is a canonicalised byte-range for video seeking: a single range only, bounded to a fixed ceiling; malformed or multi ranges are dropped. Abandoned client requests cancel the upstream call.

### 2.3 Rate limits and quota governors
- Per-client-IP per-minute throttles on cost-bearing endpoints: a stricter default for the AI/voice endpoints (tens per minute) and a looser one for maps places lookups (about a hundred per minute). Setting zero disables; an unparseable value falls back to the default rather than to unlimited. Over-limit responses are sanitised 429s. The docs are explicit that these are in-memory, per-process, per-IP guards, not billing caps; real spend protection must be set in each vendor's budget and quota consoles.
- Tile endpoints for the street-imagery provider have a separate per-IP limit.
- The routing proxy limits per client (60 a minute) and in total (200 a minute), spaces upstream calls one second apart per fair-use policy, caps route span and legs, and coalesces identical requests.
- A daily upstream-tile budget governor on the paid-freemium traffic provider is set so a full month of use fits under the provider's free quota; over budget it serves cached or stale tiles instead of hitting upstream.
- Transit has a per-feed admission limiter plus a failure cooldown ladder. Flight, geocoding and enrichment proxies use small concurrency, drip dispatch and exponential-style backoff (rate-limit backoff capped around an hour).
- Free community APIs without published quotas (aircraft-metadata lookup) get deliberately tiny fan-out and once-per-key-per-session asks.

### 2.4 Caching and staleness
- Cache lifetimes are chosen per source volatility and per upstream etiquette: seconds for transit snapshots and local receiver reads; about a minute for navigation-integrity samples; a couple of minutes for traffic tiles, routes (ten minutes) and weather overlays; five minutes for storm advisories, fire perimeters and regional weather; 15 minutes for camera catalogues and launch data; 30 minutes for fire hotspots; 45 minutes for the radio directory; an hour for gridded wind model fields and vector detail tiles; six hours for satellite elements; a day for street-imagery coverage tiles and aircraft-metadata lookups (including negative answers); and exact-time radar/satellite images up to 24 hours.
- Serve-stale-on-failure is common: last good radio catalogue up to a week, last weather observation up to half an hour, last launch response during throttles.
- Only successful responses are cached (flight OAuth path). OAuth token refreshes are coalesced to a single in-flight refresh.
- Caches are memory or git-ignored local disk. Camera imagery is never written to disk. Third-party content whose terms prohibit storage (the commercial maps content) is used live only and never cached or committed. A short transit position history is held in memory only and cleared on restart.

### 2.5 Response caps and timeouts
- Bounded bodies everywhere: single-digit megabytes for transit protobufs, route responses and local receiver JSON; a larger ceiling for relayed video ranges; row caps on the ship feed (large server cache, smaller client render cap with a label cap) and on each camera pack; paged-feature-service transfer limits honoured.
- Short upstream timeouts (a couple of seconds for local receivers, ~12 s for routing). Entity-count and string-length ceilings when decoding transit protobufs; differential feeds refused.
- Viewport-bounded reads for vector-tile-derived layers (maximum tiles per country, concurrent reads, displayed record cap).

### 2.6 Redaction and error hygiene
- Errors returned to clients are sanitised: internal details and upstream error text are not echoed. Local receiver responses identify feeds by band only.
- The voice debug log scrubs credentials and embedded images; by default it also omits transcripts, typed/spoken text and tool arguments/results, with an opt-in switch for debugging. It is rotated by size but not expired, so the docs advise deleting it afterwards.
- The navigation-integrity proxy strips aircraft identity metadata and returns only integrity facts.
- Provider-credential files are never committed; the environment template holds names and placeholders only. Local credential storage is plaintext but owner-restricted; the OS keychain is called the stronger option.

### 2.7 Privacy, telemetry and testing
- No analytics, crash reporting, advertising or product telemetry was found in code, build scripts, markup, CI or the content-security policy. Residual outbound metadata: identifying user agents and referers sent to community services as their policies require; the visitor's address reaches every browser-direct provider (tiles, quake feed, street imagery, the font host loaded at page start), radio broadcasters and embed hosts. Self-host fonts in Kawn. Detail: reverse-engineering 11 section 6.
- Unit testing is large: roughly 562 test files and about 6,400 test declarations by static count (the reference's own older figure of about 2,250 is stale). CI gates every push: policy doctor, formatting, package-boundary and import-direction checks, full unit suite, production build, and a fixture-driven street-level browser gate; about 100 browser acceptance scripts exist but are not run in CI. Detail: reverse-engineering 11 section 9.

### 2.8 Other stated limitations and ethics
- The voice agent is told to act only through a fixed set of app-control tools and to ignore instructions embedded in feed text; model output is treated as untrusted.
- Camera frames are relayed unaltered; no face, plate or object recognition, no named-person search, no tracking of individuals. Plate-reader cameras are mapped only from community-tagged locations.
- Any program on the same machine can use the local endpoints; a local reverse proxy that hides client address defeats loopback checks.
- The product is a dev/preview-grade server; exposure beyond localhost needs a separately reviewed authentication proxy.

---

## 3. Upstream data sources

"Keyed" means a credential is needed for full function; "browser" in the key column means the credential is intentionally client-visible. Terms are as stated in the reference docs; where a doc says none is published, that is recorded.

| Source | What it provides | Keyed? | Cost model | Stated terms, licence, redistribution limits |
|---|---|---|---|---|
| OpenSky Network | Primary worldwide live aircraft snapshot | Optional (OAuth client credentials; anonymous works rate-limited) | Free tier, rate-limited | Non-commercial research/education licence. Operational use of the API in a live product may need prior written agreement even for non-profit/government. Citation of the founding paper requested. |
| adsb.lol (point API and military/traces) | Fallback regional flights (250 nm circle), military flights, traces, aircraft-reported navigation-integrity data | Keyless (future feeder keys possible) | Free community | ODbL 1.0; attribution to contributors; share-alike on derived databases. |
| adsbdb | Aircraft type/registration by hex; airline and route for a callsign | Keyless | Free, no published quota | No licence published. Route data is credited to two individuals and, per its own notice, may not be copied, published or put into other databases without permission; deployers must obtain permission themselves. |
| Local ADS-B decoder / SDR | The operator's own receiver feed | Local only | Own hardware | Not a service; data not stored or redistributed; ADS-B may be incomplete, delayed or spoofed. |
| AISStream | Live ship positions and static data over a persistent socket | Keyed (server-side) | Free, beta | No formal terms; AIS is a public broadcast; courtesy credit. Supports bounding-box and message-type filters and a feed-silence watchdog. |
| CelesTrak | Satellite orbital elements | Keyless | Free | US-government-origin; no licence; citation requested. |
| Launch Library 2 (Space Devs) | Launch, payload, stage and recovery metadata | Optional token | Free; 15 calls/hour anonymous | Data may be used and shared in any form; avoid forwarding without added value; attribution encouraged, not mandatory; no accuracy guarantee. |
| Google Maps Platform (3D tiles, geocoding, places, street view) | Photorealistic 3D globe, place search, nearby places, panoramas | Keyed (one browser key, optional separate server key) | Pay-per-use with billing | Proprietary ToS; content must not be cached, stored, rehosted or committed; attribution/logo mandatory; every panorama load billed. |
| Cesium ion | Hosted 3D tiles, world imagery, terrain | Keyed (public token, browser) | Free community tier with eligibility and quotas | Community plan for eligible non-commercial use; check current terms. |
| Esri World Imagery | Keyless satellite basemap | Keyless | Free | Usable in public apps with attribution; Esri may change access; review location-platform terms at scale. |
| OpenStreetMap, OpenFreeMap/OpenMapTiles, Overture | Map tiles, place data, military/area names, datacentres, dams, ALPR locations | Keyless | Free | ODbL 1.0; attribution and database share-alike; code is not licensed by it. |
| Photon (Komoot) | Keyless place-search fallback | Keyless | Free public instance | Fair use, no bulk; data ODbL. |
| Nominatim | Last-resort geocoding, forward search only on the server (reverse lookups go to the commercial geocoder) | Keyless | Free | At most one request per second, identifying user agent, no bulk; ODbL. |
| OSRM on community routing servers | Car/bike/foot routing | Keyless | Free | Attribution plus "fix the map" link, valid user agent, one request per second, no scraping or heavy use; commercial use only with restrictions; ODbL data. |
| Mapillary | Street-level imagery and coverage | Keyed (client token, browser) | Free token | Imagery CC BY-SA 4.0; API under Mapillary terms. |
| KartaView | Street-level imagery and coverage | Keyless | Free | CC BY-SA 4.0. |
| TomTom Traffic | Live congestion vector tiles | Keyed (server-side) | Freemium, about 200k tiles per month | Proprietary developer terms; cached transiently only; credit shown when live. |
| Transit realtime feeds (seven regions: Boston, Austin, Minneapolis-St Paul, Netherlands, Norway, South-East Queensland, Helsinki) | Live vehicle positions via GTFS-Realtime | Keyless | Free | Operator open data, "published for developer use"; courtesy credit; some require identifying headers. |
| GBFS bikeshare | Station availability | Keyless | Free | Per-feed, attribution-only. |
| Traffic-camera packs (Austin, Texas DOT, California, Delaware, London, Ontario, British Columbia, Calgary, NSW, Finland, Norway, Tallinn, Estonia road, Warendorf; New York off by default) | Camera catalogues, still frames, some live HLS | Keyless (optional app key for London) | Free | Mostly open-government licences requiring attribution (CC BY 4.0, Canadian provincial/city open licences, Norwegian NLOD 2.0, TfL attribution); some "public data" with courtesy credit; New York needs a signed data-sharing agreement so it is off unless the operator holds one; Delaware notes public availability is not a blanket redistribution licence. Frames fetched on demand, never stored. |
| NOAA GFS | Global wind, temperature, pressure grids (0.25 degree source, resampled to about 1 degree for display) | Keyless | Free | US public domain. |
| ECMWF open data | Alternative wind model | Keyless | Free | CC BY 4.0 plus ECMWF terms; must show attribution, modification notice and disclaimer. |
| NOAA nowCOAST (radar, satellite, lightning) | Observed rain radar (US), satellite IR, lightning density (Americas/Pacific) | Keyless | Free | NOAA disclaimer; lightning is a derived product from a commercial detection network and only the density product may be distributed. |
| NOAA NHC/CPHC | Tropical cyclone advisories | Keyless | Free | NWS public data terms; no endorsement. |
| NASA FIRMS | Active fire hotspots (VIIRS, MODIS) | Keyed (free map key, server-side) | Free with shared transaction quota | Public domain; acknowledgement text required. |
| NASA GIBS and CMR | Recent satellite imagery layers and granule catalogue (called by the browser directly) | Keyless | Free | US public domain; acknowledgement text; no bulk use. |
| NIFC wildfire perimeters and InciWeb | US wildfire perimeters and incident links | Keyless | Free | US public domain. |
| USGS | Earthquakes (magnitude 2.5+, 24 h; browser-direct) | Keyless | Free | US public domain. |
| Community plate-reader extract (OSM-derived, US and Canada) | Mapped plate-reader camera locations | Keyless | Free | ODbL via OpenStreetMap; locations only, no plate or video data; not relevant to the UAE. |
| Google Fonts | Interface typefaces (browser-direct) | Keyless | Free | Terms not recorded in the reference ledger; leaks visitor address at page load; self-host. |
| Open-Meteo | Current weather at 0.1 degree cells for atmospheric effects and the regional briefing | Keyless | Free | CC BY 4.0; adjacent linked attribution required. |
| Google News RSS | Locality headlines | Keyless | Free | Personal, non-commercial use only; commercial deployments must replace or get permission. |
| GDELT DOC | Fallback headlines | Keyless | Free | Commercial use allowed with citation and link. |
| Radio Browser | Geolocated internet radio directory | Keyless | Free | Public-domain directory data (PDDL); individual stream terms apply; no audio relayed. |
| Re:Earth/Mapterhorn terrain | Keyless terrain mesh and height lookups | Keyless | Free | CC BY 4.0 mesh; public-domain geoid. |
| OpenAI (voice realtime, short summaries, transcription) | Voice control and HUD summaries | Keyed (server-side, ephemeral tokens to browser) | Pay-per-use; session cost meter and cap | Vendor terms; optional experimental reuse of a local ChatGPT/Codex sign-in, which the docs warn the vendor has not documented. |
| Bundled snapshots | Submarine cables, datacentres, dams, military names, country/region polygons, neighbourhoods, US counties, camera ground heights | None | Shipped in repo | Submarine cables are CC BY-NC-SA, non-commercial, must be deleted for commercial use; OSM-derived sets are ODbL; Natural Earth, Census and city open data are public domain/PDDL. A separate event pack is CC BY-NC. |

**Weather sources and coverage (important for Kawn).** Wind comes from GFS or ECMWF forecast fields (default GFS), read by byte range and refreshed hourly; the 1 degree display grid is too coarse for a city, so plan finer resolution. The observed products are all NOAA nowCOAST: rain radar covers the contiguous US only; regional satellite cloud covers North America only; only the global infrared mosaic (about 3 km, hourly, 2 to 3 hours late) reaches the UAE; lightning density covers the Americas and Pacific only. Tropical-cyclone advisories cover the Atlantic and eastern/central Pacific, so Arabian Sea and Gulf of Oman storms never appear. Kawn therefore needs a local meteorological radar and nowcast source (National Center of Meteorology, subject to agreement) plus dust and fog products. Detail: reverse-engineering 04 sections 11-13 and 10 section 2.4.

Observations useful for the product (see the brand definition for its name): the reference keeps an explicit carve-out between its own code licence and every data licence, shows attribution in-app in an expandable credits panel, and exposes a kill switch per camera pack. Several sources it uses are non-commercial only (primary flight network, news RSS, cable map, one event pack) which is a direct warning for any commercial product. Kawn guidance: keep a per-source licence ledger with a redistribution flag, block non-commercial datasets (primary flight network, news RSS, cable map, event pack, unlicensed route data) from commercial builds, and keep every provider credential server-side with only restricted client keys exposed. All licence text is as the reference states it and needs legal review.

---

## 4. API contract (abstract capabilities)

The browser-facing surface is a set of same-origin, read-mostly JSON capabilities. Each returns uniform shapes, a data-source/health indication, and sanitised errors.

- **Flights:** worldwide snapshot with source provenance and fallback (base cache 9 s, stretched up to 300 s as the daily credit budget thins; regional community fallback within 250 nautical miles when the primary is empty or over two minutes old); aircraft enrichment by hardware identifier; per-aircraft track; military-flight set; navigation-integrity grid facts for a region; local receiver aircraft per band.
- **Vessels:** current ship snapshot from the server cache with staleness flag and count caps (stale after 120 s of feed silence, reconnect back-off from 5 s to 5 min; client renders up to 12,000 rows).
- **Space:** satellite element sets; recent launches.
- **Weather:** wind field by model and forecast time (with model issue time; one-hour cache, last good retained); observed radar/satellite/lightning manifests of up to 13 recent frames plus exact-time images (immutable for 24 h, metadata refreshed every 2 to 10 minutes, explicit stale state); cyclone advisories; point weather observations for atmospheric effects; regional briefing combining local conditions and headlines.
- **Hazards:** active fire hotspots (24 h window), fire perimeters and incident links, earthquakes.
- **Traffic and routing:** congestion tiles with budget governor; route between two to twelve coordinates for a travel mode with optional step detail.
- **Transit:** list of registered regions and their state; live vehicle snapshots per region; short recent-history for one region.
- **Bikeshare:** station availability, nearest first, via the allow-listed relay described in section 2.2.
- **Cameras:** catalogue with provenance, attribution and per-camera heading confidence; still-frame retrieval and range-limited media retrieval by camera identifier only; live video session handling with release.
- **Street level:** coverage tiles and imagery lookups for the enabled imagery provider; one provider active at a time.
- **Places and geocoding:** nearby-place search, text search, forward-only geocoding fallback (queued at one request a second); the place-search chain is offline names, commercial places, Photon, then Nominatim.
- **Radio:** bounded station directory with health status and a click-count acknowledgement by known identifier.
- **Terrain:** ground-height lookups for entity placement.
- **Voice/AI:** mint short-lived realtime session credential; periodic short HUD summary; debug-log sink; status/login for optional ChatGPT sign-in path.
- **Settings:** local-only credential panel to add, replace, remove provider keys; triggers restart.
- **Assistant protocol:** tool surface for external agent clients over local stdio and local HTTP, plus a panel relay for app files and data.

Fallback chains: flights primary then last good then regional aggregator; news RSS then GDELT; traffic then built-in simulation; routing then a labelled straight line; terrain heights then bundled geoid maths; camera frame then street still then placeholder. Everything keyed has a keyless or explicit key-required state.

Contract conventions: identifiers never URLs; bounded inputs; sanitised 429/403/404; each response indicates freshness and degraded state; unknown capability paths return a structured not-found rather than the front-end page.

---

## 5. UAE equivalents (candidates; all "verify licence" - none confirmed online)

| Category | Candidate sources | Notes / what to verify |
|---|---|---|
| Flights around DXB, DWC, AUH, SHJ | OpenSky (bounded box query); adsb.lol point API; ADS-B Exchange style community aggregators; own ADS-B receivers (the Gulf has dense feeder coverage); airport authority flight-information pages for schedules. Verify licence. | Non-commercial restrictions on OpenSky are the main risk. Official schedule data (Dubai Airports, Abu Dhabi Airports) likely needs agreements. UAE airspace sensitivities and military traffic filtering should be reviewed. Verify licence. |
| AIS in the Arabian Gulf and Strait of Hormuz | AISStream with a bounding box over the Gulf; commercial AIS vendors; port authority data (Ports and Customs / Free Zone authority, Abu Dhabi Ports, DP World) if available. Verify licence. | Terrestrial coverage in the Gulf may be patchy; satellite AIS usually paid. Verify licence. |
| Transit and RTA | Dubai RTA open data and developer portal (bus, metro, tram, taxi, Salik, parking), Abu Dhabi Integrated Transport Centre feeds, Dubai Pulse. Check whether realtime vehicle feeds exist in a standard realtime transit format or only proprietary APIs. Verify licence. | Likely need registration or an agreement; realtime may be limited. Verify licence. |
| Open government data portals | Dubai Pulse / Dubai Data; Bayanat (UAE federal open data); Abu Dhabi Open Data portal; Federal Competitiveness and Statistics Centre; Sharjah and other emirate portals. Verify licence. | Check the UAE open data licence, whether commercial reuse and redistribution are permitted, Arabic/English metadata and API availability. Verify licence. |
| Real estate | Dubai Land Department open datasets (transactions, rents, projects), RERA data via Dubai Pulse, Abu Dhabi Department of Municipalities and Transport (DMT) / Tamm, Sharjah Real Estate Registration; commercial portals are scraping-prohibited. Verify licence. | Prefer official open data; avoid scraping listing sites. Personal data in ownership records must not be shown. Verify licence. |
| Weather and air quality | UAE National Center of Meteorology (NCM) public products; Open-Meteo (keyless, CC BY 4.0 per the reference docs) for forecasts and air quality; NOAA GFS and ECMWF open data for wind; satellite imagery from NASA GIBS; Environment Agency Abu Dhabi and Dubai Municipality air quality stations; community sensor networks. Verify licence. | US-only radar and Americas-only lightning mean the reference weather layers do not cover the UAE; the global cloud mosaic is the only carry-over. NCM radar/products may require agreement for redistribution. Dust and fog events matter in the region. Verify licence. |
| Construction and urban development | Dubai Municipality and Dubai Land Department project datasets; Abu Dhabi DMT permits; OpenStreetMap construction tags; satellite imagery (Sentinel-2/Landsat via NASA and Copernicus) for change detection; municipal 3D city models if released. Verify licence. | Official permit data may be limited; imagery-based inference should be labelled as inference. Verify licence. |
| Cameras and traffic | Dubai RTA and Abu Dhabi traffic camera feeds if published; TomTom or similar commercial traffic by key. Verify licence. | Public availability is not a redistribution licence; privacy rules in the UAE are strict about imagery of people and vehicles. Verify licence. |
| Base map, search, routing | OpenStreetMap-derived tiles and routing (self-hosted for scale), Google Maps Platform or alternatives, Photon/Nominatim fallbacks. Arabic place-name coverage should be tested. Verify licence. | Public routing servers forbid heavy use; plan to self-host. Verify licence. |
| Space, fires, quakes, radio | CelesTrak, NASA FIRMS (free key), USGS, Radio Browser filtered to the region. Verify licence. | These are global and carry over unchanged, subject to the same terms as in the table above. |

---

## Spec revision log
- Cross-site gate narrowed to the endpoints actually covered; all other proxies are ungated (doc 10 section 4, doc 11 section 7).
- Upstream-address rule corrected: the bike-share relay takes a browser-supplied, allow-listed address; map-feature relay is operator-upstream only (doc 10 sections 1, 2.5, 2.7).
- Key registry stated as ten entries, nine shown (doc 11 section 2); added privacy/telemetry (none found) and test-count (about 562 files) subsection (doc 11 sections 6, 9).
- Nominatim marked forward-only; added browser-direct sources, plate-reader extract and fonts host to the source list (doc 10 sections 2.7, 3).
- Filled weather sources and coverage gaps for the UAE, plus cadences, caps and fallback chains for server code (doc 10 section 2, doc 04 sections 11-13).
- Added Kawn licence-ledger and server-side-key guidance next to the non-commercial warning (doc 10 section 3, doc 11 section 7).

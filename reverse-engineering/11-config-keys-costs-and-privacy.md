# 11 - Configuration, Keys, Costs, Privacy, Security Posture, Install Modes and Testing (behavioural reverse-engineering)

**Purpose.** This document records how the reference product is configured and operated: which provider keys exist and what each one unlocks, which are optional and which are practically required for a given capability, where keys are entered and stored, how cost is metered and capped, every outbound network destination, what telemetry exists (none was found), the security posture, the ways the product is installed and run (including the one-click installer), and the testing strategy and quality gates. It is written for a clean-room team rebuilding an original UAE city-intelligence product. No code, file names, variable names or routes are reproduced; provider names, product names and numeric limits are. Everything is derived from reading documentation and source, not from running the app. Confidence tags: [OBSERVED-IN-SOURCE], [FROM-TESTS], [FROM-DOCS-ONLY], [INFERRED]; where a section carries no tag, it was cross-read between documentation and code and they agree.

Mechanism detail: 13f part 1 (sections 1G-1M, keys, platform, build, tests) and part 3; geocoder chain and token retry in 13a; backend limits in doc 10.

---

## 1. Zero-key baseline

The application **starts and is useful with no keys, accounts or payment**:
- Keyless satellite basemap (Esri World Imagery) with keyless terrain; plain OpenStreetMap tiles take over automatically if the satellite service is unreachable; the globe continues without terrain if terrain is unavailable.
- Working without keys: flights (anonymous, rate-limited, with a regional community fallback), military flights, satellites, earthquakes, public cameras, radio, launches, transit, bike share, directions routing, wind, observed radar/clouds/lightning, cyclone advisories, mapped plate-reader cameras, mapped installations, fire perimeters, bundled datasets, coordinate and bundled-place search, street-level imagery from the keyless provider, recent imagery, weather and regional briefs, simulated traffic.
- "Eighteen of twenty layers have a keyless path"; a layer row waiting on a key names it and shows a key-required state.
- First-run panel offers four starting missions (Live Contacts, Space Missions, Environmental, Explore Manually).

---

## 2. Key matrix (what each unlocks)

The in-app settings registry holds ten provider entries, of which nine are shown in the panel and one (the separate server-side maps key) is hidden advanced configuration that is not counted in the missing-key tally. [OBSERVED-IN-SOURCE] Tier legend from the product: free (register, paste), metered (billing-enabled account).

| Provider | Needed for | Required? | Tier | Exposure | Notes |
|---|---|---|---|---|---|
| Google Maps (browser key) | Photorealistic 3D globe direct from Google; Google place search; Google Street View panoramas; 3D scene context nearby-places | Optional (3D can instead come through Cesium ion; place search then falls to free geocoders) | Metered | **Client-visible by design** - must be restricted by HTTP referrer and API allow-list | Billing must be enabled; first ~1,000 3D tile sessions per month free at the time of writing; each panorama load billed |
| Google Maps (server key) | Server-side nearby-places, text-search recovery, Street View fallback frames | Optional, hidden advanced entry; falls back to the browser key | Metered | Server-only | Lets operators restrict by server IP and to just two APIs |
| Cesium ion token | Ion-hosted Google 3D, Bing aerial and aerial-with-labels stacks, world terrain | Optional | Free community tier (eligible personal/non-commercial) | Client-visible; should be a public read-assets token with URL restrictions | Quotas apply |
| OpenAI | Voice assistant (API key) and the five-word HUD summary | Optional (voice needs it or the experimental ChatGPT-login mode) | Metered | Server-only; browser gets one-time client secret | See cost section |
| AISStream | Live ships | Optional (layer reads unavailable without it) | Free | Server-only | |
| NASA FIRMS | Live fires | Optional (fires layer empty/"key required" without it) | Free, shared quota | Server-only | |
| TomTom | Real traffic flow/congestion on the simulated-traffic layer | Optional (simulation without it) | Free tier 200,000 tiles/month | Server-only | |
| OpenSky OAuth client id + secret | More flight-polling credits | Optional (anonymous works) | Free | Server-only | Basic and credentials-file modes are advanced launcher-only configuration |
| Mapillary client token | Street-level coverage and photos (default provider) | Optional ("KEY REQUIRED" state without) | Free | Client-visible by design | |
| Launch Library 2 token | Higher request allowance for Space Missions | Optional | Free | Server-only | 15 calls/hour anonymous |

Capabilities that need **no key at all**: wind, rain radar, satellite clouds, lightning, cyclone advisories, mapped plate-reader cameras, transit, directions, Delaware live video, coordinate and bundled-place search, KartaView street imagery, radio, earthquakes, launches (reduced allowance), satellites, community flights.

Other (non-registry) operator settings: voice model overrides (standard/mini ids, voice, reasoning effort, context size, retention, caption model, HUD-summary model), a content-logging switch for the voice debug log, per-IP throttle values, bind host and port, additional trusted host names, allowed embed-framing ancestors, ship-feed filters/silence watchdog/render caps, local-receiver feed addresses, traffic daily tile budget, camera pack enable switches and caps, optional operator-run detailed-map (Overpass) upstreams, a "forbid API-key voice" flag, and the location of the Codex credential file for ChatGPT-login. All are documented in a names-and-placeholders template that holds no secrets.

---

## 3. Where keys are entered and stored

### 3.1 In-app Provider Settings (the "POWER UP" panel)
- A chip in the bottom-right corner reads POWER UP with the count of keys waiting while any supported key is missing. **Correction (13f 3.2):** when every visible key is set the code retires (hides) the chip; "POWERED UP" is only the label if it is shown (for example when the panel is reopened), and the reference README line saying otherwise should not be copied. If a compact layout hides the chip, a query switch on the address reopens the panel. The dialog renders entirely from a server status listing (one row per key: what it unlocks, where to get it, a paste field).
- The panel lists each provider with a tier badge, what it unlocks, a link to get the key and a field. Actions: add, replace, remove; **SAVE KEYS** writes and restarts the server; the page then reloads itself with the capability on.
- Validation: unknown names refused; max 16 values per save; each value up to 512 characters; payload JSON only.
- **Externally configured values** (from the shell environment, a launcher keychain import, or similar) show as "configured externally" and are read-only; the panel never overwrites them. Provenance is captured at process boot, before any file is merged in, so the panel's own freshly written keys are not later misclassified as external, and a stray launcher marker in a file cannot redirect writes.
- Only the browser-safe ("client-exposed") providers get re-injected into the browser bundle on restart; the panel shows them with a restrict-your-key reminder.
- **Gate** (all must hold): direct loopback socket; local Host header; exact same Origin on writes; JSON content type (forces a CORS preflight the server never answers, closing simple-request forgery); no proxy/CDN forwarding headers; launcher sharing/LAN tunnel off. Responses are no-store and unframeable. Disabled for remote users and absent in production builds.
- Store choice: a Pinokio-managed launch writes its app-scoped environment file; any other launch writes the repository-root environment file. The file is created/locked to owner-only permissions **before** a secret is written; writes are atomic (temporary file then replace); an unreadable existing store aborts the save rather than risk wiping other keys; UTF-16 files written by Windows editors are decoded correctly and rewritten as UTF-8; on Windows the owner is established through the current user's security identifier.
- Credentials are **plaintext at rest**, owner-restricted, git-ignored. The operating-system keychain (macOS) is described as the stronger option.

### 3.2 Other ways
| Route | Behaviour |
|---|---|
| Process environment variables | Highest precedence; shown as external/read-only. |
| Local environment file (terminal clone) | Read by the build tool's loader; template shipped with names and placeholders only. |
| macOS Keychain via the recommended launcher | Precedence uniform: explicit shell variable, then environment file, then Keychain item. An empty string counts as "not set" on both sides; empty values are removed from the child environment so they cannot shadow a configured key. Keychain items documented for Google, OpenAI, AISStream, FIRMS, Cesium ion, TomTom, OpenSky. |
| OpenSky credentials import | A helper imports a downloaded credentials JSON (either naming style) into the keychain, or a path can be passed at launch. Resolution order: environment pair, credentials file, keychain. |
| Pinokio's native configure form | **Do not use** on the affected release (8.0.40 per the census): it saves the nested file to the wrong place and logs submitted values. Use the in-app panel instead. |

### 3.3 Client-visible key controls
Only three keys reach the browser (maps, 3D token, street-imagery token) through an explicit allow-list at build time; the panel build used inside AI assistants carries the same two map keys and can be built with separately restricted ones. A key restricted by HTTP referrer must also allow the assistant host's page.

---

## 4. Cost and metering behaviour

| Item | Behaviour |
|---|---|
| **Voice (API key)** | Live "~$" readout, STD/MINI tier toggle (next session), soft warning at $2 (amber + one log line), hard cap at $5 (session ends, no new tool dispatch, running tools finish). Estimates err high; incomplete estimates carry an asterisk. Caption transcription is metered too. Details: doc 08 section 7. |
| **Voice (ChatGPT-login)** | Counts only, "COST UNKNOWN", no cap. |
| **HUD summary** | Tiny text model call (a nano-class OpenAI text model by default) every 15 s while HUD visible and view changes; not metered in the UI. Throttled at 30/min/IP. Default voice models: a second-generation OpenAI realtime model (standard), a mini realtime tier, a small transcription model for captions, voice named Marin, low reasoning effort, 3,000-token context window with 0.5 retention. [FROM-DOCS-ONLY: model names drift] |
| **Google 3D / places / Street View** | Pay-per-use; documents advise budget alerts and per-API quotas; per-IP guard on place lookups is 120/min. Street View chip starts off because each panorama is billed. |
| **Cesium ion** | Free community quota; "Google 3D through ion" is free within quota for eligible use. |
| **OpenSky** | Credit-based; the server stretches its cache as the daily credit budget thins so a full day of continuous use does not exhaust it; honours the provider's retry-after. |
| **TomTom** | Soft daily cap on upstream tile fetches (6,000 by default, derived from 200,000/month); over the cap serves cached/stale tiles; status shows count vs budget; cache hits cost nothing. |
| **FIRMS** | 30-minute cache, single-flight, sequential fetch to respect the 5,000-transactions/10-minute key quota. |
| **Launch Library** | 15-minute cache; anonymous 15/hour. |
| **Everything else** | $0 (community services); fair-use rules (1 req/s for geocoding and routing). |
| **Other metering** (13f SYS-191) | Google 3D session billing note, the OpenSky credit governor, the TomTom daily tile budget and provider quotas are surfaced as status chips; element sets are disk-cached. The Google access token for 3D tiles is short-lived: renewed one minute early, one in-flight renewal shared, one retry per refused tile request (doc 10 section 2.7). |
| **Benchmark spend** (developer only) | The voice benchmark has a total cloud-spend budget flag and uses the app's own rate table; an unknown OpenAI model id resolves to the priciest known table; Gemini rates carry a verified-on date; unattributed tokens bill as audio (the conservative direction for a cap). Pricing tables are dated external facts (2026-08-18 and 2026-09-23 in the source): re-verify before any Kawn costing. See section 9.6 |
| **Throttle semantics** | Per-IP, in-memory, per-process; reset on restart; a caller with many addresses gets a bucket each; **not billing caps**. The documentation insists real protection is the vendor-side budgets and usage limits. |

The product's own cost stance: "most layers $0; free-key tier $0 with signup; Google 3D free through ion within quota or metered direct; OpenAI voice is the only routinely metered item". Operators are told that sharing the server on a LAN exposes spendable quotas to everyone on the network.

---

## 5. Outbound network destinations

### 5.1 From the local server (never the browser)
| Group | Destinations (by provider) |
|---|---|
| Aviation | OpenSky Network (identity-provider token host + global states + tracks), adsb.lol point/military/trace API, adsbdb |
| Maritime | AISStream websocket |
| Space | CelesTrak, Launch Library 2 (The Space Devs) |
| Hazards/weather | NASA FIRMS, NIFC WFIGS ArcGIS service, InciWeb, NOAA nowCOAST WMS, NHC current storms + NOAA map services, NOAA GFS (AWS open data), ECMWF open data, Open-Meteo, Google News RSS, GDELT |
| Mobility | TomTom traffic, FOSSGIS OSRM, GBFS operator feeds (Lyft, Bluebikes, BCycle, Biketown, Cogo, Public Bike System tenants; browser names the address but the server only accepts that allow-list), seven transit operators' GTFS-Realtime feeds (MBTA, Texas open-data portal for CapMetro, Metro Transit, OVapi, Entur, TransLink, HSL) |
| Cameras | Agency catalogues and frame hosts (Austin open data, Texas DOT, Caltrans, Delaware DOT, TfL, Ontario 511, DriveBC, Open Calgary, Live Traffic NSW, Digitraffic Finland, Statens vegvesen, Tallinn, Tarktee, Warendorf, optional NYC DOT) London's frame bucket on public cloud storage, and Google Street View still fallback |
| Map data | Nominatim (forward geocode only), Re:Earth terrain heights, operator-configured Overpass servers, Google Places |
| Street imagery | Mapillary tiles |
| Radio | Radio Browser mirrors (directory + click count) |
| Voice | OpenAI client-secret minting, OpenAI responses (HUD summary), OpenAI Realtime; a local sign-in command-line tool (ChatGPT-login mode only) |
| Local | Operator's own ADS-B decoder feeds (loopback/private only) |

### 5.2 From the browser directly
- Google Map Tiles / Maps JavaScript / Geocoding (3D, Street View, place search), Cesium ion services (tiles, imagery, terrain, endpoint lookups), Esri World Imagery tiles, OpenStreetMap tiles, OpenFreeMap vector tiles, Re:Earth terrain mesh, Photon geocoder, NASA GIBS imagery and Worldview exports, USGS earthquake feed, KartaView API and image hosts, Mapillary Graph API and viewer assets, a community-hosted plate-reader extract (tile host), Bing/virtual-earth imagery through ion, NASA Common Metadata Repository granule queries, and font hosts (Google Fonts, loaded at page start with preconnect hints, so Google sees the visitor IP on every launch). Developer-only: the voice benchmark script can call a Google generative-AI live endpoint, but it is not part of the app.
- **Voice**: OpenAI Realtime service for the session negotiation and the audio/data channel.
- **Radio**: the selected broadcaster's stream host (the broadcaster sees the listener's IP).
- **Event embeds** (Nepal scene only): YouTube/YouTube-nocookie, Facebook and X scripts and frames, disabled in embedded shells that would redirect them.
- **Optional hardware**: WebUSB to a local RTL-SDR (data stays in the browser).
- The content-security policy is permissive for connections (any https/wss) because of the number of providers, restrictive for scripts (self, blob, the maps script host and the three embed hosts).

---

## 6. Telemetry and data collection

- **No analytics, crash reporting, advertising or product telemetry was found** in the application, server, build scripts, CI workflow or markup [OBSERVED-IN-SOURCE, re-checked this revision]. The search covered the commonly used analytics, session-replay, crash-reporting and tag-manager products by name, beacon-style sends, report-to style policy headers and a reporting address in the content-security policy; the only hits were false positives (ordinary words such as plausible or a local variable name). The page makes no measurement call of its own; every outbound call is a data-feed or tile request listed in section 5. The only "telemetry" in the code base is (a) a scene-run log that stays local and is user-downloadable, and (b) the voice developer log (local disk, redacted, content omitted by default).
- The server sends **identifying user-agent strings and referers** (several carry the project's public repository address; Digitraffic receives a dedicated client-name header; Entur an identifying client-name header) to community services as their policies require (CelesTrak, Nominatim, OSRM, Radio Browser, Overpass, Digitraffic, etc.) - this identifies the app, not the user, but it is outbound metadata.
- Requests made on the user's behalf reveal the user's IP to: every browser-direct provider, radio broadcasters, and embed hosts.
- Voice audio goes to OpenAI; screenshots may go with questions (doc 08 section 8).
- Dev-server request logging and Provider-Settings values: only presence is ever reported.
- The maintainers have announced a separate hosted version; its data practices are unknown. UNVERIFIED.
- Ethics line enforced in docs and review: no named-person search, no face recognition, no tracking of individuals; camera frames pass through unaltered with no recognition; plate-reader layer maps tagged locations only (no plate data, no video); data may be delayed, incomplete, modelled or wrong and must not be used for navigation, emergency response, medical, investment or other safety-critical decisions.

---

## 7. Security posture

| Area | Posture |
|---|---|
| Model | Local-first, public-data client; explicitly "not a hardened production service". The development/preview server is a **key broker**: anyone who can reach it can spend the operator's quota. |
| Network exposure | Binds to the loopback interface; LAN is opt-in (loud launcher warning); the one-click launcher refuses LAN and tunnel sharing and discards stale sharing flags; Host-header allow-list ahead of all routes; wildcard/suffix names ignored. |
| Secrets | Server-side except the three browser keys; explicit build allow-list; plaintext owner-only local storage; Keychain stronger option; templates never hold secrets. |
| SSRF/open relay | No client URLs except the bike-share relay (https only, fixed operator host allow-list, two fixed document names, redirects refused); fixed hosts/registries; manual redirect checks; DNS-validated pinned fetches for radio; range header canonicalised; abandoned requests cancel upstream. |
| Cross-site | Gate only on the voice credential, HUD summary, debug-log sink, the two Places lookups and the street-imagery status and tiles; stricter gate for settings and ChatGPT-login; JSON-only assistant endpoint; same-origin checks. Flights, traffic, fires, routing, ships and camera relays are not gated [OBSERVED-IN-SOURCE]. |
| Framing | Denied by default on every document (frame-denial header plus frame-ancestors none); embed mode allow-list (or any) applies to embed-mode documents only; the assistant panel needs no framing. |
| CSP | Full policy (same-origin default, no inline scripts, script origins only for the maps host and event media, frame sources limited to the media players, no objects, blob workers, any https/wss for connections and any https for images); accepted weaknesses documented (eval allowance needed by a bundled UI library; a trusted maps script host with legacy callback endpoints). Detail: doc 10 section 4. |
| Launcher sharing | The assistant HTTP endpoint refuses all requests (not only proxied ones) while launcher sharing is on; the settings and login surfaces are disabled entirely in that state (13f 3.2 item 5). |
| Input handling | Bounded bodies/rows/entities; sanitised errors; schema-validated assistant tool arguments; scene files and imports are data only (no scripts, no credentials, no arbitrary HTML); import limits; prototype-key refusal; plaintext-only cards. |
| Assistant protocol | Loopback-only transports; "transport safety, not authentication"; any local program can use the tools; a panel relay key is a routing guard, not access control; settings, credential, model, assistant and internal routes refused to the relay. |
| Voice agent | Acts only through a fixed tool list; treats feed text as untrusted; model output untrusted. |
| Known limits | Any local program can mint voice sessions; a local reverse proxy hiding the client address defeats loopback checks; the ChatGPT-login path may violate vendor terms; per-IP guards are not billing caps. |
| Vulnerability reporting | Private reporting channel; acknowledge, investigate, credit. |
| Supply chain (review rules) | Inspect dependencies, lockfiles, scripts, CI, plugins, launchers, binaries, symlinks, outbound hosts; never run contributed code with repository secrets; privileged-trigger CI patterns forbidden; isolated disposable environment (Linux sandbox preferred) for risky PRs, no personal secrets or browser profiles. |
| Licence posture | Code MIT; **third-party data and assets are not MIT**; non-commercial datasets are bundled with a carve-out (cables, event pack); in-app attribution popover plus a data-source ledger kept in sync; media provenance recorded; third-party software notices kept for added packages. |

---

## 8. Install and run modes

| Mode | Who | Steps / behaviour |
|---|---|---|
| **One-click installer (Pinokio, Windows/macOS/Linux)** | Non-technical users | Needs Pinokio 8.2 or later. Menu shows Install -> Start -> (Open, Server log) ; after install also Update and "Repair installation". Install: creates the app-scoped environment file from a template if absent, runs the locked-dependency install, runs the setup doctor in a Pinokio-faithful mode (no Keychain import, only that environment file), writes an owner-only "installed" marker, and says "Ready. Return to Pinokio and choose Start." Start: runs as a background daemon on 127.0.0.1 on a free port, waits for a "Ready at <url>" line, then stores that url so the menu offers "Open God's Eye View". Sharing flags are pinned to disabled sentinels so no tunnel can start. Update re-runs the doctor and dependency install; Repair reinstalls. Known: the embedded view may be choppy - open the URL in a normal hardware-accelerated browser; embeds are disabled in this shell. A Windows onboarding job in CI exercises the install path. |
| **Terminal / coding-agent path** | Contributors | Node 24.14.x or 26.x (25 warns EOL; older refused; newer warns); clone; locked install; run the doctor; run the dev server (default port 4173). |
| **Recommended macOS launcher** | Developers | Clears the build cache and resolves keys from shell / environment file / Keychain before starting; local-only by default; LAN via explicit host switch; prints key-exposure warning and LAN URL. Variants: a stricter-binding "secure" launcher and a camera-source tuning launcher. |
| **Production build + preview** | Checking a build | Build uses a separate cache directory from the dev server; preview serves the built front-end and the same provider APIs (build shape: doc 10 section 9); **Provider Settings and setup routes are development-only**; "preview is not a production server"; browser keys are embedded at build time so rebuild after changing them; unknown API paths return JSON 404. |
| **MCP stdio** | AI clients | Command registered with Claude Code, Codex CLI, Claude Desktop, Codex/ChatGPT desktop (config block, absolute paths, Windows-WSL variant); needs the app running; shared panel key per install. |
| **MCP over HTTP** | AI clients by URL | Loopback-only path on the app server. |
| **Embedded panel / embed mode** | Chat clients, pages | Panel build must be rebuilt after app changes; 10-20 s first load; framing allow-list. |
| **Setup doctor** | Everyone | Read-only readiness report; JSON mode for CI. |

Performance baseline (one Apple M5 / Chrome 150 capture, 1440x900, Aug 2026, not a minimum spec): median app-ready about 0.6 s and settle about 1.86 s, 60 FPS in motion and at rest, JS heap about 105 MiB at start; heaviest cold layer activation was cameras (about 19.6 s, 48 cameras, 193 MiB) then space missions (3.6 s) and radio (3.5 s, 750 stations); submarine cables and datacenters had the highest heap (412 and 328 MiB). Software rendering verifies behaviour, not performance.

---

## 9. Testing strategy and quality gates (in words)

### 9.1 Layers of verification
1. **Unit/contract tests** (built-in Node test runner; test files sit beside the module they cover under the source and server trees; a meta-test fails if a test tree is not included in the runner). The reference's own notes quote about 2,255-2,295 tests, but that figure is stale: the current tree has roughly 560 unit-test files and about 6,400 test declarations by static count (parametrised cases may expand it further) [OBSERVED-IN-SOURCE, static count only]. They test pure policy and contract modules (camera guard, cost tracker, voice protocol, analyst engine, schema validators, scene document, sharing bundle, security gate, key-setup core, geocoder chain, source adapters with fixtures, panel/UI markup structure, accessibility focus rules, documentation claims). Two allocation micro-benchmark files (14 tests in all) run only on the calibrated Node major (24) and are skipped elsewhere, so totals depend on the Node version. **Never quote a bare total** (13f correction 4): the reference's own notes put the executed total at about 2,255 on the main branch under Node 25 versus about 2,295 under Node 24, and the 13f census counts about 562 test files; the executed-run figure and the static declaration count are different measures and neither was re-run here [FROM-DOCS-ONLY / OBSERVED-IN-SOURCE, static].
2. **Tracking regression suite** (needs a running dev server and a Chromium): asserts smooth glide on lock-on, calibrated follow framing, trail glued to the icon, military mirrored, annotation z-order.
3. **Headless browser acceptance harnesses** - about 100 scripts, one per feature area (voice routing/auth/WAV in both open-mic and push-to-talk, scenes/director camera-packs-interactions-sharing-timing, traffic, transit, weather, wind, cctv, cockpit, street-level, annotations/draw/geo-import, floor/height datum, first-run, radio, FIRMS, GNSS, map-source tray, panel resize/drag, performance, attribution, etc.). They drive the real app, many with hermetic fixtures (e.g. street-level with every provider answered from fixtures so a dummy token never leaves the page), record the WebGL renderer, save evidence screenshots/JSON, and include "mutation" harnesses that revert each fix one at a time to prove the test is load-bearing. Software-rendered runs validate assertions only; headful real-GPU runs are the visual sign-off surface; screenshots only count when tile loading has settled.
4. **Manual field-test script**: scenario list for voice annotation honesty, route honesty, partial-failure honesty, clear/new-topic races, tracking feel, live video soak (five minutes continuous), with explicit "pass looks like / old bug looked like" statements; deterministic no-mic console hooks exist for annotations.
5. **Recorded-audio voice acceptance** for both input modes and for both auth modes (credential-free outputs saved).

### 9.2 CI gates (every pull request and push to main)
| Job | Checks |
|---|---|
| Verify (Node 24.14.0 and 26.x) | Setup-policy doctor (JSON), formatting check on the adopted file set, package-boundary check (every export belongs to exactly one declared boundary group; each group built and checked for out-of-ownership imports) and import-direction check, share-link layer-token reservation check on pull requests (tokens are immutable and never reused; new layers take the next free token), full unit suite, production build. The allocation gate is required only on Node 24.14. |
| Street-level browser gate | Production build with a dummy token, preview server, fixture-driven browser gate in strict mode (fails on a missed header press unless the panel moved) and the portable panel-resize gate; evidence uploaded on failure. |
| Windows onboarding | Runs the one-click install path, formatting and boundary checks, a short list of focused onboarding unit tests (setup doctor, launcher environment and contract, key-setup hardening, map startup, HUD summary response, ChatGPT-login realtime) and the production build on Windows. |
Concurrency cancels superseded runs; actions are pinned by digest; minimal read-only token permissions and no stored checkout credentials; job timeouts 20 / 30 / 20 minutes; the verify job is not fail-fast across its two Node versions; the layer-token check needs full git history and runs on pull requests only; the browser-gate job installs a headless Chrome and keeps failure evidence 14 days. [OBSERVED-IN-SOURCE]

### 9.3 What CI does **not** do
It does not run the tracking suite or any acceptance harness (they need a live server and a browser); contributors must run the harness covering what they touched and cite it in the pull request. Before a PR: build, unit tests and the tracking suite must be green.

### 9.4 Review process (maintainers)
Establish a trusted policy revision; review usefulness (does it fit the public-data local-first product, API cost, maintenance), design (cancellation, teardown, duplicates, over-engineering), security (boundary changes, new hosts, SSRF, credentials, telemetry), tests/docs (tests that fail on the base; update the runtime reference, changelog and data-source ledger for source changes); choose execution isolation proportional to risk; validate with the standard commands plus the built app via preview; verify actual GPU renderer for performance claims; preserve contributor authorship; record a structured private report; "ready" means gates passed, not merged.

### 9.5 Documentation as a gate
A runtime-state document, changelog and data-source/attribution ledger must be updated with runtime or source changes; new sources require a ledger row **and** an in-app credit entry. Tests assert what docs may claim. **Caution for builders (13f 3.2 item 6):** the reference's current-state document still lists weather radar under "not currently in runtime" although observed rain radar shipped (September 2026), and a source comment says three HUD layouts where there are four. Treat code and the changelog as authoritative over summary prose.

### 9.6 Voice-routing benchmark harness (developer only; 13f GAP-1, SYS-023)
A command-line harness measures how reliably a speech/LLM provider picks the right tool for a spoken phrase, with no app page involved. It sends every provider the production instructions and production tool schemas (for the OpenAI realtime provider it uses the exact session body the credential-minting capability would use), a fresh context per phrase, stubbed tool results, and up to four continuation rounds until the model stops calling tools. Providers: OpenAI Realtime; Gemini Live (a native-audio model that rejects text output, so the harness requests audio and bills the spoken confirmation at the audio-output rate); a local model runtime; any OpenAI-compatible chat endpoint. Four suites: **core** (the routing phrase table with semantic grading plus typed-argument checks), **coverage** (probes for newer layers, each row graded correct / honest-refusal / hallucinated-capability / wrong-tool, graded twice: an original rubric against the original tool schema and a second rubric for the current schema), **dialogue** (two-turn exchanges where turn two depends on turn one, such as a follow-up filter), **capability** (area handles, imagery, map data, point-and-ask phrases). Flags: reusable output folder to collect several models, suite and phrase filters, a total cloud-spend budget, temperature, context size, a reasoning toggle for local models, and a summarise mode. Cost rules are in section 4. A pure speech-lint checker (13f GAP-8) flags hedge phrases and "approximately" before an exact tool number and counts spoken versus answer words per turn; it feeds the per-turn span record and benchmark grading. **Kawn:** run the same harness on candidate Arabic/English models; add Arabic, Gulf-dialect and code-switched phrase rows; keep the honest-refusal grade because it measures the failure the product forbids (claiming a capability that does not exist).

### 9.7 Ordering rules that prevent lost spend and deadlock (13f GAP-4)
(a) The voice cost tracker may only be swapped when the session is fully settled (not active, no data channel and no peer connection), because after an error status the transport can still deliver a late completion carrying usage; swapping earlier loses that spend or lets repeated tier toggles dodge the cap. (b) Every tool call must be answered once, even refused ones: a call from a superseded response gets a terminal "superseded" output, because an unanswered call leaves the conversation waiting and deadlocks the model; the refusal itself must not trigger a new response.

### 9.8 Other test-adjacent notes
- Offline imagery capture tools (13f GAP-9) are developer utilities, not product features: stitch satellite tiles at a chosen zoom (about 6.4 cm per pixel at zoom 21, 3.7 cm at 22) into a square ortho image with a ground-sample-distance report, fetch eight compass-heading street images at a point, reproject a panorama to pinhole views, and render a headless 3D capture. Useful as ground truth for Kawn's UAE camera-pose calibration.
- Detection-idle rules (13f GAP-6) are a tested pattern: repaint on change, request exactly one more frame only while spanning work is outstanding, every outstanding-work predicate must terminate, paint and demand read the same monotonic frame timestamp.
- Removed or retired in the reference (do not specify for Kawn): experimental AI video-edit look, shift-drag camera calibration, the first weather radar, public Overpass instances as a default, and the left-rail map stack panel (13f section 2.5).

---

## Kawn notes (UAE city-intelligence: Dubai, Abu Dhabi, Sharjah; Arabic + English, RTL)

**Keep**
- Zero-key start with graceful degradation and explicit "key required" states; per-feature key naming on the layer row.
- Presence-only settings status; externally-managed read-only values; atomic, permission-first credential writes; abort-on-unreadable-store rule.
- Provenance-aware attribution popover and a licence/terms ledger as a release gate.
- Layered testing: pure-policy unit tests, hermetic browser gates with fixtures, mutation harnesses, strict CI for boundaries/format/tokens, manual honesty scenarios.
- Immutable share-link identifiers and the review/evidence rules for risky changes.
- Honest incomplete-cost indicator and "not for safety-critical use" disclaimer.

**Adapt**
- Replace plaintext local key storage with an admin console backed by a secrets manager; per-role access; rotate keys; audit access.
- Replace per-IP in-memory throttles with per-user/tenant quotas and real spend caps; budget alerts wired to provider consoles.
- Replace one-click Pinokio with an installer/packaged desktop or a hosted tenant deployment; keep a diagnostic "doctor" and an "update/repair" action; provide Arabic UI for setup.
- Data-residency: UAE-hosted voice/LLM (or fully local), UAE-hosted tiles/geocoding/routing; review PDPL and emirate-level data rules; log retention policy for voice content (off by default, explicit consent when on).
- Add a privacy notice and consent for microphone, screenshots to a model, and any third-party embeds; add a content-security policy that restricts connections to an allow-list instead of any https.
- CI: add Arabic/RTL visual regression, accessibility checks, language-specific voice acceptance (Gulf dialect audio fixtures), data-licence lint in CI, and licence-flag checks that block non-commercial datasets in commercial builds.
- Key registry per UAE provider (RTA / ITC / municipality APIs, meteorology, mapping vendors) with the same "unlocks / how to get / tier / client-exposed" metadata.
- Do not ship browser-visible keys beyond what a vendor forces (maps key, tile token); restrict at the vendor. Hold Google 3D under Kawn's own commercial agreement.
- Keep the first-run missions idea but re-theme for UAE (Skyline, Ports, Weather/Dust, Explore); a mission must run through the same layer-enable paths as the visible controls and must not set the detection override flag (it would silently disable the military-look auto-preset for the session).
- Owner decisions (docs/09): Dubai default view, Scope default feather 100, location tray - none changes key or privacy posture, but the setup/onboarding copy must be Arabic/English.

**Skip**
- ChatGPT/Codex sign-in reuse; macOS-Keychain-specific launcher variants (keep concept); LAN sharing opt-in (replace by authenticated multi-user deployment); Pinokio-specific workarounds.
- Non-commercial datasets and unlicensed route data in any commercial build.

## Open questions / UNVERIFIED
- No network capture was performed; the list of outbound destinations comes from documentation and source strings and may miss minor hosts (e.g. library-internal calls, map-library asset hosts).
- Absence of telemetry is now based on a source-wide search (see section 6) plus the content-security policy, not on documentation alone; third-party widgets (maps script, embedded video players, fonts) have their own tracking outside the product's control and were not network-captured. UNVERIFIED at runtime.
- Real test counts vary by Node version (about 2,255 under Node 25 versus about 2,295 under Node 24 in the reference's notes; the census counts about 562 files); the figure in section 9.1 is a static count of declarations, not an executed run.
- Real-device behaviour of the voice stack on iOS Safari and Android Chrome; whether any telemetry exists beyond the local debug log (none found by reading, no network capture); the stored camera-calibration record and edit component may still be reachable after the interaction was removed; the Windows/Claude Desktop shared panel key across processes is unconfirmed upstream.
- Provider pricing/quotas (Google 3D free sessions, TomTom, OpenAI rates) are mid-2026 snapshots flagged by the reference as needing release-time verification.
- Whether Windows Credential Manager or Linux keyring options exist beyond the plain-file route - none documented.
- Exact behaviour of the one-click installer on Linux sandboxes/GPU-less machines not examined.
- The unit-test runner's parallelism/timeouts and flake policy - not documented beyond the allocation split.
- No application run, no install, no live testing was performed for this document.

## Revision log
- Fixed the key-registry count (ten entries, nine visible, one hidden) and added default voice model tiers in descriptive terms.
- Telemetry claim re-checked across code, scripts, markup, CI and the content-security policy: still none found; method and residual third-party leaks (fonts, maps, embeds, radio, identifying user-agents) spelled out.
- Replaced the stale unit-test count with a current static count (about 560 files, about 6,400 declarations) and kept the Node-version caveat.
- CI gates completed: Windows job also runs focused onboarding tests and the build; added timeouts, fail-fast, history and evidence-retention details.
- Outbound destination list extended (bike-share operator allow-list, transit operators, London frame bucket, geocoding and CMR browser calls, fonts, developer-only benchmark host) and the security table now states which endpoints are cross-site gated and which are not.
- **Pass 3 (census merge).** Folded in 13a, 13d and 13f: chip behaviour corrected (retires when all keys set; label only when reopened) and status-listing rendering noted; Pinokio native form release named; extra metering rows (status chips, Google tile token renewal, benchmark spend with dated pricing); security table gains full framing/CSP wording and the sharing-on refusal of the assistant endpoint; test totals now explicitly Node-version dependent (about 2,255 versus 2,295) with a never-quote-a-bare-total rule; new sections 9.6 (voice-routing benchmark harness and speech lint), 9.7 (cost-tracker swap and tool-call answer ordering rules), 9.8 (offline imagery tools, detection-idle rules, retired features) and a builder caution about stale reference prose; build-cache note in install modes; Kawn notes and open questions extended. Confidence: census-derived [OBSERVED-IN-SOURCE] or [FROM-DOCS-ONLY] as tagged; no new run.

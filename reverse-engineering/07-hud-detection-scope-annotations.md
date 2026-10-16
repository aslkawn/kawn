# 07 - HUD, Detection Overlay, Scope, Annotations, Measurements, Recording and Credits (behavioural study)

**Purpose.** This document describes the screen-space "instrument" layers that sit on top of the globe in the reference product: the intelligence-style heads-up display (four layouts and every field), the one-line situational summary, the detection overlay that boxes and labels moving objects (density, allocation, fade, outside opacity, colours), the Scope keyhole as it interacts with detection, the Draw/whiteboard annotation system and file import, measurement readouts, the scene-recording mode with safe-frame guides, and the mandatory credits/attribution surface. It captures what the user sees and the numeric rules, for a clean-room rebuild. Unverified items are marked UNVERIFIED.

Confidence tags used below: [OBSERVED-IN-SOURCE] confirmed by reading the reference implementation, [FROM-TESTS] confirmed by an automated test, [FROM-DOCS-ONLY] only in project prose or the live walkthrough, [INFERRED] derived. Numeric values in this document were re-checked against the implementation in the revision pass unless a section says otherwise.

Mechanism detail: 13f sections 1F to 1H and gaps 6 and 11 (HUD, detection zero idle cost, scope defaults); 13e gaps G2, G3 and G8 and its corrections (recording mode, safe-frame geometry, annotation intent hints); 13b AIR-116 and AIR-117 (aircraft detection).

All of these layers are decorative-plus-informational overlays: the HUD and detection are read-only; Draw is the only one that creates user content.

---

## 1. Heads-up display (HUD)

### 1.1 What it is
A full-screen overlay showing fictional "reconnaissance imagery" metadata derived from the live camera. Almost every value is computed from the camera pose and the clock; a few identifiers are random per session and purely decorative. Toggle with the HUD button in Display (key **H**), choose a layout with the Layout dropdown (revealed when the HUD is on). HUD on/off and layout are part of the share link and of scenes.

### 1.2 Visibility rules
- First-load baseline: HUD **on**, layout **Tactical**.
- Three modes: auto (follow the look: show for CRT/NVG/FLIR, hide otherwise), forced on, forced off. Any manual toggle or explicit set switches auto off permanently for the session; the scene/recording system, voice and scripting can request auto again. Because the first-load baseline sets the HUD explicitly on, and every military-look selection does too, auto is unreachable for ordinary users by clicking; it can only be re-entered through voice, scripting or recording mode (resolved from the implementation; in auto mode the HUD shows for CRT, NVG and FLIR and hides for the rest). [OBSERVED-IN-SOURCE]
- The HUD fades in and out over 0.4 s when shown or hidden. It is a pure display layer and takes no pointer input.
- A hidden HUD costs nothing: telemetry timers skip work while it is hidden, the geoid lookup is not requested, and no summary requests are sent.
- Voice and tools can set the HUD on/off/auto and the layout, receiving the resulting state back.

### 1.3 Fields (full set, shown in Tactical and Operator)
Top-left block
- A fake classification banner (top-secret style marking), a mission/sensor identifier pair (random per session, "KH-11-style" mission id 4000-4199 and "OPS" sensor id 4100-4199), the **mode label**, and a **SUMMARY** caption with the summary line (section 3).
- Mode label shows the current look: NORMAL, CRT, NVG, FLIR, ANIME, NOIR or SNOW (Cyber appends a "// CYBER" tag in a red accent).

Top-right block
- **REC** with a red dot that blinks every 800 ms, followed by the current UTC date-time to the second with a Zulu suffix (updates every second).
- A pseudo orbit line: an orbit number (47000-47999) and a "descending pass" number (100-299), both random per session.

Bottom-left block
- **MGRS** grid reference of the camera's sub-point at 10 m precision, space-separated into grid zone, 100 km square, easting, northing; shows dashes if conversion fails.
- Latitude and longitude in degrees-minutes-seconds with two decimals and hemisphere letters (lat two-digit degrees, lon three-digit).

Bottom-right block
- **GSD** (ground sample distance) in metres and **NIIRS** (image-quality rating). GSD = camera height above the ellipsoid x 0.000375 (minimum 0.01 m); NIIRS = 10.25 - 3.32 x log10(GSD in inches), clamped 0-9, one decimal.
- **ALT** (rounded metres) and **SUN** elevation (one decimal, degrees). ALT is reported as height above **mean sea level**: the camera's ellipsoidal height minus the local geoid undulation (EGM96 grid, loaded lazily, about 2.7 MB, requested on the first telemetry tick of a visible HUD, cached per roughly 1 km cell); until it loads (or if it fails) the uncorrected height is shown, and when it arrives the readout and summary repaint together. Reason: a cockpit parked on a runway otherwise reads negative altitude.
- **AIS** line: normally a dash. When a vessel is selected it becomes a three-line readout - name (trimmed to 32 chars), type (24 chars), speed, heading, MMSI, position time, and a STALE marker if the selected vessel missed recent refreshes. Slightly brighter when active.

Left edge (text rotated 90 degrees, vertical): **COLL** (current UTC time) and **ONA** (off-nadir angle in degrees, one decimal).
Right edge (rotated the other way): fixed decorative text "band PAN, bits 11, level 1A".

Operator/Minimal/Cyber also show a **bottom bar** with the compact line: MGRS, LAT, LON. Operator additionally shows a **top bar** with three segments: the fake classification (dim), the mission id (centred, brighter) and "page 1/1" (dim).

### 1.4 Layout definitions

| Layout | Composition | Hides |
|---|---|---|
| **Tactical** (default) | Four corner blocks with large corner bracket glyphs; vertical edge text; no top or bottom bar. Corner blocks sit 36 px from the left and right edges; top blocks start at about 14% of viewport height (min 136 px); bottom blocks sit above the bottom dock at roughly 2% of viewport height plus 6.5 rem (the older fixed 180 px was superseded by the dock-relative value) | Top bar, bottom bar |
| **Operator** | Tactical plus a centred banner at the top (about 52% of viewport width, max 720 px, dark gradient with cyan glow, top edge at about 10% of height, min 90 px) and a bottom bar (about 48% of width, max 680 px, sitting roughly 2% of height plus 9.8 rem above the bottom); the four corner blocks become framed rounded boxes (8 px radius, dark translucent, hairline border) instead of bracket glyphs | Bracket glyphs |
| **Minimal** | Only the top-left block (mode and summary, narrower, 280 px max, moved up to about 88 px from the top) and a bottom-left coordinate block plus the compact bottom bar; the bottom-left block sits roughly 2% of height plus 5.4 rem above the bottom and its bar about 9 rem | Classification, system ids, top-right, bottom-right, both edge texts |
| **Cyber** | Red/slate theme: a large thin-ringed circular **sonar disc** (up to 80% of viewport width or 104% of height) with 8 range rings (default) and a rotating sweep (one revolution per 5.2 s) behind the corner cards; corner blocks are chamfered red-framed cards; bottom bar is chamfered with a red gradient edge and sits near the bottom above the dock (width up to 48% of viewport, max 680 px); no top bar; the fictional ids are dropped (it keeps only mode, summary and locality). Whole app shell reskins (see doc 05). Entering Cyber explicitly also switches the look to FLIR with Ironbow 0.42 | Classification, system ids, top bar |

- HUD text colour follows the look (amber CRT, green NVG, white FLIR, cyan others); Cyber forces near-white text with muted-red glow and border.
- In Cyber, a small "sonar" control block appears under Layout with ON/OFF, five sliders (see doc 05) and a status line; sonar settings are session-only.
- The Layout dropdown lists Tactical, Operator, Minimal, Cyber; an unknown name falls back to Tactical.
- Layout changes re-flow neighbouring panels (left rail, right rail, credit line). Concretely: in the **Tactical** layout, whenever any panel in the right rail is expanded, the collapsed sibling panels' launchers are hidden so the expanded one has the rail to itself; in Operator, Minimal and Cyber the collapsed launchers stay visible. (This is the real difference behind the old observation that the Context affordances looked different between Tactical and Operator.) In Cyber the rails also act as an accordion: opening one panel collapses the others, and they are restored when Cyber is left. [OBSERVED-IN-SOURCE]
- The Cyber layout on desktop widths (961 px and up, 650 px tall and up) additionally places the upper-left status card at a fixed 238 px width about 10% from the top and 28 px from the left, moves the rotated COLL/ONA edge text into the clearance lane above the left rail, and puts the bottom-right block in the corner at 28 px; in this large-window case the Cyber bottom bar is hidden altogether (it appears only in the mid-size band, wider than 720 px but narrower than 961 px or shorter than 650 px, and is hidden again at 720 px and below); the left rail is a 300 px column with a red hairline on its left, and the summary line fades out at its right edge instead of showing an ellipsis. [OBSERVED-IN-SOURCE]

### 1.5 Update cadence
| Item | Cadence |
|---|---|
| UTC time and Zulu stamp | 1 s |
| REC dot blink | 800 ms |
| Camera-derived values (MGRS, lat/lon, GSD/NIIRS, ALT/SUN, COLL, ONA) | 250 ms, only while visible |
| Deterministic summary line | immediately on first telemetry, on camera settle, when a feed's state changes, when altitude datum correction arrives |
| AI summary | on first settled view, then every 15 s when something changed |

### 1.6 Derived values
- **Sun elevation:** simplified solar model - declination from day of year, hour angle from UTC plus longitude/15; can be negative at night.
- **View band (altitude classes for the summary):** street under 1.2 km; city under 5 km; metro under 30 km; regional under 250 km; global above. (The detection label budget uses slightly different bands, section 5.2.)
- **Region label:** coarse bounding boxes - arctic, antarctic, North America, South America, Europe, Africa, Asia, Oceania, otherwise northern/southern oceanic grid.
- **View window:** the width and height in kilometres of the camera's view rectangle on the ground (longitude span x 111 km x cos(latitude), latitude span x 111 km), antimeridian-safe; shows N/A when the view rectangle is not computable (full-globe).
- **Nearest known point:** against a built-in table of curated landmarks (nine reference cities, about five landmarks each, including Dubai; an older comment in the source says eight) using great-circle distance; reads "near <landmark> (<city>) <n>km" only when within **150 km**, otherwise falls through to a "sector" readout showing latitude and longitude with hemisphere letters at two decimals (earlier bug: announcing a landmark 2,470 km away).
- **Local time tag:** UTC plus the integer round of longitude/15 hours (approximate - not the real civil time zone). It is printed as an offset tag such as UTC+4, not as a clock time.
- **Region label quirk (matters for the UAE):** the region boxes are tested in a fixed order, and the Africa box reaches east to longitude 55, so a camera over Abu Dhabi (about 54.4 E) reads AFRICA while Dubai and Sharjah (just east of 55) read ASIA. [OBSERVED-IN-SOURCE]
- The corner telemetry's ALT is mean-sea-level corrected, while the view-band thresholds in the summary line deliberately use the uncorrected height they were tuned against, so the two can differ by the geoid offset near a band edge. [OBSERVED-IN-SOURCE]

### 1.7 Narrow-window reflow of the HUD [OBSERVED-IN-SOURCE for the rules; the resulting picture is INFERRED]
- **Up to 720 px wide:** the two vertical edge texts (COLL/ONA and the band text) are removed in every layout. Corner blocks move to a 16 px inset on the left and right. The Operator top banner stretches to the screen width minus 24 px and drops its centring (it keeps its top position of about 10% of height, min 90 px); the bottom bar is also meant to stretch and use 9 px type (see the precedence note below). The Cyber layout drops both bars entirely. [OBSERVED-IN-SOURCE]
- **Rule precedence matters:** the layout-specific, dock-relative positions for the bottom blocks and the bottom bar are written with a more specific selector than the narrow-screen rule, so they win. The practical result is that the bottom bar keeps its dock-relative height and 48%-of-width size but is left-aligned at 12 px (the centring was removed) instead of the intended full-width strip. Treat this as a likely defect, not intended behaviour. [INFERRED]
- **Summary line:** it never wraps; at a narrow block width it is cut with an ellipsis (Cyber uses a fade-out mask instead). The top-left block can grow up to 420 px wide, so on a 375 px phone the two top corner blocks are expected to overlap or run off-screen; nothing in the rules resizes them. [INFERRED; not seen live]
- **Minimal layout and the dock:** when the bottom dock reflows (two rows at 980 px and below) the Minimal coordinate block and its bar are meant to rise with it (extra 6 to 7 rem), but a later, equally specific rule restores the single-row heights, so the block may sit behind the taller dock. Unconfirmed visually. [INFERRED]
- **Cockpit:** at 760 px and below the map HUD is hidden outright; above that it renders at about 78% scale and 72% opacity in the four peripheral corridors, with its summary line narrowed to at most 34% of the width (380 px) and set in 9 px type, and its edge texts pulled in to 22 px. [OBSERVED-IN-SOURCE]
- **Embed mode** hides the HUD entirely. [OBSERVED-IN-SOURCE]
- There is no phone-specific HUD design: no condensed single-line variant, no tap-to-expand. For a mobile product the HUD should become a single status strip.

---

## 2. Intel summary line

### 2.1 Deterministic line (always available)
Text composed locally from the latest metrics: mode label, view band, locality (near-landmark or sector), region, altitude (metres, or kilometres with one decimal at 1 km and above, MSL), view window (width x height km, minimum 1), sun elevation (integer degrees), ONA (integer degrees), local-time tag, and - only when something is wrong - a feed-provenance suffix. The suffix names the overall non-nominal state (stale, degraded, fallback, loading or unavailable) and up to two affected layer names. Fields are separated by vertical bars. Before any telemetry exists the caption reads a standby phrase.

### 2.2 AI line (optional upgrade)
- Needs a server-side key for a text model; without it the server answers a deliberate "not configured" payload (a 200 response with four fields) and the deterministic line simply stays. This is not treated as an error.
- Request context: place labels, street labels and nearby-place labels read from the basemap label context, the names of enabled layers, and a feed-provenance envelope (per enabled layer: state, source, count). Camera coordinates are not sent as a reason to invent places.
- The instruction to the model: exactly **five words**, no punctuation or markdown, only the supplied labels; if any enabled layer is non-nominal the five words must include that state word and must not present the view as live.
- Client rules: aborts after 5 s; a response is discarded in favour of the deterministic line if it fails or does not contain the required state word; failures reset the change signature so the next 15 s tick retries; a stale summary is dropped if the feed provenance no longer matches; the output is sanitised to exactly five words (the server never relays provider error text); requests are skipped when the context is unchanged and when a policy gate (cost/rate limit) says not to; a camera move invalidates in-flight requests (stale answer dropped). The AI line is typed in two characters every 24 ms; deterministic updates appear instantly.
- Observed lag: the caption can briefly name a previous landmark after a fast camera move (it settles on the next camera-settle refresh). It is also refreshed by voice/camera-settle.
- Rate limiting: the shared realtime/summary endpoints allow 30 requests per window per client (server concern).

---

## 3. Detection overlay

### 3.1 What it does
Draws screen-space corner-bracket reticles and small two-line label cards on visible moving or mapped objects across layers: aircraft (civil and military), satellites, vessels, road-traffic dots, transit vehicles, camera feeds, fires, bike-share docks, local receiver aircraft. It is an overlay on a transparent canvas above the scene; it selects a stable subset so labels don't flicker, avoids collisions, and fades labels away from the centre keyhole.

### 3.2 Controls (Display panel, "DETECT")
- Main button with the current profile name; when detection is off it reads "DETECT"; when on it reads SPARSE, BALANCED or DENSE (Dense lit with a special hue). Key **D** toggles off/on (restores the last non-off profile).
- **Density** slider: five stops 0, 25, 50, 75, 100 (step 25). Default 75.
  - 0 and 25 -> SPARSE; 50 -> BALANCED; 75 and 100 -> DENSE. Any arbitrary value is rounded to the nearest stop (below 12.5 -> 0, 12.5 up to 25 -> 25, above 25 and below 75 -> 50, 75 up to below 87.5 -> 75, 87.5 and above -> 100). [OBSERVED-IN-SOURCE; corrected: the older text put exactly 75 in the middle band]
  - Choosing a profile by name (voice/legacy "survey/normal/panoptic/god/on") sets that profile's canonical density: Sparse 25, Balanced 50, Dense 75. A profile that conflicts with an explicit density is rejected.
- **Allocation:** Elastic (default) or Weighted. A stored preference (browser storage) survives sessions; the share link carries it too.
- **Fade** slider 0-40%, default **7%**: the width of the fade band beyond the keyhole, as a percentage of keyhole radius.
- **Outside** slider 0-100%, default **1%**: opacity of labels and cards beyond the fade band. (Defaults have been retuned several times by the owner: outside 5 -> 3 -> 1, fade 16 -> 7.)
- The three tuning rows (allocation, fade, outside) and the density row appear only when detection is on.
- Hand-editing any detection control (or voice/tools) marks detection as user-overridden for the session so look presets never change it again, and claims the "visual" lane against a pending share-link restore.
- Detection is **on by default for every look including Normal** (Dense at 75%).

### 3.3 Keyhole geometry
Single shared circle at screen centre with radius 1.05 x half the viewport height. Labels inside are fully opaque; between radius and radius + fade band opacity falls linearly to the outside value; beyond that it stays at the outside value. The same circle drives the Scope mask, the celestial ring, the Contacts compass and cockpit tapes.

### 3.4 Selection, density and budgets
- **Label budget** (collective maximum labelled objects) depends on the view scale (camera altitude) and density stop:

| View scale (altitude) | 0 | 25 | 50 | 75 | 100 |
|---|---|---|---|---|---|
| Street (< 1.2 km) | 9 | 23 | 45 | 68 | 90 |
| City (< 4.5 km) | 7 | 18 | 35 | 53 | 70 |
| Metro (< 20 km) | 6 | 14 | 28 | 41 | 55 |
| Regional (< 150 km) | 4 | 10 | 20 | 30 | 40 |
| Global (>= 150 km) | 6 | 14 | 28 | 42 | 56 |

  (Global has a larger ceiling because the space around the Earth disc has room for satellite cards.) In the measured baseline, 25/50/100 percent produced 14/28/56 labels.
- **Allocation across layers:**
  - *Elastic:* equal share per layer with demand; unused entitlement is lent to layers that still have demand.
  - *Weighted:* each layer with demand first gets one slot (if capacity allows); the rest is split proportional to the square root of its candidate count times a semantic weight (military 1.4, road traffic 1.15, camera feeds 1.1, flights 1.0, satellites 1.0, ships 1.0, bike share 0.9); remainders go by largest fraction.
- Candidates per layer are capped (2,600) and per-layer cohorts are materialised as bounded reservoirs (64-256), chosen by a stable priority then hash so the same objects win each solve. Semantic priority: military 120, camera feed 50, vehicle 35, aircraft 30, satellite 25, ship 25, other 10; layers can force-skip labels (priority 1000) for special cases.
- **Sparse mode** also draws a faint circular focus ring (radius = half the smaller screen dimension, 35% opacity, glow colour) and selects only objects inside it.
- Membership/collision are re-solved every **125 ms** (accepted labels reproject every frame). Labels are placed on a 32 px collision grid; a label fades in over 150 ms, fades out over 300 ms, stays at least 2.5 s once shown, and a dropped one is on a 1.2 s cooldown before it can return - preventing strobing. New detection (enable) fades in over 220 ms. A paint slower than ~22 ms makes the sensor lane skip some frames (callouts replay the last solve so plates don't strobe against persistent brackets).
- **Zero cost when idle (five rules to copy, census merge):** detection takes no continuous-render hold. It repaints on change and asks the frame governor for exactly one more frame only while work remains outstanding. (1) Every outstanding-work test must terminate: the enable fade-in, label fades in both directions, and an unrun solve. (2) Paint and demand read the same monotonic frame timestamp, never a wall clock. (3) A skipped paint defers, never cancels, from one shared decision. (4) A changed detectable set (a layer tick or a visibility change) marks the solve dirty without itself requesting a frame. (5) Drawing happens from the single post-render listener, not before the render, so the boxes match the final camera frame. Measured: 0 renders per 5 s on a parked empty scene versus 301 with a hold. Coupling to remember: aircraft brackets stay prompt only because the aircraft layers themselves hold the render loop; remove those holds and prompt brackets disappear silently. [OBSERVED-IN-SOURCE, 13f SYS-120, SYS-127, GAP-6]
- Detection **suspends** during inter-city world jumps (preset city or landmark pill clicks while another preset city is active, and reset globe; not typed or voice flights, see doc 06 section 1.3) and resumes afterwards, and is kept on through Contacts/Cockpit by policy (doc 06).
- In Cockpit, bracket opacity is multiplied by 0.45 so brackets don't dominate first person (labels and user tuning are unaffected). Aircraft brackets also keep a readable floor: 35% alpha at the default 1% outside opacity, scaling smoothly with the Outside slider (0 stays 0, 100% rises to 100%) so the slider still does something for brackets; other types follow the keyhole alpha strictly.

### 3.5 What is drawn per object
- A corner-bracket reticle (four L-shaped corners, segments about 55% of the smaller half-side, at least 4 px), batched into one stroke; for aircraft the reticle scales with the aircraft billboard as the camera approaches (near 1 km factor 3.0, far 8,000 km factor 0.5). Transit vehicles use a variant.
- A **single-line label card** (corrected in the revision pass; the earlier "two-line card with class" is a retired design whose helper still exists but is not used for painting): the bright identifier (callsign or similar) followed on the same baseline by one dim "micro" field (a flight level, speed in knots, or similar metric, drawn slightly smaller at about 80% strength). Layout: a thin tier-coloured accent bar on the left edge (about 2-3 px), 6 px right padding, 4 px top and bottom padding, monospace 10 px text (the micro field at about 90% of that size), a rounded dark plate (3 px corner radius, about 46-56% opaque, slightly stronger for satellites, scaled down to 22% when the label sits over open sky so it reads like bare text), and a thin leader line (1 px, tier colour at about 60% strength) from the object's reticle corner to the nearest corner of the plate. The card is as wide as its text needs (the width is character count times a fixed glyph width), so there is no wrapping and no per-field truncation in the painting step. [FROM-TESTS: the host test states "single-line track label - bright id, one dim micro field"]
- Content by type (what is actually fed to the label): aircraft - callsign (else registration, else hex) plus flight level; ships - name plus speed in knots (blank when moored or unknown); transit - route or vehicle label plus mode and activity (for example the mode and a speed in km/h); camera feeds - a camera code; fires - a fire id; road dots - a vehicle id; bike docks - station name; satellites - name (a class tag may be folded into the name by the satellite layer; [FROM-DOCS-ONLY]); converted novelty contacts - their novelty class label (the entries for camera feeds, fires, road dots, bike docks and satellites come from the earlier study and were not re-read per layer). The aircraft "class" field exists on the object but is not painted. A selected or tracked object is skipped by the ambient label (it has its own card).
- Colour **tiers** (default cool palette): civil aircraft cyan, military amber, sea teal, space violet, vehicle slate; road traffic buckets jam red / slow amber / free green / no-data grey; transit modes bus green, tram amber, subway red, rail lilac, ferry sky, unknown pale grey. The CRT palette shifts these toward ambers, NVG toward greens with military shifted to red-orange and free-flowing traffic cyan, FLIR toward whites and warm tones. (Values in doc 05.) The overlay also applies a screen-blend with slight contrast/saturation and a soft glow (about 3 px), plus faint animated scanlines (period 4 px) tinted per look.
- Selected/tracked objects get a distinct larger "tracked card" painted on top (never distance faded): title 13 px semibold, details 11 px, cyan selected border. The tracked readout reads only the layer's cached display position (never a fresh evaluation), which avoids jitter; tracked items keep a box (distance-scaled, about 56 by 44 px near versus 22 by 14 px far) but suppress the ambient label; ship cards use the vessel layer's own protected card. Aircraft layers feed the drawer only frustum-visible and horizon-visible contacts, military weighted highest. In Cockpit the detection set omits only the active cockpit subject (see doc 06). [13f SYS-125, SYS-126; 13b AIR-116, AIR-117]
- Cards with an action (for example "select this aircraft") are mirrored into an off-screen **"visible map targets" accessibility list** with a polite status region, so assistive tech users can reach what sighted users click.
- A debug banner with counts and timing exists but is hidden unless a query switch is used.

---

### 3.6 Label placement and leader lines [OBSERVED-IN-SOURCE]
**Ambient detection labels (the single-line cards above):**
- Each object has a reticle of a given half-size (it grows for aircraft as the camera nears, see 3.5). A label has exactly four candidate positions, one per diagonal: up-right, up-left, down-right, down-left of the reticle. The card sits 8 px sideways and 12 px vertically clear of the reticle corner it hangs from, and the leader runs from that reticle corner to the card's opposite corner (for example, a card above and to the right is joined from the reticle's top-right corner to the card's bottom-left corner).
- A candidate position is discarded outright if any part of the card would fall within 4 px of the screen edge, would overlap a reserved piece of interface chrome, or would sit where its fade opacity is zero. The reserved list is shared with the rich cards below: title block, style indicator, top action cluster, loading chips, both rails, the Display panel, the bottom dock, the voice pill, the attribution line, the four HUD corner blocks and both HUD bars, the mission/event side panels, and (in Cockpit) only the two solid context and signal windows; the cockpit's thin line-art instruments are deliberately excluded because the cards render beneath them. Nothing is clamped or nudged: if all four positions fail, the object keeps its reticle and gets no label.
- Among the surviving positions the solver prefers the one the label used last time (a "sticky" corner, so a label does not hop around as the camera moves), then the first free one in the order up-right, up-left, down-right, down-left. "Free" means not overlapping an already accepted label on a 32 px collision grid. Rejected objects follow the fade, minimum-lifetime and cooldown rules in 3.4. Accepted labels reproject every frame; membership and corner choice are re-solved only every 125 ms.
- The leader is always drawn with its card; there is no separate leader fade or animation for ambient labels.

**Rich world cards (tracked/selected readouts, installation cards, camera thumbnails and similar, drawn by the shared overlay host):**
- Four candidate positions as well, but straight above, below, right and left of the anchor, with a default gap of 12 px (larger gaps when the anchor has a dot of its own). The automatic order starts with "above" when the card fits above the anchor (at least 4 px from the top) and "below" otherwise, then tries the other vertical side, then right, then left. A source can ask for a preferred side, or for vertical-only placement. Cards are clamped to stay at least 4 px inside the screen and are not dropped for touching an edge. Interface chrome is treated in two classes: chrome that paints above the cards (panels, dock, title, voice pill) is merely avoided when a clear position exists, because the chrome would hide the card anyway and a card that hops or vanishes near a panel was judged worse; chrome that paints below the cards (the HUD text) is an absolute veto, since a card must never cover HUD text. If every position hits the veto class, the card is not drawn at all.
- The leader for above/below placements is strictly vertical from the anchor to the nearest card edge, even when edge clamping has pushed the card sideways (a diagonal stub read as "not attached"); for left/right placements it runs horizontally to the card edge at the anchor's height clamped to the card's vertical extent. A "straight" style is the default; an "elbow" style (vertical, then horizontal) exists for side placements. Leaders are 1.35 px in the shared style (about 1 px on scaled cards) and can reveal progressively when a scene animates them.
- Anchor dots are 3.2 px radius with a dark 1 px ring; card plates are about 82% opaque dark teal (94% when selected) with a 4 px corner radius, a pale hairline border (cyan 72% when selected) and monospace type of 10 to 13 px.
- Distance, altitude and scale fades apply to size and opacity so far-away cards shrink; an anchor-separation pass keeps neighbouring anchors from stacking on one another (cards of the same collision group are spaced; pinned, active and protected entries are never dropped).
- A hard budget caps the number of simultaneously painted ambient cards at about 1,150 across all sources; pinned and protected cards bypass it. [OBSERVED-IN-SOURCE]

## 4. Scope (circular viewport) and feather - detection interplay

Visual details are in doc 05 section 4. Behavioural summary for this layer:
- **Scope** button (default on) and **Feather** slider (default 11%, 0-100%). Scope darkens outside the keyhole; Feather softens only the mask edge, separate from the detection **Fade** that softens label opacity. 0 is a hard cut, 100 a long smooth vignette. The live walkthrough compared 100 and 0 captures.
- The adaptive terminus fades the corners to solid black below 7,000 km camera height; a link can pin it between 94% and 100%.
- Mechanism: one fixed canvas below the detection surface carries a radial gradient, transparent inside the shared keyhole and feathering to near-opaque page black outside; it redraws only on resize, tuning change, pixel-ratio change or a quantised alpha step, and has no animation loop. The height sample is throttled and the repaint gated on the quantised step (about 12 repaints across a zoom gesture). A feather of 0 takes the hard-crop path. A link that omits the feather field is parsed at the old default of 35%, so an old link keeps what its author saw (see doc 05 section 3.4 on defaults living in several places). [13f SYS-140 to SYS-142]
- **Kawn owner decision (docs/09, 2026-10-16):** Kawn opens with Scope on and feather at 100% (smoothest fade, no hard edge), over Dubai; the reference's 11% is not the Kawn default. Scope stays one tap to turn off and feather stays adjustable. The "centre" that matters is the free map area (excluding open side panels and, on phones, the sheets), not the whole window, and the hero live data (flights, metro, key points of interest, hotspots) belongs in that centre zone while the edges stay calm. The visual treatment must be Kawn's own.
- Scope on/off and feather are part of the share link and scene shots; the compositing order, bottom to top, is: globe, scope mask, celestial ring and HUD text, detection sensor surface (brackets), detection/world cards, then panels and chrome. The HUD text therefore sits beneath detection, and the label solver keeps cards off the HUD text rather than relying on stacking. [OBSERVED-IN-SOURCE; corrected from an earlier statement that put the HUD on top]
- When Scope is off, the keyhole circle still defines the detection fade band (labels still fade toward the outside opacity even though the picture is not masked).
- Tilt view keeps the vignette.

---

## 5. Draw, annotations and whiteboard

### 5.1 Two ways in, one engine
Marks reach the globe either from voice (the assistant resolves names to places) or by hand with the **Draw** tool. Both feed the same annotation engine, so hand-drawn marks look, persist, de-duplicate, list and clear exactly like spoken ones.

### 5.2 Draw tool (Display, "Draw")
- Toggle button; when on, reveals a **Shape** choice (Area, Line, Pin), a label text box (up to 120 characters; the engine trims labels to 80), a colour dropdown (Primary, Amber, Cyan, Green, Red), a **Clear** button and a live hint line.
- While active it **owns the pointer**: ordinary selection (aircraft, camera feeds) yields, and the engine's own click-to-select and double-click-to-track are suspended and restored afterwards. A vertex clicked on top of an aircraft is just a vertex. If another tool owns the pointer the hint says that tool is using it.
- Interaction: click to add vertices placed on the real surface (the live preview "rubber band" follows the height under the cursor; finished marks drape on the ground and the height is dropped). **Double-click** or **Enter** finishes; **Backspace** removes the last vertex; **Esc** has two steps (resolved): with at least one vertex placed it discards the shape in progress and stays in Draw mode with a fresh empty shape of the same kind; with nothing placed it leaves Draw mode (the toggle turns off, the pointer is handed back, the preview disappears, the hint clears). Marks already finished are never touched by Esc. Escape is taken first, before any other Escape owner, and is consumed so nothing else reacts to that press (no deselect, no search close); it is ignored only while focus is in some other text box, but works from the label box. Enter finishes only when focus is on the map, the page body or the label box and a shape has been started; Enter on a focused button presses that button (e.g. Clear) rather than finishing. Backspace removes the last vertex unless focus is in the label box. A pin is placed with Enter or a double-click; further clicks move it. [OBSERVED-IN-SOURCE] Playwright-style note from testing: a bare click did nothing without a preceding mouse move; a double-click zoomed the camera outside Draw mode.
- Limits: Area needs 3 vertices, Line 2, Pin 1 (a later click moves the pin); at most **512 vertices** per shape; two clicks within 0.5 m are one vertex (so the finishing double-click doesn't add a stray point); a line shorter than 1 m or an area under 1 m2 is rejected as degenerate with a plain hint ("points are in a line - move one off it"); coordinates must be on the globe. Areas are auto-closed. Longitudes are handled continuously across the antimeridian.
- Hints (guidance by state, shown in a polite live-announcement line): before starting, "pick a shape, then click the map"; for a pin, "click where the pin goes" then "Enter to place the pin, Esc to cancel"; for lines and areas, "click N more points" until the minimum is reached; then the running measurement plus "double-click or Enter to finish, Backspace undoes, Esc cancels" (with a note when the 512-point limit is reached); a degenerate area says the points are in a line and one should move off it, a degenerate line says it has no length and to click further away; a click that adds nothing says either that the shape already has the maximum points (finish or Backspace) or that the point is off the globe; after a refused or failed finish, "that shape could not be placed" or "could not place the shape: <reason>"; and "board cleared" (shown only when Draw mode is off at the time of clearing). [OBSERVED-IN-SOURCE]
- After a shape is finished Draw stays on with a new empty shape of the same kind, the label box is emptied, and the shape is handed to the annotation engine as a persistent mark with no camera assist (hand-drawn marks never move the camera). Changing the shape type mid-shape discards the unfinished shape. Changing colour only recolours the preview. An area's drawn outline is closed automatically; a line becomes a route-kind mark that records its length (sum of great-circle segments) and no travel time. [OBSERVED-IN-SOURCE]
- The live preview is a dashed 3 px line in the chosen colour (dimmer where it passes behind scenery) with an 8 px dot at each vertex and a rubber-band segment to the pointer; an area preview also shows the closing edge once it has three points.
- Colours: primary light cyan, amber, cyan, green, red (preview colours match final colours).
- Clear removes **every** mark on the board (spoken and drawn).

### 5.3 Engine behaviour (whiteboard)
- Mark types: **pin** (highlight/point), **area** (footprint), **route**, **arrow** (connector with distance), plus labels. Unknown synonyms are normalised (vector/connector -> arrow).
- Persistence: marks persist until cleared by default; non-persistent ones live 22 s and fade over 1.2 s. At most **120** live marks. Duplicates are recognised by geometry (not label/colour); a duplicate refreshes and revives the existing mark. A persistent mark fades in over ~0.26 s.
- **Voice annotate arguments (census merge):** up to 24 marks per call (1-24), five colours, routes of 2-12 waypoints with a mode (walking, driving, cycling), a persist flag, and screen-fraction coordinates so a mark can be placed by pointing. Each mark may carry an intent ("the thing" versus "around the thing") and an entity kind (building, compound, district, street, point feature) plus a footprint flag. These steer which outline is sought: compound-sized outlines for airports, hospitals, universities, parks, stadiums and campuses; an envelope for districts. Airport-to-airport distance arrows rely on the compound-size outline centre. Annotations travel in share links and embed messages as at most 24 marks within 6,000 characters of compact encoding, drawn after the restore. [13e G8, ENV-91, ENV-102]
- Targets can come from a name (resolved through place search and, for buildings/compounds, an outline from OpenStreetMap), coordinates, or a screen position. Outline lookups that fail transiently retry after 8 s and 25 s; throttling is retried once honouring the server hint; a definitive "no polygon" keeps the honest point. Up to two outline upgrades run concurrently.
- Routes: waypoints are all resolved (a failure names the missing waypoints and aborts rather than silently dropping one); a street-following route (drive, walk or cycle) with distance and duration is requested from a public routing service; if routing is unavailable a clearly labelled direct line is drawn with great-circle distance and no travel time.
- Arrows: both endpoints must resolve; the label gets the straight-line distance appended after a dash.
- Renderers: areas and routes are draped on the 3D tiles/terrain in world space (animated flowing dashes for routes); pins, arrows and all captions are an SVG overlay re-projected every frame so marks track the camera, are culled behind the globe horizon, and use a hand-drawn "explainer" look: pulsing reticle (outer radius 34 px, inner 18 px, dot 5 px, scaling down from full size at 800 m camera height to 32% at 6 km and above), bowed arrows and glassy callout cards with leader lines. Area captions are placed at the centroid.
- Camera assist (voice and tool annotations only; hand-drawn marks never trigger it): when none of the newly drawn/refreshed marks is on screen, the camera gently frames them (single mark: its own framing; several: flight to a bounding sphere at -35 degrees with range at least 900 m or 2.6 x radius, 1.6 s). It never fires if any mark is already visible, never on a later outline upgrade, and stands down for 2.6 s after each assist to avoid stuttering. Region-scale targets are capped to roughly 290 km range.
- Annotations hold continuous rendering while any mark exists.
- All marks are world-anchored (lon/lat) so they persist across camera moves.

### 5.4 Import your own data (Display, "Import")
- Accepts GeoJSON/JSON, KML, KMZ and GPX files via a button or by dropping onto the globe; up to **12 files**, each up to **50 MB** (KMZ inflated up to 120 MB), at most **20,000** map objects and **250,000** positions per file, at most 2,000 style/link definitions per KML; names are trimmed to 120 characters. Files are read on the device only (stated in the control's hover); imported overlays can appear in map screenshots shared with voice. Each file gets the next colour from the whiteboard palette and a row with show/hide, fly-to and remove; a one-line hint reports each file's outcome. Rejections give plain-language limit messages.

---

## 6. Measurements

- **Draw measure (live):** while drawing a line, the hint shows the running length - metres below 1 km, kilometres above (one decimal under 10 km, none above); while drawing an area it shows the enclosed area in square metres, hectares (from 10,000 m2, one decimal) or square kilometres (from 1,000,000 m2, two decimals). Areas use a local planar approximation (fine at whiteboard scale); lengths use great-circle distance.
- **Voice arrow:** an arrow between two named places shows the straight-line distance in its caption; endpoints stay pinned to the world while orbiting.
- **Routes:** street-following distance (and travel time when real routing was used).
- **Camera feed footprint / viewshed** (coverage) is a separate measurement-like visual covered in the layers doc.
- **HUD derived measures:** window size in km, GSD, ONA, altitude (section 1).
- **Tracked-aircraft readout:** altitude in feet/FL and speed in knots.
- Units are metric for distance except aviation altitude (feet) and speed (knots); the HUD ALT is metres.

---

## 7. Recording and safe frames

- "Recording mode" is not a user toggle in the Display panel; it is turned on by **scene preview runs** (guided tours) and turned off when the scene stops or completes. **Corrected in the census merge:** no voice or automation tool switches it on; only scene runs do. The share link does not carry the recording state either; the flag appears only in the control-state readback. [13e correction 2]
- Options: *hide panels* (default true), *HUD mode* (default minimal; also full, off, or leave auto), *safe frame* 16:9 (default) or 9:16. **In practice:** starting a preview run always requests the 16:9 frame with the full HUD mode; the recipe's hide-panels, HUD-mode and safe-frame fields are not used for the recording chrome (only the HUD mode is converted into each shot's HUD setting, which then overrides the "full" request shot by shot). The 9:16 frame exists in the overlay, but nothing in the studied app asks for it; a recipe described as "9:16 vertical" therefore plays in 16:9. [13e correction 1]
- **Safe-frame geometry:** a full-screen, non-interactive layer above the map and below popovers; a 1 px outline at about 28% white with a 1 px dark halo; thirds guide lines at about 20% white. The 16:9 box width is the smaller of 90% of the viewport width and 90% of the viewport height times 16/9; the 9:16 box width is the smaller of 58% of the viewport height and 76% of the viewport width. Behaviour on non-16:9 screens is from style rules only (UNVERIFIED visually). [13e G3, ENV-105] Kawn: add 4:5 and A4 frames.
- Scene playback has no restore pass: layers a shot names stay as the last shot set them, only the scene's own release list is switched off at the end, and the user's HUD mode and layout are restored (map source and look are not described as restored). Kawn should offer an explicit "restore my view after the tour" option. [13e G4]
- Effects: nearly all chrome (title, style indicator, top cluster, loading chips, Display, Parameters, dock, layer panels, camera feed, Radio, Context, voice pill, Scenes, expanded street-level viewer) is hidden; the scene runtime overlay stays visible; a safe-frame guide box (16:9 or 9:16) is drawn to help composition; HUD layout set to Minimal (or Tactical for "full"), forced on or off, or auto. The attribution line stays on screen and moves to the bottom-left corner.
- The user's pre-recording HUD mode and layout are snapshotted once (re-entrant calls do not overwrite) and restored on exit; the safe-frame guide is removed. The HUD button reflects visibility.
- During scene playback the top-cluster camera buttons (clear layers, tilt, north-up, reset) are hidden because the scene owns sequencing. (The share link does not record whether recording was active; see above.)
- There is **no built-in screen capture**; recording means setting the screen up for external capture. The reference uses an in-page drawing buffer that is preserved so external tools can read frames.

---

## 8. Credits and attribution

- A single credit line sits at the **bottom-left** (inset 36 px from left and bottom), never hidden - not in Clean UI, not in recording mode (a deliberate fix after earlier footage shipped without credits). When the bottom dock is visible the line rises above it. Final position after the competing rules settle: about 2% of viewport height plus 5 rem above the bottom, 24 px from the left (12 px at 900 px wide and below), as wide as its content up to the screen width minus 24 px; the older rule that moved it to the bottom-right beside the Minimal coordinate block, and the 7 rem and 9.5 rem raised positions, are overridden by later rules and do not apply. [INFERRED from rule order; not seen live] With Clean UI or recording mode it returns to the 36 px bottom-left corner. The Cyber telemetry card leaves room for the full logo row (its bottom-left block is raised to about 2% plus 7.75 rem).
- Content: provider logos plus a short text credit for the active imagery (for example "Powered by Esri - Source: Esri, Maxar, Earthstar Geographics, and the GIS User Community"; "(c) OpenStreetMap contributors"; the Google logo and a "Data attribution" link while 3D tiles are shown; Bing credits from the ion service; the terrain provider's CC BY credit). Credits follow the source actually on screen, including after a fallback. A short inline OpenStreetMap credit (and OpenMapTiles when vector tiles are used) appears whenever a layer using OSM data displays.
- **"Data attribution" lightbox:** a link opens a panel (470 px wide, max height the smaller of 70% of viewport height and 36 rem; full-screen below 575 px) holding the complete, unabridged list of per-data-source credits. The title and Close stay fixed; the list scrolls with a stable scrollbar gutter and wraps long text. Keyboard: the link and Close are exposed as buttons (Enter or Space activates; a long Space press that hands off to push-to-talk cancels activation), the link reports expanded/collapsed, Escape closes the lightbox and returns focus to the link, clicking outside closes. Cockpit stays active behind the lightbox, and Escape belongs to the lightbox while focus is inside it.
- **Registered data credits (about 50):** OpenSky (non-commercial, with the research citation), adsb.lol (ODbL), adsbdb (and its upstream route-data authors), AISStream, CelesTrak, Launch Library 2, USGS, NASA GIBS/FIRMS, WFIGS wildfire perimeters, Overture-derived military names, Photon geocoder, Open-Meteo, a news RSS source, GDELT, a long list of public traffic-camera programmes (Austin, TxDOT, DelDOT, Caltrans, TfL, Ontario 511, Fintraffic, Calgary, Vegvesen, NYC DOT with its required verbatim disclaimer, DriveBC, Tallinn, NSW, Warendorf and others), GBFS bike-share, OSRM routing, GTFS-realtime transit, Radio Browser, Re:Earth terrain, NOAA weather products and cyclones, lightning, NOAA GFS and ECMWF IFS wind, dams, TeleGeography cables (non-commercial share-alike), TomTom traffic, Natural Earth, US Census counties, and event-specific packs. Entries are registered once at start-up as off-screen (lightbox-only) credits; a machine-readable data-source index must be kept in step.
- The credit surface is a compliance requirement for the data licences and for the Google and Cesium terms.
- Per-feature credit links exist too (for example "Weather data by Open-Meteo" in the cockpit local page).

---

## 9. Interaction with other systems

| System | Interaction |
|---|---|
| Looks | HUD colour/mode label; detection palette; HUD auto-visibility; Cyber auto-FLIR |
| Share link | HUD visible/layout; detection mode, density, allocation, fade, outside; scope on/feather/(terminus); celestial; look and parameters (not recording state, which only scene runs set and the link never carries) |
| Scenes | Per-shot HUD, detection, scope, look and bloom; suspend/restore detection around jumps; recording mode |
| Contacts/Cockpit | Detection forced on, bracket opacity reduced, HUD text kept (78% scale, 72% opacity, hidden at 760 px and below) while its on/off, layout and related controls are portalled into the cockpit Display popover; compass overlay uses keyhole |
| Voice | Set HUD on/off/auto and layout, detection density/mode/allocation/fade/outside, sonar settings, annotate, clear annotations |
| Clean UI | Hides panels but not HUD, detection or credits |

---

## Kawn notes

**Keep**
- A compact telemetry HUD concept, reinterpreted as a **city status strip**: place (neighbourhood/district), time, weather/heat, coordinates, view scale. Keep the cadence design (1 s clock, 250 ms camera values, 15 s summary) and the deterministic-first, AI-optional summary with strict "honest feed state" wording.
- Detection as **labelled objects with a stable, budgeted subset**: label budgets by zoom scale and density stops, elastic vs weighted layer allocation, collision grid, fade-in/out, minimum lifetime and cooldown anti-flicker, keyhole fade, and the accessible mirror list of visible targets.
- Draw/whiteboard with area/line/pin, live measurement, undo, Esc, double-click/Enter; same engine for voice and hand; geometric de-duplication; 120-mark cap; clear-all.
- File import (GeoJSON/KML/KMZ/GPX) with strict size/entity caps and "stays on your device" promise - valuable for planners and developers.
- Always-visible, keyboard-operable attribution and a full data-credit lightbox; compliance-first credits even in clean/recording modes.
- Safe-frame and recording-mode concept for creators.

**Skip**
- The espionage cosplay: fake classification banners, NIIRS, MGRS, KH-11 style ids, REC dot, ORB/PASS lines, "band/bits/level" edge text. They are incongruous with a civic product and unsuitable for a government audience.
- Cyber sonar (decorative; high GPU cost).
- The scanline/CRT blend on detection labels.

**Adapt**
- **Arabic and RTL:** mirror Tactical/Operator corner blocks (start/end instead of left/right); the circular scope, compass and bearing wheel stay geographic. Provide bilingual labels on detection cards (Arabic primary or English primary by setting) with bidi-safe isolation for mixed text (flight numbers, plate-like codes, Latin building names inside Arabic text) and Arabic-capable fonts; allow longer strings (Arabic names are shorter in glyph count but wider in pixels) - budgets in characters (18/26) must become pixel-width budgets. Western vs Eastern Arabic numerals toggle for all readouts. Avoid wide letter-spacing on Arabic text. Right-aligned callout cards with the accent bar on the right.
- **Locality line for UAE:** replace the eight-city landmark table with emirate / district / community tiers (for example Emirate > Area > Community > Landmark) and a sensible near-radius (a few km, not 150 km). Time uses Gulf Standard Time (UTC+4), not longitude arithmetic. Sun elevation and shade matter (heat); show temperature and humidity instead of imagery-quality metrics.
- **Detection budgets and density** need to be tuned for dense downtown clusters (Downtown Dubai, Marina, Al Reem Island, Corniche) and for mobile screens (smaller totals, larger labels, 44 px targets). Offer two stops plus "all" on mobile.
- **Tier colours** should be colour-blind-safe and carry icons/shape, not colour alone; use Kawn's own palette and keep red/green traffic semantics redundant with patterns.
- **Measurements:** metric by default with an option for imperial (expat users); support Arabic-Indic digits; add "walking time in summer heat" and "drive time now / evening" next to distances, with provenance and an "estimated" label as the reference does for routes.
- **Draw:** add shapes useful to planners (buffer circle, corridor, polygon with area in hectares and square feet, plot-ratio helpers), save/load named boards per project, and role gating; government users need an audit trail of annotations.
- **Grid reference:** MGRS is not meaningful to UAE civil users; if a grid line is wanted, use a local reference such as the Makani address system (Dubai) or a district grid. Region logic must be rebuilt: the reference labels Abu Dhabi as Africa.
- **Scope default:** owner decision is Scope on, feather 100%, Dubai on first load (docs/09); define "centre" as the free map area when panels are open and on phones; keep a one-tap off and a visible feather control.
- **Recording:** replace the cinematic recording mode with "present mode" and a share-as-image/PDF brief (messaging-app friendly); keep the 16:9/9:16 safe-frame idea for social and WhatsApp-status exports.
- **Credits:** UAE open-data licences (Dubai Pulse / Bayanat / DLD / RTA) have their own attribution wording; keep the lightbox pattern but support bilingual text and per-layer "who published this, when, official vs modelled" provenance cards.
- **Privacy/safety:** consider masking or coarsening objects near sensitive facilities and avoid showing military or restricted-airspace tags; the reference's "military" tier colour should not ship.

## Open questions / UNVERIFIED
Resolved in the revision pass and moved into the body: Escape in Draw (two-step: discard the shape in progress, else leave Draw); HUD block placement at narrow widths (section 1.7); leader-line geometry and card placement rules (section 3.6); Weighted allocation (when capacity covers every layer each gets one slot, otherwise the single slots go to the highest-weight layers first; Elastic with too little capacity serves layers in their listed order); safe-frame guides (corrected in the census merge: only scene preview runs switch recording mode on, so the guide never appears outside scene playback); HUD under reduced motion (the HUD typing effect and blinking dot have no reduced-motion handling; only the Cyber sonar sweep and cockpit animations honour it).
- Whether the HUD can be shown on a phone in a usable way (rules exist for narrow widths but the two top blocks are expected to collide below about 450 px; not seen live).
- Final on-screen position of the credit line and of the Minimal bottom block at dock-reflow widths (competing rules; later ones appear to win).
- Behaviour of the AI summary request policy gate (cost/rate limit): only its existence is visible here.
- The textual label for Cyber's HUD tag in non-English.
- Whether credits are legally sufficient when a Clean UI screenshot is exported (only the on-screen link is guaranteed).
- Per-layer label content for camera feeds, fires, road dots, bike docks and satellites was not re-read in the revision pass.

## Revision log
- Corrected the detection label: it is a single line (identifier plus one dim metric), not a two-line card with class and 18/26 character limits; documented the real four-diagonal placement, sticky corner, 4 px screen-edge rejection, chrome veto list, 32 px collision grid and 1 px leader geometry, and the separate placement/leader rules for rich world cards.
- Corrected the compositing order (HUD text sits beneath detection and cards, which are kept off the HUD text), the detection density rounding at exactly 75, the nearest-landmark table size (nine cities, not eight), and the Operator-versus-Tactical rail difference (collapsed right-rail launchers are hidden only in Tactical).
- Added section 1.7 (narrow-window HUD reflow, including the rule-precedence defect for the bottom bar and the cockpit corridor rendering), final dock-relative HUD and credit positions, Cyber desktop composition, and the Abu Dhabi region-label quirk.
- Draw: resolved the Escape behaviour, expanded hint states, finish and cancel semantics, preview look, and clarified that hand-drawn marks never trigger the camera assist.
- Resolved HUD auto-visibility and safe-frame questions; re-verified cadences, budgets table, allocation weights, bracket floor, scope numbers, import limits and annotation caps.

### Pass 3 (census merge: 13b, 13e, 13f)
- Corrections applied: recording mode is turned on only by scene preview runs (not by tools) and is not carried by the share link (7 and 9 and the open-questions text); a preview run always asks for the 16:9 frame with the full HUD mode and the 9:16 frame is never requested; the Share link row of the interaction table no longer lists recording state.
- Added: five zero-idle-cost rules for detection, with the aircraft-hold coupling (13f gap 6); tracked readout and box facts; the exact world-jump trigger for suspending detection; scope mechanism and old-link feather default (13f); safe-frame geometry; no restore pass after playback; voice annotate arguments, intent and entity-kind hints, link encoding limits (13e G8); AI summary provenance drop; cockpit detection subject exclusion (13b).
- Kawn notes added: owner decision of Scope on with feather 100% and the free-map-area definition of centre, local grid instead of MGRS and the region-label defect, safe-frame 4:5 and A4, restore-after-tour option.
- Confidence tags updated to cite the census files for touched text.

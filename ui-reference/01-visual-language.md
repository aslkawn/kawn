# 01 - Visual Language of the Reference App

**Purpose.** Describe how the original app looks, in enough measured detail that a designer can rebuild an equivalent (not identical) visual language from this document alone, then say honestly what Kawn should change. Evidence order: screenshots (cited as "shot NN"), the stylesheets (values only, nothing copied), then the earlier behaviour docs. Screenshot coordinates are in image pixels of a 1920 x 1080 frame. Shots 01-16 are captured at full UI scale; shots 17-29 look like the same UI at roughly 0.9 to 0.93 scale (smaller type, tighter dock), so quote ranges, not single numbers, when matching them.

Overall impression in one line: a dark, instrument-panel "mission control" skin. Near-black space, a single cyan accent, frosted dark glass panels with hairline borders, tiny tracked monospace capitals everywhere, and a faux-intelligence overlay (classification banner, grid reference, REC clock) laid over a photographic globe.

---

## 1. Colour system by role

### 1.1 Backgrounds and surfaces
| Role | Value | Where seen |
|---|---|---|
| Page / space background | near-black blue #0a0a0f | behind the globe, outside the circular scope, loading screen, empty corners (shots 07, 09, 21) |
| Glass panel fill | #0c0c14 at 72% opacity | left layer rail, right-hand panels, voice pill (shots 01, 17) |
| Panel under-detail | a faint cyan radial wash toward the bottom edge of floating bottom panels | dock glow (shot 02) |
| Opaque menu surface | #12121c | native dropdown lists (Layout select in shot 03) |
| Command dock fill | rgba(9,18,27) at 84%, slightly bluer than panels | bottom dock (shots 01, 02) |
| Top chips (loading, sync) | rgba(3,10,18) at 78% | "REFRESHING LIVE DATA", "LOADING FRAMES" (shot 01) |
| Detection / world card plate | deep teal-black rgba(4,12,16) at 82%; selected 94% | label cards on the map (shot 01, 26) |
| Cockpit entry button fill | rgba(2,12,18) at 90% | COCKPIT button (shot 25) |
| Inner control wells | page background #0a0a0f used inside panels (inputs, selects) | Layout select, sliders track |
| Idle tile / button fill | white at 3-4% | preset tiles, OFF toggles, chips |
| Hover fill | white at 6% | rows at 3% |
| Active fill | accent cyan at 15% | selected look tile, ON toggle |

### 1.2 Borders and hairlines
| Role | Value |
|---|---|
| Panel border | 1 px white at 8% |
| Panel border, hover | 1 px white at 15% |
| Control border, idle | white at 6-8% |
| Dock border | 1 px pale blue rgba(123,189,211) at 28%; internal dividers at 16% |
| Panel inner highlight | 1 px white at 6% along the top inner edge, plus a 1 px white-3% inset ring |
| Row divider (layer groups) | 1 px white at 10% |
| HUD frame lines (Operator boxes) | white at 16%, bars at 20-24% |
| HUD hairline default | HUD colour at 15% |

### 1.3 Text tiers
| Tier | Value | Use |
|---|---|---|
| Primary | #e8eaed | layer names, titles, values |
| Secondary | the primary at 50% | counts, group headings, inactive icons |
| Dim | the primary at 30% | micro labels, status sub-lines, units |
| Accent text | #00d4ff | active toggle label, "VIEW" in the title, active-style word, frame-rate readout |
| Soft cyan text | rgba(170-185, 245-252, 255) at 92% | loading chip label |
| Credit line | the primary at 55%, links at 70%, 1 px black text shadow | bottom-left attribution |
| Focus ring | solid #e8eaed 2 px, 2 px offset | every keyboard-focusable element (never removed) |

### 1.4 Accent family (cyan)
| Token role | Value |
|---|---|
| Primary accent | #00d4ff |
| Accent dim fill | #00d4ff at 15% |
| Accent glow | #00d4ff at 40% (text-shadow 20-30 px on title and active style; box glow 10-20 px on active controls) |
| Bright accent for cockpit / pressed states | #00dcff, #00e1ff |
| Light cyan for tracked card borders, world-card accent | #6be8ff |
| Draw primary | #8be9ff |
| Layer-list scrollbar thumb | vertical gradient 65% to 20% cyan, fully round |
| Active toggle border | #00d4ff (about 70% in secondary toggles) |

There is no second brand colour. Warm hues exist only as meaning (military, stale, warning).

### 1.5 Status colours
| Meaning | Text | Border | Fill |
|---|---|---|---|
| Live / OK / ON | #00d4ff | #00d4ff | cyan 15%, 10 px cyan glow at 15% |
| Stale / partial / FALLBACK | #ffd27a | amber at 75% | amber rgba(255,174,51) at 10% (shot 22 "FALLBACK" badge) |
| Degraded | #ffad72 | orange at 78% | orange at 10% |
| Unavailable / error | #ff8585 | red at 78% | red at 10% (shot 25 "UNAVAILABLE") |
| Loading | cyan text, outlined badge wider than ON (82 px min) | cyan | transparent (shot 01 "LOADING") |
| Listening (voice) | #5cffc6 | mint at 45% | none |
| Executing (voice) | #ffdc78 | gold at 55% | none |
| Voice error | #ff5c6e title, #ffb4bd detail | red at 60% | none; capsule becomes a rounded panel |
| Recording dot | #ff3333, 6 px red glow | none | none |
| Voice-focus row highlight | none | mint at 72% inset | mint at 10%, 18 px glow |
| Dense detection button (ON state of the Dense row) | green text, border green rgb(0,255,80) at 60% | | green at 10% (shots 01, 03) |

### 1.6 Per-layer and marker colours
Colour is attached to meaning, not to layer chrome: layer rows themselves are neutral (emoji-style coloured icon, white name, dim meta, one shared ON/OFF pill). Colours appear on the map.
| Thing | Colour |
|---|---|
| Civil aircraft, civil label tier | #22e0ff cyan; trail #00d4ff |
| Military aircraft, label tier | #ffb347 (3D tint and trail #ffb800) |
| Sea / vessels | #3fe0c8 teal; trail #39ffd5 |
| Space / satellites | #bda4ff violet |
| Vehicle / generic ground | #8fa6b4 slate |
| Road traffic: jam / slow / free / no data | #e05252 / #f0b23e / #2ecc71 / #c9c9c9 |
| Transit: bus / tram / subway / rail / ferry / unknown | #5EF08A / #FFC24A / #FF4538 / #D9A6FF / #5FD6FF / #D8DDE5 |
| Tracked aircraft | solid cyan glyph, cyan leader line and cyan 3D model (shots 26, 27, 28) |
| Selected-contact bracket | cyan; emergency squawk tint #ff4444 |
| Aircraft near unselected | pale grey-white icons with soft glow and cyan corner brackets (shot 23) |
| Data-centre pins | cyan dot with a short cyan stalk (shot 01); stalk becomes a thick pale-cyan vertical bar when tall (shot 16) |
| Undersea cables | thin cyan lines on the globe (shot 09) |
| Earthquake markers | magnitude chips "M5.6" with a red or amber left edge bar (shot 09) |
| Radio stations | green dots with grey name plates; large green discs for clusters (shot 09 "58 MUSIC") |
| Great-circle / orbit lines | thin red (shot 08 diagonal) and cyan streaks (shot 21) |
| Draw / annotation palette | #8be9ff primary, #ffb547 amber, #39d0ff cyan, #5dff9f green, #ff6b6b red |
| Celestial sun / moon | sun rgba(231,194,72) at 92% with gold glow; moon rgba(55,211,239) at 84% with cyan glow; ring hairline rgba(151,218,232) at 9.5% |

### 1.7 Per-look HUD colours (full table; look pictures are in file 02)
| Look | HUD text | HUD glow | HUD hairline |
|---|---|---|---|
| Normal, Anime, Noir, Snow | cyan rgb(0,255,255) at 60% | cyan at 40% | cyan at 15% |
| CRT | amber rgb(255,170,0) at 80% | amber | amber |
| NVG | green rgb(51,255,51) at 80% | green | green |
| FLIR | white at 70% | white | white |
| Cyber layout (any look) | near-white text, muted red #c45454 border/glow, bright red #e16666 | | |
The HUD text colour is the only chrome that changes with the look; panels, dock and chips stay cyan/blue in every look (shots 03, 04, 05).

### 1.8 Cyber skin (alternative shell, shot 19)
| Role | Value |
|---|---|
| Accent | desaturated red #c45454, bright #e16666 |
| Slate base | #26343a, raised #334a53 |
| Panel fill | rgba(18,24,27) at 92% |
| Grid lines | pale blue at 10% |
| Text | light grey to near-white (#f5f7f6 family) |
| Borders on hover | red |
| Shape | chamfered (cut-corner) frames for panels, buttons, bottom bar |
| Panel widths | 272 px left, 312 px right, 200 px collapsed |
Everything chrome turns red/slate while map imagery keeps its colours. ON toggles become red-outlined with red text (shot 19).

---

## 2. Typography

### 2.1 Families
| Family | Weights | Role |
|---|---|---|
| JetBrains Mono (fallback: SF Mono, Fira Code, system monospace) | 300-700 | title, all micro labels, HUD, counts, chips, cards, buttons |
| Inter (fallback: system UI sans) | 300-600 | layer names (13 px), preset tile labels (11 px), prose, panel body text |
| Material Symbols Outlined (subset of about 35 glyphs) | 400 | top action cluster, panel launchers, celestial markers, cockpit button |
| Colour emoji | n/a | layer-list icons (satellite, plane, medal, dish, ship, car, bus, bike, camera) and some look tiles (moon, sparkle, snowflake) |
The reference loads fonts and icons from a public service at runtime; offline it falls back to system fonts and loses icons.

### 2.2 Size / weight / tracking table
| Element | Size | Weight | Tracking | Case | Colour |
|---|---|---|---|---|---|
| Title "GOD'S EYE VIEW" | 26 px | 600 (the word VIEW is 300 and cyan) | 8 px | upper | primary, 30 px cyan glow |
| Tagline "NO PLACE LEFT BEHIND" | 11 px | 400 | 5 px | upper | dim |
| Frame-rate readout | 11 px tabular figures | 400 | 1 px | n/a | cyan |
| "ACTIVE STYLE" label | 10 px | 400 | 3 px | upper | dim |
| Active style value (NORMAL, NVG...) | 16 px | 500 | 5 px | upper | cyan, 20 px glow |
| Panel header title (DATA LAYERS, DISPLAY, CCTV) | 10-11 px | 600 | 1.1-3 px | upper | dim, followed by a 1 px cyan-tinted line that runs to a small square collapse button |
| Layer-group heading (MOVEMENT, CAMERAS) | 10 px | 600 | 1.4 px | upper | secondary |
| Layer name | 13 px | 500 | 0.3 px | as written | primary |
| Layer meta line (source, "never", age) | 9 px | 400 | 0.5 px | as written | dim |
| Layer count (3.9K, 247) | 11 px | 400 | 0.5 px | n/a | secondary |
| ON/OFF pill | 9 px | 400 | 1.5 px | upper | dim; ON cyan |
| Sub-chips (mode chips, legends) | 8 px | 400 | 1.2 px | upper | dim |
| Look tile label | 11 px base; shots show uppercase small mono about 9 px | 500 | 0.3 px | upper in shots | secondary; primary when active |
| Look tile key hint | 8 px mono | 400 | 0 | n/a | dim, only on hover/active, top-right of tile |
| Dock labels (LOCATION, VISUAL PRESETS) | about 9-10 px | 400 | about 2 px | upper | primary at 70% |
| Loading / sync chip | 8 px | 400 | 1.35 px | upper | soft cyan; value part bold #00e1ff at 1.6 px |
| Voice pill status | 10 px / detail 9 px | 400 | 0 | as written | cyan / secondary |
| HUD classification line | 12 px | 600 | 2.5 px | upper | HUD colour |
| HUD system ids | 13 px, 80% | 400 | 2 px | upper | HUD colour |
| HUD mode label | 16 px | 700 | 4 px | upper | HUD colour |
| HUD summary caption "SUMMARY" | 9 px, 55% | 400 | 1.8 px | upper | HUD colour |
| HUD summary text | 11 px, 88% | 400 | 1.2 px | upper, single line, ellipsis | HUD colour |
| HUD corner readouts | 13 px | 400 | 2 px | upper | HUD colour |
| HUD REC line | 13 px | 400 | 2 px | upper | HUD colour; dot 14 px red |
| HUD orbit/pass line | 12 px, 70% | 400 | 1.5 px | upper | HUD colour |
| HUD edge texts / bars | 10 px, 50% | 400 | 1.4-1.5 px | upper | HUD colour |
| Detection label (ambient) | 10 px mono, micro field 90% of that | 400 | 0 | as written (callsign style) | tier colour on dark plate |
| Tracked / world card title | 12-13 px | 600 | 0 | as written | near-white 96% |
| Tracked card detail | 10-11 px | 400 | 0 | as written | blue-grey 92% |
| Cockpit entry button | 10 px | 600 | 0.14 em | upper | #dceeff |
| Credit line | 10 px | 400 | 0 | as written | 55% |

### 2.3 Rules that define the voice
1. Mono capitals with generous tracking for anything "instrument": headers, status, HUD, chips. Sentence case only for layer names, card content and prose.
2. Hierarchy comes from opacity (100 / 50 / 30%) and tracking, not from size or weight. Almost everything is 8-13 px; the largest text on screen is the 26 px title.
3. Numbers use mono and tabular figures; counts are right-aligned in a fixed 30 px column; readouts zero-pad (three-digit headings, five-digit altitudes) and "roll" digit by digit in the cockpit.
4. HUD text always has a 4 px soft glow in its own colour and is forced to capitals by style rather than by content.
5. Status words use a short fixed vocabulary: OFF, ON, LOADING, ENABLING, FALLBACK, UNAVAILABLE, STALE, SNAPSHOT OK.

---

## 3. Iconography
- **Two icon languages.** Outline symbols (Material Symbols Outlined, 18 px in the top cluster, 16-20 px elsewhere) for actions and controls; colourful emoji-style glyphs for layer identity in the left list. The mix reads as slightly inconsistent; the emoji set is the most "consumer" element in an otherwise austere UI.
- **Top action cluster (shots 01, 02):** five 36 px circles at the top centre with 8 px gaps: clear layers (a slashed layers glyph), copy link (chain), tilt / orbit-view (a hexagon-frame glyph, highlighted cyan when on, as in shots 13, 22), north-up (an arrow with a tiny cyan "N" above it that rotates with the camera heading), reset globe (globe). Idle: glass fill, 1 px 8% border, secondary-colour glyph; hover scales 1.1 and brightens the border; press scales 0.95; active adds a cyan border, cyan glyph and a 14 px cyan glow.
- **Look tray tiles:** circle (Normal), grid square (CRT), crescent moon (NVG), a thermometer-like red glyph (FLIR), four-point sparkle (Anime), half-filled circle (Noir), snowflake (Snow). Icons are 20 px, secondary colour; active icon goes cyan with a 12 px glow.
- **Map glyphs:** aircraft are white or pale silhouettes in a top-down plane shape (shots 23, 26-29); tracked plane solid cyan; helicopters and small craft get their own silhouettes; cameras are tiny cyan camera-body icons drawn at density in clusters (shot 01); vessels triangle; ALPR, data centres (square-in-square), cables (wave lines), dams, earthquakes, rockets, fires use small coloured glyphs in the list.
- **Reticles:** four L-shaped corner brackets (thin 1 px, cyan), segments about 55% of the half-side, drawn in sets around any detected object (shots 03, 16, 23).
- **Brand mark:** a circular "eye + globe" logo, 52 px wide, with a 12 px cyan drop-shadow; the iris follows the pointer on desktop; when radio is broadcasting, two thin semicircular arcs pulse above it (1.8 s).
- **Keyboard hints:** tiny bordered "kbd" chips (4 px radius, 25% pale border, 60% text) inside buttons such as COCKPIT.

---

## 4. Spacing scale, radii, layout units
| Token | Observed values |
|---|---|
| Base steps | 2, 4, 6, 8, 10, 12, 16, 20, 24 px; 36 px outer margin |
| Screen margin | 36 px from left and right for title, HUD blocks, credits and voice pill; title and style indicator 32 px from top |
| Panel inner padding | 12 px; list scroller 3 px; row padding 8 x 10 px (6 px bottom) |
| Gap between stacked panels | 8 px (right rail) |
| Gap between top-cluster buttons | 8 px |
| Tile gap in look tray | 8 px; tile padding 14 / 16 / 10 px; min width 78 px |
| Dock padding | 6 px, bottom offset 2% of height, width min(94% of viewport, 1240 px) |
| Left rail width | 272-300 px expanded (shots show about 280), collapsed pill about 176 px; left offset 52 px; top about 26% of height |
| Right rail width | 330 px column; Display panel about 272 px; right offset 52 px |
| Radii | panels 16 px; buttons and tiles 10 px; dock 12 px; chips 7 px; ON/OFF pill 4 px; sub-chips 3 px; HUD Operator boxes 8 px; world cards 4 px, ambient label plates 3 px; round buttons and voice pill 999 px; cockpit button 8 px |
| Row height | layer row about 50 px (two lines); group heading block about 44 px with its divider |
| Touch targets | most controls 30-38 px tall; ON/OFF pill about 22 px tall (below accessibility minimum) |

---

## 5. Elevation, glow and blur
| Element | Blur | Shadow / glow |
|---|---|---|
| Glass panels, voice pill | backdrop blur 24 px, saturate 1.4 | 0 8 32 black 50%, plus 20 px cyan glow at 8% on the voice pill |
| Command dock | blur 28 px, saturate 1.35 | 0 12 36 black 58%, plus 28 px cyan at 6% |
| Top-cluster buttons | blur 16 px | none at rest |
| Loading chips | blur 14 px | 0 8 24 black 34% plus 14 px cyan at 8% |
| Title | none | 30 px cyan text glow; faint radial cyan glow behind (200 x 80 px, blurred 20 px, 60% opacity) |
| Active controls | none | cyan 10-20 px glow at 14-18% |
| Cockpit button | none | 18 px cyan at 14% |
| HUD text | none | 4 px own-colour text shadow |
| Detection overlay | screen blend, 3 px soft glow | scanlines at 4-9% |
There is no stacked elevation system (no 1-2-3 levels). Depth is expressed by blur plus a 1 px border, and by a z-order ladder: scope mask, celestial ring and HUD, detection surface, world cards, credits, panels, dock and cockpit, voice pill, lightbox and toasts, clean-view exit.

Motion tokens: 150 ms for hover and press, 300 ms for layout, both on a standard ease; opacity fades 180 ms; HUD fade 400 ms; look crossfade 500 ms; camera flights 1.4-4.2 s; REC dot blink 800 ms; live pulse 1.6 s; summary types two characters every 24 ms.

---

## 6. Imagery treatment
- **The globe is the hero.** Photographic satellite imagery fills the whole viewport edge to edge; chrome floats over it. Default source is a satellite basemap (shots 01-02); photoreal 3D city tiles appear in the Gulf demo shots (22, 24).
- **Scope darkening.** Outside a centred circle (radius about 1.05 x half the viewport height) imagery falls off to near-black, with a soft feather (default 11%). Shots 01 and 05 show the soft default; shot 07 (feather 0, scope on) shows a hard circular crop; at globe altitude the corners hold faint stars. This makes every screenshot look like looking through a lens and leaves the screen corners clean for text.
- **Imagery is never tinted by chrome.** Only looks filter the picture (file 02); panels sit above in true colour.
- **Photo thumbnails** (CCTV feed in shot 01): a 16:9-ish rounded frame (about 290 x 160 px, 4-6 px radius) with a burned-in timestamp and a camera name at the bottom, then a small status pill "SNAPSHOT OK" and one dim line of metadata.
- **Labels over imagery:** dark translucent plates keep text legible; plates fade to 22% when over open sky; leader lines (1-1.35 px) connect plate to dot.
- **Credit line is part of the picture:** a Cesium ion logo mark plus underlined "Powered by Esri", OpenStreetMap and "Data attribution" links, always bottom-left at 36 px inset, 10 px mono at 55% (shot 01).

---

## 7. Cosmetic overlays (the "spy-satellite" layer)
| Overlay | Content and look | Position (1920 x 1080, Tactical) |
|---|---|---|
| Classification banner | text in the style of a top-secret marking, 12 px / 600 / 2.5 px tracking, HUD colour | top-left block, x about 59, y about 163 (shot 03) |
| Mission and sensor ids | two codes "KH11-nnnn OPS-nnnn", 13 px at 80%; random per session | directly under the banner, y about 186 |
| Mode label | current look name, 16 px bold, 4 px tracking | y about 213 |
| Summary caption and line | "SUMMARY" 9 px at 55%, then one 11 px line such as a view band + nearest landmark + region + altitude + sun elevation; types in two characters at a time | y 240 and 257, single line, ellipsis at the block's 420 px limit |
| REC clock | red 14 px dot (blinks, 800 ms) then "REC" and a UTC date-time with Z suffix, 13 px / 2 px tracking | top-right, right-aligned to x about 1860, y about 165; dot at x about 1610 |
| Orbit / pass line | "ORB: 47351 PASS: DESC-119" at 12 px / 70% | y about 186 under REC |
| Corner brackets | large (22 px) corner glyphs at 60% opacity, one beside each corner block | Tactical only (shots 03, 05) |
| MGRS and coordinates | grid reference then latitude / longitude in degrees-minutes-seconds with two decimals | bottom-left, x about 59, y 918 and 939 |
| GSD / NIIRS, ALT / SUN, AIS | three right-aligned lines, 13 px | bottom-right, right edge about 1860, y 891 / 913 / 939; AIS is dim and shows a dash |
| Vertical edge texts | "COLL: <time>", "ONA: <angle>" rotated -90 degrees on the left at x about 35; "BAND: PAN  BITS: 11  LVL: 1A" rotated +90 degrees on the right at x about 1885; both 10 px at 50% | vertically centred, y about 340-540 |
| Operator top banner | classification dim, mission id centred and brighter, "PAGE 1/1" dim; 10 px, 1.4 px tracking | centred, y about 109-131 (shot 17) |
| Operator / Cyber bottom bar | MGRS, LAT, LON in one 10 px line | centred, y about 895-916 |
All values are fiction derived from camera pose and clock. Counts, ids and the "descending pass" number are random per session.

---

## 8. Logo and brand treatment
- Wordmark: "GOD'S EYE VIEW" set in the mono at 26 px, 8 px tracking, three words on one line; the third word is thin (300) and cyan, the first two are semibold off-white. Eye-and-globe symbol 52 px to the left with a cyan glow, 12 px gap. Under it, a tagline in 11 px dim caps, 5 px tracked ("NO PLACE LEFT BEHIND").
- In Cyber the whole wordmark turns white with a red-tinged symbol and the tagline red (shot 19).
- Active-style indicator mirrors the wordmark on the right edge (label 10 px dim, value 16 px cyan).
- Narrow widths: at 720 px the title shrinks to about 16 px type and a 36 px symbol; at 520 px the second and third words and the tagline disappear leaving symbol plus first word.
- Third-party credits (Cesium ion, Esri, OpenStreetMap) are baked into the bottom-left and are legally required; nothing may hide them.

---

## What makes it feel the way it does
1. **One accent on near-black.** Cyan is the only chromatic UI colour; the imagery supplies all other colour. That restraint is why a very busy screen still reads as one system.
2. **Hairlines, not fills.** Almost every container is a 1 px 8% border on 72% glass; solid fills appear only for the active state.
3. **Tiny tracked caps.** 8-11 px mono with 1.4-5 px tracking creates the "instrument" voice; widening tracking is how the design signals hierarchy.
4. **Opacity as hierarchy.** 100/50/30% text tiers and 60/80% HUD tiers replace colour and weight changes.
5. **Glow as state.** Selection and live state are a cyan glow plus a coloured border, never a thick fill.
6. **The scope circle.** Darkening everything outside a circle frames the picture like optics and leaves clear dark corners for the HUD.
7. **Fiction as texture.** The classification banner, REC clock, orbit ids and grid reference carry no function but supply atmosphere.
8. **Constant micro-motion.** Blinking REC dot, typing summary line, rotating compass, pulsing logo arcs keep the screen "alive".

---

## Kawn improvements
| Area | Problem in the reference | Kawn direction |
|---|---|---|
| Tone | military cosplay (classification, MGRS, NIIRS, REC, orbit ids, red crosshair language) | civic, calm: replace the banner block with a neutral place card (area name, time, data freshness); drop fake ids entirely; keep the REC dot only in an explicit "recording" state |
| Legibility | 8-10 px mono caps at 30-50% opacity, white-on-photo | minimum 12 px for any readable text, 11 px only for decorative ticks; raise dim text to at least 4.5:1 contrast on the glass fill; make the glass fill 85-90% in sunlight/high-contrast mode |
| Touch | ON/OFF pill about 22 px tall, 36 px round buttons, hover-only tooltips | 44 px minimum targets, tap-and-hold or visible labels instead of hover |
| Mobile | floating rails over the map (rails become edge scroll bands at 720 px) | bottom sheet for layers and details (collapsed / half / full), top search bar, floating compass and reset; keep glass but opaque enough for sun glare |
| Light theme | dark only | add a warm paper light theme (backgrounds near #f6f4ef, glass white at 78%, text near #14202b, accent a deeper teal for AA contrast); dark remains for night and projection |
| Arabic / RTL | tracked mono capitals, left-weighted layout, no Arabic face | pair an Arabic UI family with the Latin mono/sans; drop tracking on Arabic; replace capital micro-labels with size/weight hierarchy; mirror rails, dock order, sliders, chevrons; keep compass, scope circle and map true; numeral setting for Western or Eastern digits |
| Colour semantics | amber/red used for both military and error; green for traffic-free and for radio and for Dense | reserve red for errors and alerts only; give each data family one hue from a restrained 6-8 colour categorical set tested for colour-blind safety, with a shape or icon second channel |
| Brand | cyan glow title with fake-intel tagline | calm wordmark in both Arabic and Latin, no glow on text, one accent used sparingly (selection, primary action) |
| Icons | outline symbols mixed with colour emoji | one icon family, two weights (outline / filled for active), UAE-relevant glyphs (tower, metro, palm, camera, dust), emoji only in user content |
| Attribution | always visible bottom-left (good) | keep the rule; make it a compact "Data and credits" chip that opens a sheet |
| Elevation | blur only, no scale | define three levels (map overlay chips, panels, modal sheets) with consistent shadow and border steps |
| Motion | constant decorative loops | keep state-change motion (150-300 ms); make decorative loops (blinks, typing, sweeps) opt-in and honour reduced-motion everywhere including the REC dot and typing effect, which the reference does not |
| Contrast of the scope | corners go to full black | keep as an optional "focus" mode; default to a full rectangular map in civic use |

---

## UNVERIFIED
- Exact computed look of the look-tile label (shots show small uppercase mono; the base rule is an 11 px sans at normal case; a later rule probably wins).
- Whether the shots 17-29 scale difference is a different viewport size or a UI zoom; sizes are quoted as ranges where it matters.
- Exact line height and padding of collapsed panel mini-status lines.
- Final font size of the dock labels and whether the dock tray labels use 9 or 10 px type.
- No shot of the Minimal HUD layout was supplied (shot 18 is absent from the folder); its placement values come from the stylesheet only.
- Cockpit visuals come from stylesheet values and descriptions, not from a static screenshot.
- Light-mode glyphs of sun and moon (stated in earlier docs) were not observed.

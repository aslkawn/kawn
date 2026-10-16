# 04 - Components and states (catalogue)

**Purpose.** Catalogue every reusable interface component of the reference product with its look, size, state variants and motion, precisely enough that a designer can draw an equivalent set. Geometry of the whole screen is in file 03; this file is the part library.

**Evidence.** Screenshots 06, 10 to 12, 17, 19, 22 to 36 (describe what is visible), plus the product's stylesheets for numbers (sizes, radii, opacities, timings). Colours are hex or rgba with their role. No code or selectors are reproduced.

**Global tokens used throughout** (so the tables can stay short):

| Token (role) | Value |
|---|---|
| Page background | #0a0a0f |
| Glass surface | rgba(12,12,20,0.72) over the globe, blur 24 px, saturate 1.4 (chips use blur 16 px) |
| Opaque menu surface | #12121c |
| Hairline | 1 px white at 8%; hover 15% |
| Accent | #00d4ff; dim fill 15%; glow 40% |
| Text | primary #e8eaed; secondary 50%; dim 30% |
| In-world card accent | #6be8ff |
| Warning amber | #ffd27a text, rgba(255,190,82,0.75) border, rgba(255,174,51,0.10) fill |
| Degraded orange | #ffad72 text, rgba(255,137,72,0.78) border, rgba(255,112,48,0.10) fill |
| Error red | #ff8585 text, rgba(255,92,92,0.78) border, rgba(255,60,60,0.10) fill |
| Detect/ok green | rgba(0,255,80,0.9) text, 0.6 border, 0.1 fill |
| Street-level green | #05cb63 |
| Dock family colours | bar rgba(9,18,27,0.84), border rgba(123,189,211,0.28), tray rgba(8,20,30,0.94) |
| Fonts | JetBrains Mono for labels, values, chips; Inter for names and prose (13 px names, 12 px selects, 0.72 rem captions); Material Symbols Outlined for round-button and tile icons; layer icons are small coloured emoji-like glyphs |
| Radii | panels 16; buttons 10; rail toggles 8; chips 3 to 4; segmented 6 outer; pills 20 or 999; round buttons 50% |
| Motion | fast 150 ms, smooth 300 ms, both with a standard ease-in-out curve (0.4, 0, 0.2, 1) |
| Focus ring | 2 px solid #e8eaed, offset 2 px outside or minus 3 px inside when the parent clips; never removed |

---

## 1. Buttons

### 1.1 Round icon button (top-centre row)

| Attribute | Value |
|---|---|
| Look | glass circle, 36 x 36, 1 px hairline, 18 px outline icon in secondary text colour; blur 16 px |
| Icons (left to right) | "layers off" (clear selected layers), link (copy share link), tilt/atom-like view (tilt map), compass with tiny "N" caption (north up), globe (reset to full globe) |
| Compass | tiny cyan "N" 8 px bold 2 px from the top; arrow rotates opposite to the camera heading in 150 ms |
| Default | as above; gap 8 between buttons |
| Hover and keyboard focus | border to 15% white, fill white 6%, scale 1.1 |
| Active (pressed) | scale 0.95; toggle buttons in the on state (tilt) show accent border, accent icon, 14 px cyan glow at 18% (shot 12, 22, 23: the tilt button is ringed in cyan) |
| Disabled / busy | opacity 0.55, wait cursor, no scale; remains focusable |
| Loading | label changes for assistive tech only (visual unchanged) |
| Error | none shown |
| Motion | colour, border and transform 150 ms |

### 1.2 Display toggle row button (full-width)

| Attribute | Value |
|---|---|
| Look | full panel-width row, minimum height 38, 7 x 12 padding, 8 radius, 1 px hairline, glass fill, icon 14 px then 9 px mono caps label with 1.5 px tracking in secondary text; examples: HUD, DENSE, 3D, SCOPE, DRAW, CELESTIAL, CLEAN UI, BLOOM, SHARPEN (shots 06, 22) |
| Default | dim label, no glow |
| Hover | hairline 15%, fill white 6% |
| Active (on) | accent border, accent 15% fill, 12 px cyan glow at 15%, label turns accent (HUD, SCOPE, 3D, SHARPEN in shots) |
| Active for detect/dense | green variant (border 60%, fill 10%, glow 12 px); the strongest modes (heavy density) add a 2 s pulse that swells the glow from 12 px to 22 px (green) |
| Focus | 2 px white inset ring (offset minus 3) |
| Disabled | 42 to 50% opacity, not-allowed cursor (e.g. Celestial outside the Normal look) |
| Row pairing | an on toggle can grow a sub-row beneath it (slider, segmented control) that visually joins it: the toggle's lower corners square off and the sub-row has a 0-top border and 8-radius lower corners |
| Pitch | 38 button + 7 gap = 45 vertical rhythm |

### 1.3 Text action button (scenes, camera, calibration)

| Attribute | Value |
|---|---|
| Look | 9 px mono caps, 1.2 px tracking, 6 x 9 padding, 7 radius, white 5% fill, 1 px white 14% border, secondary colour |
| Hover | text, border accent; fill accent 14% |
| Danger hover | text and border #ff7272, fill red 15% |
| Disabled | opacity 0.45, default cursor |
| Row layout | START / STOP / NEXT stretch to equal width (flex); CAPTURE SHOT and UPDATE SHOT pair; EXPORT PRESETS and IMPORT pair (IMPORT drawn with an accent border in shot 33 to mark it as the action with a pending effect) |
| Small variant (shot rows) | 8 px caps, 4 x 6 padding, 6 radius |
| Primary solid (settings dialog "SAVE KEYS") | vertical gradient from #45e0ff to #1fb9dd, text #04141c 10.5 px bold with 1.5 px tracking, 6 radius; hover brightness +8%; busy 50% with no pointer |

### 1.4 Icon-only small button

22 x 22 (panel collapse square) or 20 x 20 (Display header): 6 radius, 1 px white 12% border, white 3% fill, dim glyph (a "+" or "-" on the left rail, a triangle on the right rail); hover raises border to 15%, text to primary, fill white 7%. Import row actions are the same at 22 px with 14 px icon; hover border turns accent. Round viewer close: 26 x 26 circle, 1 px white 25%, hover inverts to white fill with dark glyph (34 px in the expanded viewer).

### 1.5 Segmented control

| Attribute | Value |
|---|---|
| Look A (Display: ELASTIC / WEIGHTED, PROXIMITY / ALL, AREA / LINE / PIN) | height 22, 6 radius outer, 1 px white 14% border, equal-width segments divided by 1 px lines, 9 px caps text, dim idle |
| Active | solid accent fill with black text (shot 06: WEIGHTED, PROXIMITY, AREA are bright cyan blocks) |
| Hover | segment fill white 8%, text primary |
| Focus | 2 px white inset ring plus a 4 px dark inner shadow so the ring stays legible on a bright segment |
| Look B (street-level IMAGERY ALL / 360 / FLAT) | pill outline (999 radius), 9.6 px mono, 4 x 9 padding, active segment green 22% fill with primary text |
| Disabled | whole group at 45% opacity |

### 1.6 Chips and toggle chips

| Chip family | Size and look | States |
|---|---|---|
| Layer sub-chip (DENSE, SHOW NEAREST, road source, MORE DETAIL) | 8 px caps, 1.2 tracking, 2 x 7 padding, 3 radius, white 4% fill, 1 px white 8% border, dim text | hover/focus: border 15%, fill 6%, text secondary; active: accent border and text on accent 15%; busy: 70% opacity, wait cursor; loading: border white 18%, text secondary; error: red border 60%, red text on red 8% fill |
| Provider chip (street level) | same, with a 7 px dot at the left in the provider colour (idle dot 45% opacity) | active: border, text and a 14% tint all in the provider colour (Mapillary green, KartaView violet, Google its own); error: amber border and text |
| Map-source chip (dock tray) | 9 px caps, 0.6 tracking, 4 x 7 padding, 4 radius, five equal columns with 4 px gap (three columns under 620 px wide) | active: accent text/border, 15% fill, 10 px glow; unavailable: text 28%, no hover, not-allowed cursor, small "ION" badge 7 px |
| Time-step chip (imagery, timeline) | height 22, 0 x 4 padding, ellipsised | disabled 40% |
| Legend swatch | 6 px dot with a 1 px white 18% ring, 8 px mono caption | no states |
| Calibration value chip | 9 px mono, cyan-grey text #91edff, 4 x 7 padding, 7 radius, white 4% fill; click turns it into a 58 px number input with accent border | hover: accent 45% border |
| Quality badge | 90 min width, 9 px caps, 8 radius; calibrated: green text #adffbf with green tint; curated: yellow; edited: amber; default cyan | |
| Source badge | pill, 8 px caps, accent 32% border; loading amber; error red | |

### 1.7 Layer state chip (ON / OFF and status words)

| Attribute | Value |
|---|---|
| Size | 22 high, 3 x 10 padding, 4 radius, 9 px mono, 1.5 px tracking; status words (LOADING, DEGRADED, STALE, PARTIAL, FALLBACK, UNAVAILABLE) widen it to at least 82 and tighten tracking to 0.8 |
| OFF | dim text, white 4% fill, white 8% border |
| ON | accent text and border, accent 15% fill, 10 px glow at 15% (shot 06: Transit, Bike Share) |
| LOADING / ENABLING | grey or accent outline, same fill as OFF with accent text |
| STALE, PARTIAL, FALLBACK | amber trio (text #ffd27a) |
| DEGRADED | orange trio |
| UNAVAILABLE | red trio, row cannot toggle |
| Hover | border 15% |
| Focus | white inset ring |
| Busy | stays focusable, ignores clicks |
| Motion | colour, fill, glow, min-width: 150 ms (the chip grows smoothly as the word changes) |
| Rule | when a status word is showing, the count to its left is hidden for rows whose count is not meaningful (ALPR) |

---

## 2. Sliders

| Attribute | Value |
|---|---|
| Rail slider (Display: density, fade, outside, feather, sharpen, bloom) | native range with 3 px track (white 8%, 2 radius), thumb 10 px circle in accent with 6 px accent glow; row = optional 8 px dim caps label (left), track (flex), 9 px cyan value (right, min width 28, right aligned) with "%" |
| Parameter slider (look parameters) | track 3 px; thumb 12 px; label column 72 px 9 px secondary mono; value 10 px cyan, min width 32; rows 10 px apart |
| Street-level "since" slider | native range with the green accent colour |
| Imagery timeline slider | full-width native range in a three-column grid (step back, track, step forward) with 8 px mono end-point dates under the track and a readout line |
| Hover | thumb glow grows to 12 px |
| Focus | 2 px white ring at 4 px offset (rail sliders); 1 px accent ring at 2 px offset (quantitative sliders) |
| Disabled | 42% opacity, not-allowed |
| Visible fill | none; the track has no coloured portion, so position is read from the thumb and the number (a weakness, see Kawn) |
| Step | 0.01 when max <= 1, else 0.1 or 25 for density (stops at sparse / balanced / dense) |
| Motion | none beyond the glow |

---

## 3. Switches

The product has no switch component. Every on/off is one of: a toggle row button (1.2), a status chip (1.7), a pill labelled OFF / ON on the Street Level header (is the layer switch: pill 999 radius, 9.6 px mono, 2 x 7 padding, 1 px border; on = green text and green 55% border; busy = cyan; warning = amber "KEY REQUIRED" in shot 32), a native checkbox in dialogs (13 px, accent #36dcff) or a two-state text button (CCTV OFF/ON, AUTO HOP OFF/ON, COVERAGE three-state, CYCLE OFF/ON). A one-line "pressed" meaning is carried by colour plus the word; there is no sliding knob.

---

## 4. Selects and inputs

| Component | Look | States |
|---|---|---|
| Rail select (HUD layout) | height 24, 6 radius, white 6% fill, white 14% border, 10 px mono, 0.6 tracking, native dropdown arrow in white; popup list uses the opaque menu colour #12121c with primary text; disabled options 30% | focus: white inset ring |
| Panel select (scene, camera) | 12 px sans text, 6 x 8 padding, 7 radius, white 6% fill, white 14% border; long names ellipsise (shot 33 shows "Nepa" clipped) | disabled 45% |
| Text input (draw label) | height 24, 6 radius, same fill and border, 10 px mono | focus: accent border; placeholder dim italic |
| Search input (location) | no box of its own: transparent field inside the dock tray, 11 px (tray) or 13 px sans, caret accent, placeholder dim italic; width animates 0 to 180 px (or fills the 132 to 190 px slot in the dock) in 300 ms | focus: native white outline ring (shot 24 shows a bright rectangular ring around the field while typing); searching: 50% opacity |
| Number field (calibration) | 58 px, 9 px mono, accent 10% fill, accent 50% border, spinner hidden | |
| Key field (settings dialog) | full width, 10.5 px mono, dark fill rgba(4,14,20,0.72), 1 px rgba(100,200,220,0.22) border, 5 radius | focus: border accent 65%; placeholder 42% |
| Native file picker | for imports; no custom look | |

---

## 5. Panel header and collapsible

| Attribute | Value |
|---|---|
| Container | glass panel: 16 radius, 1 px hairline, padding 14 top and 12 bottom with 16 sides (Layers) or 14 all sides (Scenes, CCTV, Street, Weather); soft shadow (0 8 32 black 50%) with 1 px top inner highlight (white 6%); a blurred radial glow behind (inset minus 18 to 20 px, opacity 25%) in accent, or green for Street Level, or light blue (rgba(98,181,255,0.2)) for Context |
| Header row | flex row: title, flexible hairline divider, optional status/count, collapse square; 12 px bottom margin when open, 2 px when collapsed |
| Title | 9 px mono caps, 3 px tracking, dim (8 px and 1.8 tracking when collapsed); on right rail 8 px, 1.4 tracking |
| Divider | 1 px, gradient from hairline to transparent; when the panel is open on the rails the gradient becomes accent: from 28% accent through 18% at 58% to clear, with a 7 px glow |
| Collapse square | see 1.4; glyph "-" when open and "+" when closed on the left; triangle pointing right when open and left when closed on the right |
| Collapsed look | launcher plate 176 wide, about 42 high, content reduced to title and square |
| Collapsed body | removed, not hidden-by-height |
| Click on collapsed panel | expands |
| Open transition | opacity, visibility and a 8 px rise over 300 ms for the first reveal of the Layers panel; later expands are instant |
| Sticky | header stays fixed while the body scrolls (z 2) |
| Floating variant | header becomes a drag handle (grab cursor); 14 px diagonal grip hatch at the bottom-right corner (accent lines at 60%, 55% opacity, 100% on hover); 7 px invisible resize edges on all sides |
| Cyber variant | red and slate frames with chamfered corners, small red tick before the title, 200 px collapsed width (shot 19, 33) |
| Mini status | when a dock tray is collapsed, a tiny two-line status (8 px label dim, 9 px value accent with glow) replaces the body |

---

## 6. Layer row

| Attribute | Value |
|---|---|
| Container | padding 8 top, 10 sides, 6 bottom; 8 radius; hover fill white 3%; 2 px gap between rows; voice-focus highlight rgba(92,255,198,0.10) with 1 px inset mint ring and 18 px mint glow |
| Top line | left: 16 px icon + 13 px name (Inter 500, primary, 0.3 tracking, wraps to two lines when the chip is wide); right: count (11 px mono, secondary, min width 30, right aligned) + state chip |
| Meta line | 9 px mono, 0.5 tracking, dim, indented 26 px to align under the name; patterns "source - age" ("CelesTrak - never", "TeleGeography - 26s ago"), or a status text; multi-line when long (shots 10, 22: up to 3 lines) |
| Sub-controls | appear only when on: flex-wrap row indented 26, 4 x 8 gaps; chips, legends, a 9 px info line, or an ordered step list (max height 150, scroll); step item = number then text, 8 px mono, active step accent, current step 2 px accent left border |
| Group heading | 10 px bold caps (600), 1.4 tracking, secondary colour, margin 8 x 10 x 2, 1 px white 10% rule above (not on the first group) |
| List | column, 2 px gap, scrolls with thin cyan scrollbar (6 px wide; track white 4%; thumb gradient accent 65% to 20%) |
| Row pitch | 50 px (name on one line plus meta), 62 px (name wraps), more with sub-controls |
| States | off, enabling (grey chip), on (cyan), loading (cyan LOADING), degraded/stale/partial/fallback (amber/orange), unavailable (red, row dims slightly, shot 06 shows its row on a lighter plate on hover), disabled/busy (chip ignores clicks) |
| Examples in shots | Live Flights with "FALLBACK" amber chip and count 247 (shot 22); Military Flights ON 184; Cameras 3.9K LOADING (shot 06); Mapped ALPR Cameras ON with "SHOW NEAREST" chip and an info swatch "Camera badges 6" (shot 10) |
| Motion | hover fill 150 ms; chip colour 150 ms |

---

## 7. Info cards

All world cards are drawn on one canvas above the globe, not as page elements. They share one family.

### 7.1 Shared card style

| Attribute | Value |
|---|---|
| Plate | rgba(4,12,16,0.82); selected rgba(5,18,24,0.94) |
| Border | 1 px rgba(190,232,242,0.18); selected 1 px #6be8ff at 72% (a 1.25 px line on tracked cards) |
| Radius | 4 |
| Title | JetBrains Mono 600, 12 px (13 px selected or tracked), colour rgba(232,240,244,0.96) |
| Detail | JetBrains Mono 500, 10.5 px (11 px tracked), colour rgba(147,161,173,0.92) |
| Label (plain tag) | 500 10 px |
| Leader | 1.35 px line rgba(147,213,228,0.58) to an anchor dot of 3.2 px radius with a 1 px dark ring |
| Placement | above the anchor by default, falls below if there is no room; viewport margin kept; text widths are cached per font |
| Fade | cards beyond the "keyhole" fade out across the fade band (default 7%) to the outside opacity (default 1%) |

### 7.2 Flight card (tracked) (shots 26, 27, 29, 30)

| Attribute | Value |
|---|---|
| Size | about 185 to 265 wide x 38 to 58 high, centred above the aircraft with a short vertical leader |
| Line 1 (title) | callsign, flight level or altitude, ground speed: "TCEDH - FL195 - 451 kts"; below 18,000 ft it prints feet: "BAW615 - 1700 ft - 81 kts"; adds "- STALE" when the data is old (shots 27, 29) |
| Line 2 (detail) | operator and aircraft type: "Gulfstream Aerospace G450"; "British Airways - Airbus A320 251NSL" |
| Line 3 (detail) | route when known: "BDS -> LHR", "LHR -> LAX" |
| Accent | cyan top edge on the plate when tracked |
| Companion marks | aircraft silhouette 40 to 90 px (cyan solid when tracked, white when idle) in a four-corner reticle bracket about 44 px with 1 px cyan stroke; a cyan track line (about 2 px) from the aircraft along the recent path |
| Untracked tags | small flat tag: 2 px accent bar at the left, callsign (9 px) and a smaller "FL019" in dim (shots 23, 26) |
| Nearby-contact plates | very small (about 95 x 28) plates "SHT6Y - 8.9 km / BRG 065 - CRS 000" with a 12 px blue arrow glyph that points along the bearing (shot 26, 29) |
| States | idle tag, selected card, tracked card (larger fonts), stale (suffix), acquiring (strip shows "ACQUIRING") |
| Motion | arrow rotates with bearing; card tracks position every frame |

### 7.3 Site, vessel, satellite, quake, cable and other labels

| Type | Look in the shots |
|---|---|
| Data-centre or installation card (shots 06, 24) | two-line dark plate with a 2 px cyan left bar: bold name (12 px) and operator beneath (10 px grey); a cyan 6 px dot with a short cyan stem below the card marks the site |
| Cluster badge (shot 10) | small dark rounded plate "58 MUSIC", "184 MUSIC" (count + category in 9 px caps) with a round marker |
| Plain label (radio station, camera tag) | one-line flat plate; camera tags in shot 06 are tiny cyan caps with a left bar, "CAM-" prefix |
| Quake | small plate "M3.1" with a coloured left bar (red to yellow by magnitude) |
| Satellite | orange-tinted small tags in the CRT look: name text with a tiny box marker (shot 36) |
| Vessel | like flight but with ship glyph (not in any shot; AIS key missing) |
| Draw label | small dark chip (10 px mono) with an anchor dot (shot 31) |

### 7.4 Rail cards (weather, recent imagery, context cohorts)

| Attribute | Value |
|---|---|
| Frame | 6 radius, transparent border; adjacent cards separated by a 1 px rgba(98,181,255,0.28) line; hover/focus border #79c2ff; open card border accent 55% and fill rgba(42,113,171,0.09) |
| Heading | icon + 13 px sans title (secondary; open card title #e8feff), badge on the right in 9 px mono dim |
| Lines | 9 px mono, 0.5 tracking, dim, pre-line wrap |
| Colour ramp | 4 px bar with a 2 px radius |
| Scale text | 9 px tabular numerals, right aligned |
| Chips/actions | flex-wrap row, 6 px gap |
| Daily imagery card | 94 x 112, five-row grid (thumbnail 42, date 13, sensor 11, cloud 11, badge 18), 1 px white 10% border, horizontal strip 120 tall with proximity scroll-snap |
| Roster item (space missions) | 42 min height, grid: 10 px marker, text column, 12 px chevron; 4 radius, 1 px rgba(98,181,255,0.13) border; hover/focus: border cyan 58%, fill cyan 9%, shifts 2 px right, four 9 px corner tick marks appear inside |
| Mission buttons | 9 x 12 padding, 5 radius; focus button cyan, replay button orange (#ffc078 text, 62% orange border), hover brightens the border and adds a 14 px glow; transport trio of 34 high buttons with 15 px glyphs |
| Nav buttons | 94 min width, 7 x 10 padding, 11 px mono, 0.12 em tracking, cyan |

---

## 8. Tooltips and toasts

| Component | Look |
|---|---|
| Tooltips | native browser tooltips from the title attribute; the product ships no custom tooltip surface. Dimmed controls explain why in this text (missing key, unavailable source) |
| Toast | see file 03 region 27: 10 x 24 padding, 8 radius, glass fill, 1 px accent border, 11 px mono caps, 2 px tracking, accent text, 20 px cyan glow at 15%; enters with a 20 px rise and fade over 0.3 s; stays 2 s; single instance, new text replaces old |
| Status strip pill | padding 6 x 10, 7 radius, fill rgba(3,10,18,0.78), border rgba(0,212,255,0.24), blur 14 px; label rgba(184,244,255,0.92), detail bold #00e1ff with 1.6 tracking; states: retry (amber), cancelled (amber), error (salmon #ff9b8e), acquiring (cyan with outer glow) |
| Loading chip animation | strip appears after 160 ms; chips fade and rise 150 to 300 ms |

---

## 9. Dialogs and lightboxes

| Dialog | Size | Look | Behaviour |
|---|---|---|---|
| First-run card | 544 wide (34 rem), 1.15 rem padding, 10 radius | diagonal near-black teal gradient from rgba(10,27,37,0.97) to rgba(7,13,22,0.95), 1 px rgba(54,220,255,0.42), shadows 22 px 72 px black + 40 px cyan haze, blur 28 px; a 2 px cyan scan line across the top edge with a 16 px glow | scales from 0.985 and rises 12 px as it appears; tiles slide right 2 px on hover |
| First-run header | 9 px mono caps, kicker in cyan; title 20 to 28 px (responsive 1.25 rem to 1.75 rem), weight 500, slight negative tracking; description 12.5 px, 66% white, 1.5 line height | | |
| Mission tile | grid (icon 27 px, text, arrow 20 px), min height 57, 0.6 x 0.7 rem padding, 6.7 radius, fill rgba(29,69,81,0.18), 1 px rgba(100,200,220,0.18); title 11.5 px bold mono caps, subtitle 10.5 px dim | hover/focus: fill rgba(25,151,178,0.16), border rgba(54,220,255,0.62), 2 px shift; busy: 48% opacity; explore tile is plainer (white 2.5% fill) |
| Footer | checkbox (13 px, accent #36dcff) with 10 px label "do not show again", "ESC to dismiss" hint 7.7 px; note line 10 px mono | |
| Settings dialog | 576 wide, same card, rows with LED (8 px, green with glow when set), tier dot, "exposed" tag (amber outline), "get key" link cyan, unlock text 10.5 px, key input; set rows tinted green rgba(30,81,57,0.12) with green border; close icon button top right | Save = primary solid button |
| Credits lightbox | 470 wide, max height 70 dvh, list 12 px with 1.35 line height, scrollable; full screen under 575 px | title and close stay fixed; z 200 |
| Expanded street viewer | see file 03 6.7 | Esc shrinks |
| Import preview | listed in behaviour notes; not in any shot | |
| Scrim | native backdrop rgba(0,8,14,0.65) for the sharing dialog; first-run and settings have no scrim (they float over the live globe) | |
| Chip close | 16 px close icon in a 0.15 rem padded square; hover border accent 50% | |

---

## 10. Dock tray (Location / Visual presets)

| Attribute | Value |
|---|---|
| Wing collapsed | 167 x 68 zone inside the dock, 8 x 14 padding; label row: 16 px mask icon in 30% white + 9 px mono caps label 2 px tracking in 72% white; hairline underline below the label (gradient from 30% white to clear); two mini-status lines (9 px city in #bdefff, 8 px landmark in secondary, max 190, ellipsised) |
| Wing hover / tray open | icon and label turn accent with a soft 5.6 px glow on the icon (shots 10, 11, 12 show LOCATION in cyan with a cyan pin) |
| Tray | popover 544 wide, padding 1.55 rem top (room for the pin) and 0.6 rem others, 6.7 radius, rgba(8,20,30,0.94), 1 px rgba(123,189,211,0.32), shadows (14 px 38 px black 58%, 24 px cyan haze), a rotated 9.6 px square notch at the bottom pointing at the wing |
| Pin button | 26 px circle, rgba(8,24,34,0.96) fill, 14 px glyph at 58% opacity; hover or pressed: full opacity, rotated minus 12 degrees, scale 1.08 |
| Tray open motion | 180 ms fade and scale from 0.96 to 1 with origin at the notch; 140 ms hover delay; 420 ms close delay |
| Location contents | search toggle (30 square, 6 radius) + field, a hairline, city pills (7 x 10 padding, 6 radius, 9 px mono caps), landmark row (max height 0 to 50, 250 ms) with pills (name 10 px sans + key chip 8 px mono) |
| City pill | white 4% fill, white 8% border, 20 radius in default (6 in dock), hover lifts 1 px, active: accent border/text/fill and 12 px glow |
| Look tile (presets) | button column: 16 px icon (18.4 px in tray), 8 px mono caps name, key letter 7 px at the top-right revealed on hover/active; min 58 wide and 59 high in tray; 7 radius; fill white 3%, hover white 6% and 1 px lift; active: accent border, accent 15% fill, 20 px glow and inner 1 px accent ring, icon glows |
| Collapsed dock state | wings show label + mini status; tray hidden |
| Disabled/unavailable | map-source chips dim with reason |

---

## 11. Search field and chips (Location)

| Attribute | Value |
|---|---|
| Layout in tray | one row: magnifier circle button (30 to 32 px) with the field to its right in a 132 to 190 px slot ending in a hairline; then city chips in a horizontally scrolling row (no scrollbar, shots 10 to 12 show the row clipped at "LONDON") |
| Default text | "Search any location..." dim italic; suggested cities AUSTIN, SAN FRANCISCO, NEW YORK, TOKYO, LONDON (then PARIS, DUBAI, WASHINGTON DC, TALLINN off-screen) |
| Active search | field text 11 px sans primary; ring white 2 px (shot 24 "Heathrow Airport"); chips stay; field dims to 50% while a request is running |
| Result | camera flies; mini-status updates; toast on failure ("Location not found", "Search failed") |
| Chip look | see city pill above |
| No suggestions dropdown | there is no autocomplete list; Enter submits |

---

## 12. Timeline and scrubber

| Attribute | Value |
|---|---|
| Where | the shared "observed history" block for rain radar, satellite clouds and lightning inside the Weather card (no screenshot because those layers are off in every shot) |
| Layout | padding 5; a three-column grid: step-earlier chip, native slider, step-later chip; end-point dates under the slider at 8 px mono; controls row (Latest, Play/Pause chips, readout) with 6 px gap; readout 9 px secondary with tabular numerals |
| Behaviour | drag previews immediately, commits after 150 ms idle |
| Scenes progress | pill track 18 high, white 6% fill with 1 px white 10% border, inner fill gradient accent 20% to 75% with the percentage in 8 px text at its right end; width animates over 100 ms linear (shot 33 shows an empty bar and "Ready") |
| Imagery comparison divider | draggable vertical line with arrow-key support |
| No global time scrubber | the main screen has no timeline bar; the reference reserves it for weather and replay |

---

## 13. Voice capsule

| Attribute | Value |
|---|---|
| Capsule | 140 x 83 (8.75 rem wide, 5.35 rem tall), overlapping the dock by about 15 above and below (negative margins), 12 radius, fill rgba(8,31,42,0.94), 1 px rgba(0,212,255,0.36), shadows 12 px 30 px black plus 26 px cyan haze |
| Content | top-left kicker 7 px caps "AI AGENT" dim; tiny STD/MINI tier button (11.5 px tall, 0.18 radius, accent 34% border; pressed amber) and a cost readout 6.4 px ("~$0.00", warns amber, caps red); mic orbit circle 39 px with rgba(0,212,255,0.06) fill, 1 px accent 48% ring and 14 px glow; label "ON/OFF" 6.7 px beneath; an audio-level strip (height 36) at 54% opacity idle, 100% when listening; status word 7 px; detail line 6 px ellipsised ("VOICE STANDBY" in shot 06) |
| States | off: as above; connecting/listening/executing: visualiser at 100% and status line shown; listening: mint #5cffc6 border (45%) and status; executing: yellow #ffdc78 border (55%) and status; error: border and status #ff5c6e, capsule becomes a panel-style multi-line block, detail text #ffb4bd |
| Error tray | 19 to 24 rem wide, 13.6 px padding, 6.7 radius, fill rgba(34,10,16,0.96), 1 px red 50%, 14 px shadow; header row (10 px bold caps red) with DISMISS button (7 px caps, 4 radius); message 12.5 px sans; hint 8 px mono |
| Voice card | 352 wide, 0.62 x 0.72 rem padding, 10 radius, entry animation rises 7 px and fades; header with kicker (7 px, cyan 72%), phase word (amber while busy), dismiss square 17.6 px; YOU/AI caption rows with 11.5 px sans text; "THIS" pointer row with a bordered label; plan list with glyph states (empty box, spinner dot amber, check accent, cross red) 9 px mono; result block |
| Cost thresholds | amber at about 2 dollars, red at about 5 dollars |
| Motion | meters height and opacity 70 and 90 ms linear; card 200 ms |
| Focus | hold-key push-to-talk plus a click for open mic |

---

## 14. Scene shot list

| Attribute | Value |
|---|---|
| List | column, 6 px gap, 96 to 160 px high, scrolls (6 px thumb white 16%) |
| Row | 9 radius, 1 px white 12% border, white 3% fill, 7 x 8 padding; title 10 px mono 1 px tracking in primary; small LOAD and DEL buttons right; meta line 8 px dim "NORMAL - OFF - 4.0s + 0.9s" (look, detection, duration + hold) |
| Active row | accent border, accent 8% fill (shot 33) |
| Rename | double-click turns the title into an input with accent border and panel-dark fill |
| Empty | dashed 1 px white 14% box, 8 radius, centred 9 px text |
| Danger | DEL hover red |
| Playing | all buttons but STOP at 45%; caption pill at screen top-centre: black 45% fill, 1 px cyan hairline, 11 px mono, 2 px tracking, #bff4ff |
| Imported/shared | preview dialog before commit |

---

## 15. Other parts

| Part | Spec |
|---|---|
| Scrollbars | rails: 6 px wide, track white 4% with full rounding, thumb accent gradient 65% to 20% (Layers) or 8 px with a 1 px accent border (CCTV); thin cyan bar on Display; dock and tray rows: hidden; Context panel in shots shows a plain light thumb |
| CCTV well | 16:9, 10 radius, dark teal gradient with faint cyan radial at the top-left, min height 150 (120 under 720 px); image fades in 180 ms |
| Source badge | 999 pill, 8 px caps |
| Compass | see 1.1 |
| Safe frame | white 28% 1 px with black 35% outer hairline and thirds at 20% white |
| Scope ring and vignette | circular keyhole mask with soft 11% feather; faint ring line white-blue |
| Celestial markers | 20 px glyphs: sun warm gold with 7 and 18 px glow, moon cyan with 7 and 17 px glow |
| Orbit badge | cyan 10 px caps with glow |
| Credits | 11 px, 70% white links, underlined |
| Frame rate readout | cyan 11 px tabular numerals under the brand |
| Selection corner brackets | cyan, 4 corners, 1 px, about 44 px square (aircraft) and 20 to 36 px (generic) |

---

## 16. State matrix (summary)

| Component | Default | Hover | Active/on | Focus | Disabled | Loading | Error |
|---|---|---|---|---|---|---|---|
| Round icon button | glass circle | scale 1.1, border 15% | accent ring and glow | white 2 px ring | 55% | none | none |
| Toggle row | hairline box | fill 6% | accent (green for detect) | white inset ring | 45% | none | none |
| Layer state chip | OFF grey | border 15% | cyan ON | white inset ring | busy ignores clicks | LOADING cyan | UNAVAILABLE red |
| Chip | grey | border 15% | accent | white inset ring | 70% | grey-white border | red |
| Segmented | dim | fill 8% | solid cyan | inset ring + dark ring | 45% | none | none |
| Slider | thumb with glow | glow 12 px | n/a | ring | 42% | none | none |
| Select | dark | none | n/a | inset ring | 45% | none | red border (aria-invalid) |
| Panel | glass | none | glow | per-control | none | spinner inside | red message strip |
| Layer row | transparent | fill 3% | n/a | n/a | n/a | meta says loading | red chip + meta |
| Info card | plate | n/a | cyan edge (selected) | n/a | n/a | "ACQUIRING" strip | "STALE" suffix |
| Toast | hidden | n/a | visible 2 s | n/a | n/a | n/a | same style |
| Dock tray | closed | opens after 140 ms | pinned | ring holds it open | n/a | n/a | n/a |
| Voice capsule | off | border brightens | listening mint / executing yellow | ring | n/a | connecting | red capsule and tray |
| First-run tile | slate | cyan border, 2 px shift | n/a | cyan ring | busy 48% | status line cyan | note stays with retry text |

---

## 17. What makes it feel the way it does

- **Almost nothing has a fill colour.** Controls are translucent white at 3 to 6% with a hairline, so at rest the interface is a set of faint outlines; accent colour only appears when something is on, which makes "what is active" instantly readable at a glance across the screen.
- **State is colour plus a word.** Chips always print their word (ON, FALLBACK, UNAVAILABLE), which helps accessibility and makes the interface honest about data quality.
- **Glow is the only elevation.** There are no drop-shadow cards in the rails; surfaces differ by glow strength.
- **Tight radii on small things (3 to 4) and big radii on containers (16)** gives a technical, instrument-like contrast.
- **The in-world card family (4 radius, dark plate, hairline, mono text) is deliberately different from the glass rails (16 radius, blur)** so the map feels like a data layer and the rails feel like tools.

---

## 18. Kawn improvements

- **Touch and size.** Raise all hit areas to 44 px (the reference uses 20 to 38 px, chips as small as 17 px high, sliders with 10 px thumbs); 12 px minimum text; chips 32 px tall on touch.
- **Visible slider fill and value** with a proper large thumb (20 px on touch) and direct number entry.
- **Real switch component** (track plus knob) for layer rows, with text "On/Off" as well, instead of a tiny word button.
- **Arabic and right-to-left.** Mirror chips, segmented controls, sliders (fill from the right), chevrons and notch positions; avoid all-caps tracked labels; use an Arabic-capable family for names, mono only for figures and IDs; support Eastern Arabic numerals; leader lines and card placement must handle right-to-left text.
- **Light and high-contrast themes** for every component above (surface white 78%, hairline black 8%, accent teal darkened to meet AA on light); status colours must keep 4.5:1 text contrast: the reference's amber and red tints on 10% fills are borderline.
- **Calmer tone.** Rename classification-style strings and ISR terms; use plain status words ("Live", "Delayed", "Using backup source", "Key needed") with an icon in addition to colour; replace the aircraft-targeting reticle with a soft circular halo.
- **Cards.** Add a proper tracked-object panel (photo, name, facts, actions) in the context rail, keep the small in-map plate for labels, and let the card be keyboard focusable and readable by assistive tech (the reference needs a parallel hidden list).
- **Tooltips.** Provide a real tooltip component (keyboard focus triggers it, includes shortcut hint and reason when disabled) instead of native title text.
- **Autocomplete.** Add a suggestion list to the search field with bilingual results; keep recent places chips.
- **One scrollbar and one toast system.** Unify; allow a persistent error banner with an action ("Retry", "Use another source").
- **Motion.** Keep 150/300 ms and respect reduced motion for the pulse, shimmer and tray scale; add a skeleton instead of a shimmer-plus-text for loading wells.
- **Timeline.** Provide a single global time control for live versus replay on the main screen rather than only inside weather.

---

## UNVERIFIED

- No shot shows: the settings dialog, first-run card, voice card, error tray, the CCTV with an active frame, the timeline, import preview, expanded street viewer, cluster of vessel or satellite cards in the default look, tooltips (none are custom). Their numbers come from the stylesheets only.
- Exact pixel size of in-world card padding (inferred from shots; the canvas painter's padding was not read).
- Icon identity for the layer rows is read from small images; some may be emoji-like system glyphs that vary by operating system.
- Whether the Display panel's light scrollbar thumb in the Context panel is a deliberate choice (looks like a missing style).
- Colours of in-world markers per layer (cyan for sites, grey/green for radio, red-yellow for quakes) are read visually; exact hex values are not confirmed.
- Hover behaviour of canvas cards (not testable in static shots).

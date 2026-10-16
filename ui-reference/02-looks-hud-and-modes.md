# 02 - Looks, HUD Layouts and Display Modes

**Purpose.** Describe what the seven whole-screen looks, the four HUD layouts, the circular scope, the detection overlay, the cockpit view, recording / safe-frame mode and Clean UI look like on screen, with positions and values, so a designer can recreate equivalents. Main evidence: shots 01-05, 07-09, 13-21, 22-29 (1920 x 1080; shots 17-29 are at a slightly smaller UI scale). Parameter ranges and behaviour rules come from the earlier behaviour docs; this file is about the picture. Shot 18 (Minimal layout) is not in the evidence folder.

---

## 1. The seven looks

All looks filter only the 3D scene (imagery, tiles, map glyphs, 3D aircraft). Panels, dock, HUD text, scope darkening and detection overlay are drawn on top and are never filtered. Looks crossfade over 500 ms. Selecting a look happens from the Visual Presets tray: a floating panel above the dock with a row of seven tiles and, beneath a hairline, a "MAP SOURCE" row of five chips (shot 02).

### 1.1 Preset tray anatomy (shot 02)
| Part | Look |
|---|---|
| Tray panel | x about 688-1230, y about 838-975, glass fill, 12 px radius, sits 20 px above the dock; a small round bell button hangs off its top-right corner (x about 1229, y 837) |
| Tiles | seven tiles in one row, each about 70 x 58 px, 8 px gap; icon on top (20 px), label under it in tiny caps; idle tiles near-transparent with a faint border |
| Active tile | cyan 1 px border, cyan 15% fill, outer cyan glow, icon and label brighten (shot 02 Normal, shot 03 NVG) |
| Order | Normal, CRT, NVG, FLIR, Anime, Noir, Snow |
| Map source row | label "MAP SOURCE" at left, small cyan word "SAT" at right; five chips: Google 3D, Bing Aerial, Bing Labels, Esri Satellite, OSM; the chosen chip has a cyan outline and 15% fill (Esri Satellite in shots 02-05); unavailable chips are dim |
| Dock link | the VISUAL PRESETS half of the dock brightens to cyan with a picture icon while the tray is open |

### 1.2 Look-by-look picture
| Look | Colour | Texture | Edge treatment | Animation | Best seen |
|---|---|---|---|---|---|
| **Normal** | untouched true-colour imagery; green parks, tan roofs, brown-green river | none | none | none | shots 01, 07, 08, 22 |
| **CRT** | image pushed to dark olive-amber; greens turn khaki, whites turn warm yellow | strong horizontal scanlines every few pixels (about 4 px period), faint RGB stripe mask, coarse pixel grain | darker corners; barrel bulge available (set to 0 in the tactical preset) so edges stay straight in shot 05 | rolling bright band, tiny horizontal jitter, flicker | shot 05 |
| **NVG** | monochrome phosphor green; bright sources bloom into soft halos | fine scintillation noise, honeycomb texture, subtle scanlines | circular tube vignette: black outside a green disc roughly 800 px across, brightest centre, steep falloff; the HUD and panels still sit on the black corners | scintillation sparkle | shot 03 |
| **FLIR** | greyscale "white-hot": water black, roofs and roads bright, vegetation mid-grey (an Ironbow colour ramp black to purple to red to yellow to white is available with the Ironbow slider) | soft diffraction blur, coherent drifting noise, contour banding | circular lens vignette like NVG but softer; black outside | slow noise drift | shot 04 |
| **Anime** | saturated, luminance banded into flat cel steps: teal sea, orange sand, deep green; slight warm shift | none; posterised flat areas | dark outlines traced along coastlines and roads (default thickness 0.5) | none | shot 13 |
| **Noir** | black and white, high contrast with a 15% sepia warmth; sea reads pure black, land bright white | film grain 0.5 | darkened vignette 0.5 | none | shot 14 |
| **Snow** | cool blue-grey wash, partial desaturation, white frost on bright areas, faint fog at the bottom | five layers of falling white snowflakes (near ones larger and faster, swaying with wind) | none | continuous | shots 15, 16 |

### 1.3 What stays constant across looks
- Panels, dock, title, chips, credits keep their cyan/blue look (shots 03, 04, 05, 13, 14, 15).
- Detection plates and brackets recolour for CRT (amber), NVG (green; military shifts red-orange), FLIR (warm white) and use the default cyan palette for Normal, Anime, Noir, Snow (compare the label strips in shots 03, 04, 05, 13).
- HUD text recolours per look (file 01, section 1.7): amber in CRT (shot 05), green in NVG (shot 03), white in FLIR (shot 04), cyan elsewhere.
- Selecting NVG, FLIR or CRT also forces the HUD on in Tactical layout, switches bloom off, sets sharpen to 49%, and sets detection to Dense 75% unless the user already adjusted it.

### 1.4 Parameter strip for the current look (Display panel)
When a look is active, a framed "PARAMETERS" block with a small collapse button appears in the right-hand Display panel (shots 03, 04, 05, 13, 14, 15). Each parameter is a mono caption left, a thin track with a cyan dot thumb in the middle, and a cyan numeric value right (11 px). Rows per look: NVG: Gain 0.18, Bloom 0.22, Scanlines 0.96, Pixelation 1.0. FLIR: Sensitivity 0.85, Bloom 0.20, WHOT/BHOT 0.33, Pixelation 1.0, Ironbow 0.00. CRT: Pixelation 1.0, Distortion 0.00, Instability 0.42. Anime: Saturation 1.0, Edge Thickness 0.50. Noir: Contrast 1.2, Grain 0.50, Vignette 0.50. Snow: Density 0.60, Wind 0.50. Normal has none.

### 1.5 The Display panel as a column of toggles (shots 03, 09, 22)
A right-hand glass panel, about 272 px wide (shot position x 1597-1867), header "DISPLAY" with a collapse arrow, then full-width tall outlined buttons stacked with 8 px gaps. Each button shows a small icon and a tracked caps label at about 9 px. States: off (dim, thin border), on (cyan border, 15% fill). The stack, top to bottom: HUD (with a Layout dropdown below it when on), DENSE (the detection button, green-outlined while on, with sliders Density, Allocation as a two-way ELASTIC / WEIGHTED switch, Fade, Outside beneath), PARAMETERS block (per look), 3D (with a two-way PROXIMITY / ALL switch), SCOPE (with Feather slider), DRAW, CELESTIAL, CLEAN UI, BLOOM (sparkle icon, slider when on), SHARPEN (magnifier icon, slider at 49% when on). The panel scrolls inside its own height; a thin cyan-gradient scrollbar sits on the right edge.

---

## 2. Scope (circular viewport) and vignette

| Aspect | Look |
|---|---|
| Shape | a perfectly centred circle, radius about 1.05 x half the viewport height (about 567 px on a 1080 px-high frame, so the circle touches the top and bottom and shows black wedges left and right on a wide window) |
| Edge | feather percentage of radius: 0% = hard cut (shot 07, crisp circular crop about x 393-1527); 11% default = soft falloff (shots 01, 13); 100% = a long smooth fade where the picture dims from the centre outward and the corners go solid black |
| Corners | near-black page colour; stars stay faintly visible at full-globe altitude (shot 09) |
| With panels | panels sit on the dark side; the left rail and right panels remain in full colour over it |
| With NVG / FLIR | a second, built-in circular vignette lives inside the scope circle (shots 03, 04), so the picture looks like a smaller disc than shot 01 |
| With Snow | the scope still darkens the sides (shot 15) |
| Off | removes the darkening: full rectangular imagery (shot 21 globe and shot 22 city fill the frame, only a soft side vignette remains from other layers) |
| With celestial ring | a thin ring (0.8 px, pale cyan at 9.5%) hugging the circle 11 px inside it, with a gold sun glyph and a cyan moon glyph 36 px inside the ring; visible only in Normal with the whole Earth disc inside the circle (shot 09 shows a faint ring and moon at bottom-right, x about 1330, y 905) |

---

## 3. HUD layouts

All layouts share one design: all text uppercase mono, HUD colour (file 01 section 1.7), 4 px text glow, no pointer input, 0.4 s fade in and out, tiny edge texts, and a layer order above the scope but below detection and panels. Positions below are image pixels at 1920 x 1080 unless stated, with the stylesheet values in parentheses.

### 3.1 Summary table
| Layout | Corner blocks | Bars | Edge texts | Other |
|---|---|---|---|---|
| Tactical | four, with big 22 px bracket glyphs, no box | none | both | default layout, shots 03-09, 13-16, 22-24 |
| Operator | four, framed rounded boxes (8 px radius, dark 36% fill, white 16% border), brackets hidden | top banner + bottom bar | both | shots 17, 20, 21, 25-29 |
| Minimal | top-left block (mode + summary) and bottom-left coordinate block only | bottom compact bar | none | no classification, no ids; no screenshot supplied |
| Cyber | four chamfered red-framed cards, left-facing status card is smaller | optional bottom bar (only in a mid-size window band) | both, but shifted | sonar disc behind, whole shell reskinned red/slate, shot 19 |

### 3.2 Tactical (shots 03, 05, 08, 13, 22, 23)
| Element | Position | Notes |
|---|---|---|
| Top-left block | x 59-470, y 155-263 (36 px from left edge; top at 14% of height, min 136 px) | line 1 classification (12 px, 600, 2.5 px tracking); line 2 ids; line 3 mode name large (16 px bold); then a SUMMARY caption (9 px, 55%) and a one-line summary 11 px, up to 420 px wide, ellipsis; a 22 px "[" bracket glyph sits at its upper-left corner (x 48, y 178) |
| Top-right block | right edge about 1860, y 155-195 | "REC" + dot (red, blinking) + UTC date-time with Z suffix, then ORB / PASS line; "]" bracket glyph at x 1870, y 178; the REC dot in shot 03 (NVG) and shot 04 shows blink states: visible in shots 04, 08, 13, 14, hidden in 03, 05 |
| Bottom-left block | x 59-335, y 912-945 (bottom edge sits above the dock at about 2% of height plus 6.5 rem) | MGRS line then DMS latitude/longitude line, with "[" bracket at its lower-left (x 48, y 920); the credit line sits right underneath it at y 964 |
| Bottom-right block | right edge 1860, y 885-945 | GSD and NIIRS line, ALT and SUN line, AIS line (dimmer, a dash by default); "]" bracket at the right end of the first line (x 1870, y 893) |
| Left vertical text | x about 35, centred around y 340-540, rotated -90 degrees | "COLL: <time>" then "ONA: <angle>" 10 px, 50% |
| Right vertical text | x about 1885, y about 320-540, rotated +90 degrees | "BAND: PAN  BITS: 11  LVL: 1A" |
| Layer rail interplay | left rail drops to y 285-890 so it clears the top-left block (compare shot 01 where HUD is off and the rail spans y 103-943) | right panels also drop to start at y 285 in shot 03 (Display) |

### 3.3 Operator (shots 17, 20, 21, 25-29)
| Element | Position | Notes |
|---|---|---|
| Top banner | centred, x 636-1284 (about 52% of width, max 720), y 109-131 (top at 10% of height, min 90) | one bar, 10 px text, gradient from black 56% to dark teal 50% with cyan glow; left segment classification (dim), centre mission id (brighter), right "PAGE 1/1" (dim); it sits under the title cluster and overlaps loading chips visually in shot 25 |
| Top-left box | x 32-430, y 151-267 | same text as Tactical in a rounded box: classification, ids, mode, SUMMARY and one summary line; no bracket glyph |
| Top-right box | x 1638-1888, y 151-205 | REC line and ORB / PASS line only; REC dot is red in shots 26, 27, 28, 29 |
| Bottom-left box | x 32-298, y 911-964 | MGRS and DMS lines |
| Bottom-right box | x 1648-1888, y 887-964 | GSD / NIIRS, ALT / SUN, AIS |
| Bottom bar | centred, x 655-1265, y 895-916 (about 48% of width, max 680) | one line "MGRS ... LAT: ... LON: ..." centred, 10 px, thin white-20% border, dark gradient (shot 20) |
| Left edge text | x about 30, y 355-540 | same as Tactical |
| Right edge text | x about 1888 | same |
In Operator the right panels and the rails keep their positions from the no-HUD layout (rail top at 283 vs 103), so the Display panel is shorter (shots 17, 25). Operator reads as a "framed instrument" where Tactical reads as "floating text".

### 3.4 Minimal (stylesheet only)
| Element | Position |
|---|---|
| Top-left block | moved up to about 88 px from the top; mode name and summary only; max width 280 px |
| Bottom-left block | 12% of viewport height above the bottom; coordinate lines |
| Bottom bar | centred, 12% of height plus 4 rem above the bottom, compact MGRS/LAT/LON line |
| Hidden | classification, system ids, top-right and bottom-right blocks, both edge texts, brackets |
Used by recording mode by default (see section 7).

### 3.5 Cyber (shot 19)
| Element | Position | Look |
|---|---|---|
| Status card (top-left) | x 25-238, y 112-186 (fixed 238 px wide, about 10% from the top, 28 px from the left) | chamfered frame with a red corner tick; "NORMAL" large in white with a red "// CYBER" tag; SUMMARY caption and one summary line truncated with fade |
| REC card (top-right) | x 1622-1886, y 152-202 | chamfered, white text, red corner tick; "REC date-time" and "ORB / PASS" |
| Coordinate card (bottom-left) | x 33-316, y 896-946 | chamfered, "MGRS" and DMS lines in white-red |
| Telemetry card (bottom-right) | x 1638-1893, y 980-1053 (pinned in the corner at 28 px) | GSD, ALT, AIS |
| Left rail | x 60-302, y 220-818, 300 px column, red hairline on its left | section titles in red with a leading red tick ("| DATA LAYERS"), 8 px; ON toggles red outline; then a "SCENES" bar with a "+" below it |
| Right rails | four stacked chamfered header bars x 1685-1863, y 220-415: DISPLAY, CCTV, STREET LEVEL, CONTEXT, each with a left-pointing collapse triangle; only one may be open (accordion) | collapsed by default |
| Edge text | left vertical text moved up into the lane above the rail (x 12, y 10-200) | |
| Sonar disc | up to 80% of width or 104% of height, centred, behind everything | very faint (about 57% intensity): eight concentric hairline rings (pale grey 17%), a red-to-teal sweep wedge about 24 degrees wide, rotating once per 5.2 s; barely visible in the still (shot 19) |
| Dock | same three-part dock, but frame gets a red chamfered edge and small corner ticks (x 755-1165, y 995-1055) | |
| Logo | wordmark turns white, symbol greyscale with red | |
| Look side-effect | entering Cyber also switches to FLIR with Ironbow 0.42 | the screenshot (shot 19) shows Normal imagery because it was taken before / without that switch, or the swap was undone; treat as UNVERIFIED |

### 3.6 HUD in extreme zoom (shot 20, ALT 970 m)
Same Operator layout; telemetry values change (GSD 0.38 m, NIIRS 6.3, ALT 970 m); the summary line reads "street" band wording. Nothing resizes with altitude.

---

## 4. Detection overlay

### 4.1 Elements
| Element | Look |
|---|---|
| Corner-bracket reticle | four L-shaped 1 px corners in the tier colour, each segment about 55% of the half-size (minimum 4 px); for aircraft, grows as the camera nears (about 3x at 1 km, 0.5x at 8,000 km) so a plane at street scale is surrounded by brackets about 50-60 px wide (shots 23, 26, 29); in a dense field they cluster into the dotted bracket swarms seen over London (shots 03, 05, 16, 09) |
| Ambient label card | one-line plate: bold identifier (callsign, camera code, station) then a dimmer micro-field (flight level in 80% size, e.g. "ABY369 FL081"); 10 px mono, a 2-3 px accent bar on the left edge in the tier colour, dark plate about 46-56% opaque, 3 px radius, 6 px right padding, 4 px top and bottom padding (shot 16) |
| Leader line | 1 px line from reticle corner to the plate's opposite corner, tier colour at about 60% |
| Rich world card | 82% teal-black plate (94% selected), 4 px radius, pale hairline border (cyan 72% when selected), two lines: bold name 12-13 px plus a dim 10-11 px subtitle (shot 01: "Tata Stratford / Tata Communications", "Stellanor London North / Stellanor Data Centers"); a 3.2 px dot with a 1 px dark ring anchors it, and a 1.35 px straight leader connects dot to card (vertical for above / below, horizontal for left / right) |
| Tracked card | larger plate, title 13 px semibold white, detail 11 px; example (shots 26, 27, 28): "BAW615 - 1700 ft - 81 kts" over "British Airways - Airbus A320 251NSL" over "BDS to LHR", cyan top border, pinned directly above the tracked plane (offset about 70-110 px) |
| Sparse focus ring | in Sparse mode only, a faint circle of radius half the smaller screen dimension, 35% opacity, in the glow colour |
| Scanlines | very faint 4 px-period scanlines (4-9% strength per look) |
| Blend | screen blend over the scene with a 3 px glow |

### 4.2 Colour tiers (default cool palette)
| Tier | Colour |
|---|---|
| Civil aircraft | cyan #22e0ff |
| Military | amber #ffb347 |
| Sea | teal #3fe0c8 |
| Space | violet #bda4ff |
| Vehicle | slate #8fa6b4 |
| Road: jam / slow / free / none | #e05252 / #f0b23e / #2ecc71 / #c9c9c9 |
| Transit | bus #5EF08A, tram #FFC24A, subway #FF4538, rail #D9A6FF, ferry #5FD6FF, unknown #D8DDE5 |
Per-look shifts: CRT toward amber and khaki (labels read orange-yellow on dark olive in shot 05); NVG toward green with military red-orange (shot 03 plates are bright green-on-black but camera brackets stay a bright cyan-green); FLIR toward white and warm grey (shot 04 brackets are white).

### 4.3 Density, fade, outside
- Inside the scope circle labels are fully opaque; beyond the radius they fade over the Fade band (default 7% of radius) to the Outside opacity (default 1%). In shot 09 the outer brackets over black space stay faintly visible; in shot 07 (scope feather 0) labels end exactly at the circle edge.
- Label counts at default Dense 75%: about 41 at metro scale, up to 68 at street scale (visible in shots 01, 16).
- Aircraft brackets keep a floor of 35% alpha even when Outside is 1%.
- In cockpit, bracket opacity is multiplied by 0.45.
- Cards stay out of the HUD text and out of panels; in shot 26 the callsign plates stop at the panel edge.

### 4.4 Tracked object look (shots 26-29)
Tracked aircraft: solid cyan silhouette (or 3D model once within 150 km altitude, shot 27 and 28 show the big cyan A320 model on the runway), a 3D-style cyan leader stretching away along its recent path to the right edge (thin 2 px line, glowing), a corner-bracket frame around it, a small crosshair-like ring around the camera centre ("keyhole" radius circle drawn as a 1 px pale-cyan hairline circle in shots 27, 28, 29), a small "N" compass tag with a triangle at the circle top (x 960, y 160, "HDG 000"), and contact arrows (small cyan arrowheads with two-line plates such as "EIN16C - 685 m / BRG 355 - CRS 295") at the circle edge pointing to nearby traffic. A "STRAIGHT-DOWN VIEW" pill appears bottom-centre (x 868-1050, y 958-990) during the top-down glide in shot 28: pill with 1 px cyan border, 10 px tracked caps.

---

## 5. Contacts roster and nearby arrows (shot 25)
The Context panel (bottom of the right rail, x 1577-1870, y 713-875) expands to show CONTACTS and SPACE MISSIONS buttons (cyan-outlined, 30 px tall, 8 px caps), a second row with COCKPIT (aeroplane icon, cyan), SEARCH NEARBY SITES, and a small icon button; below, a status line "TCEDH - 250 KM FLIGHT / VESSEL WINDOW" (10 px), then PREVIOUS / FOCUS / NEXT buttons at the clipped bottom. A bright white vertical scrollbar stands at its right edge (x 1857-1870), the brightest object in the UI. In the map, a cyan route line leaves the selected aircraft and a "SHT6Y - 8.9 km / BRG 065 - CRS 000" micro plate (8-9 px) marks a nearest-contact arrow (x 1190-1285, y 695-720).

---

## 6. Cockpit view (first-person)
(No static screenshot was supplied; values come from the stylesheet and earlier behaviour docs. Verify against the live app.)
| Part | Look |
|---|---|
| Entry button | pill-ish rectangle 38 px tall, 8 px radius, near-black 90% fill, 1 px #00dcff border at 50%, cyan 18 px aeroplane icon, label COCKPIT 10 px / 600 / 0.14 em, a "C" key chip; also embedded in the Context panel (shot 25, x 1593-1700, y 804-832); floats at the dock's right in other states |
| Global style | accent #00dcff (civil) or #ffb800 (military); emergency #ff4444; text #f4fbff; every text gets a 10 px accent glow plus a 1 px black drop shadow; perspective 920 px |
| Visor | the whole view lives inside a visor rectangle inset 62-90 px from the top, 18-48 px from the sides, 38-56 px from the bottom; a circular optical keyhole of radius min(40% of width, 52% of height) is the focus circle |
| Frame | a 2 px accent circle at 72% with an 18% inner tint on the keyhole; concentric thin rings, tick arcs and corner marks around a my-location reticle; an arched roll scale across the top (about 620 px wide, 96 px tall, tilted back 8 degrees) marking -30 to +30 in 10s; left and right pitch rails (-20 to +20) |
| Tapes | speed tape on the left rim and altitude tape on the right rim, curved to follow the keyhole circle, nine ticks each, fixed pointer, accent colour; speed three digits (e.g. 451), altitude five digits (e.g. 19500) |
| Readouts | large ground speed, heading (three digits and a degree sign) and altitude; digits roll up and down; compass tape of seven divisions 30 degrees apart; clock in UTC and lat/lon to 3 decimals |
| Top text | "FIRST PERSON", callsign, a meta line (fleet, feed state, "course aligned"), emergency tag when applicable |
| Contact window | a side panel up to 340 px (or 28% of width) with FLT / MIL / AIS / SITE counts, nearest object with a rotating arrow and "AHEAD" or "L / R nnn" bearing, footer "not an all-clear" |
| Signals window | up to five timestamped rows |
| Briefing and utility popovers | DISPLAY and RADIO launchers at the lower right, kept clear of REC and the briefing panel |
| Vision chips | PREV / CURRENT / NEXT: Optical (Normal), CRT, NVG, FLIR, Anime, Noir, Snow |
| Clouds | optional low-res cloud layer, circle-clipped to the keyhole (radius min(40vw, 52vh)), blur 0.7 px, saturate 0.72, contrast 1.08, 900 ms fade to 86% |
| Map HUD in cockpit | at 78% scale and 72% opacity in four peripheral corridors; hidden at 760 px and below |
| Traffic | nearby aircraft as silhouettes, far ones as ring-with-dot pips (civil cyan-white, military amber); own aircraft not drawn |
| Map panels | all collapsed on entry |
| Cyber | the shell skin reskins cockpit chrome too; clouds still show |

---

## 7. Recording mode and safe frame
| Aspect | Look |
|---|---|
| How it starts | only by scene playback or tools, not a user toggle |
| Hidden | title, style indicator, top action cluster, loading and sync chips, Display and Parameters panels, dock, layer rail, camera feed, Radio, Context, voice pill, scene panel, expanded street-level viewer, clean-exit pill |
| Kept | HUD (Minimal layout by default, or full / off / auto), detection overlay, scope, scene runtime bar, attribution line (moves to the 36 px bottom-left corner) |
| Safe frame | a centred rectangle drawn with a 1 px white border at 28%, an outer 1 px black hairline at 35% and an inner white 6% line; width min(90% of width, 90% of height x 16/9) for 16:9; for 9:16 the width is min(58% of height, 76% of width); plus rule-of-thirds guides: two vertical and two horizontal lines at 33.3% and 66.7%, white at 20%, 0.4% thick; no label text; sits above everything except panels (layer 95) |
| Post-recording | HUD layout and visibility are restored |
| Scene runtime | a compact bar bottom-centre showing the current shot (see scene shots 34-36 for the scenes panel) |

---

## 8. Clean UI
| Aspect | Look |
|---|---|
| Result | only globe imagery, scope darkening, HUD, detection overlay, annotations and attribution remain; panels, rails, dock, title, top cluster, chips fade to zero and stop receiving input |
| Exit pill | text "EXIT CLEAN VIEW", 10 px mono, 1.8 px tracking, 8 x 14 px padding, 999 px radius, fill rgba(8,10,14) at 88%, 1 px white border at 25%; centred 18 px from the top; above every other layer |
| Keys | V toggles; Escape does not exit |
| Attribution | stays visible at the 36 px bottom-left corner |
| Embed mode | the same but also hides the HUD and exit pill |
| Not saved | reload shows chrome again |
In Kawn terms: this is the "presentation" candidate (section Kawn improvements).

---

## 9. Other mode snapshots worth recreating
| Mode | Shot | What is distinctive |
|---|---|---|
| Globe full orbit | 09, 21 | circle of Earth about 900 px wide, deep-black space with faint star dots, teal 3D beams (orbital tracks and satellite lines) as 6-10 px glowing cyan strokes shooting off the top; HUD summary reads "GLOBAL"; ALT 24,941,017 M / 18,000,000 M; stars visible; the scope circle outside the Earth shows a very faint ring; labels pile up (radio clusters "58 MUSIC", magnitude chips M2.7 to M5.6 with a red / orange left edge) |
| Oblique city 3D | 22 | photoreal tiles with long shadow blocks; ALT 1772 M; top-down banner absent (Tactical) |
| Airport detail | 24 | pale runway grid inside a bordered tile patch; the surrounding area is blurred (lower-resolution tiles) and the labels are cyan-framed plates; the Location search tray opens above the dock showing a search field and five city chips (AUSTIN, SAN FRANCISCO, NEW YORK, TOKYO, LONDON) |
| Travelling aircraft | 23 | white plane icons with a haze glow and cyan corner brackets, tiny callsign plates "BOX712 FL033"; the Display panel shows BLOOM and SHARPEN rows active |
| Touchdown | 27, 28 | 3D cyan model of an airliner on the runway; photoreal textures; "LOAD FAILED" red chip at the top centre in shot 27 (x 922-998, y 68-88) |
| Panel states | 01, 17 | CCTV panel with a real traffic photo vs an empty dark "no feed" teal gradient box; Street Level with PROVIDERS pill (MAPILLARY) and ALL / 360 / FLAT segmented chips |
| Draw, scenes and street level | 30-36 | not covered here (separate documents) |

---

## What makes it feel the way it does
1. **Separate layers of fiction.** The looks are cheap photographic filters; the HUD supplies the sensor "story". Together they turn the same map into a "satellite feed", an "NVG feed", a "thermal feed".
2. **Recolouring the HUD with the look** (amber, green, white) is the single cheapest cue that sells the sensor; panels staying cyan keeps the tool usable.
3. **Circular masks.** The scope, NVG tube and FLIR lens share one circular shape, so switching looks never moves the focus of the picture.
4. **Text blocks hugging the corners** at constant 36 px margins let the image breathe in the middle; vertical edge text fills the dead space along the sides.
5. **Detection density** gives the world an "ambient intelligence" feel; the fade to nearly nothing outside the circle keeps clutter out of the dark corners.
6. **Cockpit** converts the same data into a first-person instrument; curved tapes and a roll arc follow the circle.
7. **Cyber** shows the shell is a skin: change panel geometry (chamfers) and colour (red / slate) and the product feels like a different tool.

---

## Kawn improvements
| Area | Direction |
|---|---|
| Looks | Replace seven sensor looks with a small set that fits UAE needs: Day (default), Night (low light, warm lamps), Heat (thermal ramp for urban heat and comfort analysis), Dust / Haze (visibility), High-contrast / Print (for government briefs). Keep the 500 ms crossfade and per-look sliders; one "Reset look" button. Drop CRT, Anime, Snow. Keep Noir as "Monochrome" for print. |
| HUD | Replace the intel HUD with a single calm status strip: place name (Arabic and English), freshness ("updated 2 min ago"), altitude or map scale, coordinates on demand. Optional second line for the AI summary with clear "AI summary" labelling. No classification, no ids, no REC unless truly recording. |
| Layouts | Offer "Standard" (status strip + scale), "Analyst" (adds coordinates, bearing, camera tilt, source health) and "Presentation" (nothing but title and logo). Retire Operator / Cyber / Minimal naming. |
| Mobile | HUD collapses to one line in the top safe area; coordinates and altitude behind a tap; no vertical edge text; scope circle optional (default off on phones). |
| RTL / Arabic | mirror strip order; keep latitude / longitude in a fixed LTR run with numerals per setting; no tracked caps; Arabic-friendly line heights (1.4-1.6); do not rotate Arabic text for vertical edges - drop vertical edge text. |
| Light theme | HUD text becomes dark with a pale halo; plates become white at 80%; the scope darkening is replaced by a soft frame blur. |
| Accessibility | no key information communicated by HUD colour alone; labels at least 12 px; detection overlay list is exposed to assistive tech (the reference already does this - keep and extend); respect reduced motion for REC, sweeps, typing, snow; sensor looks must never be required to read data; announce look changes politely. |
| Detection | cap labels to 20 on mobile, 40 on desktop by default (reference default of up to 68 is too busy); use a quieter plate (no accent bar, no scanlines) and an icon in place of colour alone; keep leader lines and sticky corner choice. |
| Scope | keep as an optional "Focus" mode, default off; otherwise full-bleed map with a soft bottom gradient behind the dock. |
| Cockpit | rename "Ride-along"; bilingual tape labels; large readouts at 24 px minimum; keep curved tapes only on tablets and up; phones show a single readout row. |
| Recording | rename to "Present"; keep safe frame (16:9 and 9:16 with thirds), add 1:1 and 4:5; export share image with credits; never hide attribution. |
| Clean UI | keep the exit pill but make it 44 px tall and add an on-screen tap area; allow Escape. |

---

## UNVERIFIED
- Minimal layout has no screenshot; its placement comes from stylesheet numbers.
- Cockpit appearance is from stylesheet and prose only; sizes of the instrument rings, exact tape placement and briefing carousel look were not seen.
- Cyber sonar visibility: the disc is almost invisible in the one still (shot 19), whether it is stronger in motion is not known; the screenshot also shows Normal imagery rather than FLIR, so the automatic look swap was not observed in this still.
- The exact moment the REC dot is lit (it blinks); visible or hidden state in each shot is incidental.
- Whether the preset tile labels are 9 or 11 px.
- Rolling-band position and intensity of the CRT effect in motion; static shots cannot show flicker or jitter.
- NIIRS / GSD formula rendering at extremes (values appear with up to two decimals).
- Dimensions of the loading-failure chip in shot 27 and how long it persists.
- Whether Noir and Anime auto-hide the HUD (shots 13, 14 show the HUD on because the baseline forced it on).

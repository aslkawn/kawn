# 03 - Screen anatomy and layout (desktop)

**Purpose.** Describe where every piece of chrome sits on the reference product's desktop screen, how big it is, what is stacked above what, how regions move when panels open or collapse, and what each state (loading, empty, error, degraded) looks like. Written so a designer can rebuild an equivalent layout without the source. Behaviour (what controls do) lives in the existing behaviour notes; this file is about geometry and appearance. Component-level look (buttons, chips, cards) is in file 04.

**Evidence and measuring convention.**
- Primary: screenshots 06, 10, 11, 12, 17, 19, 22 to 36, read at 1920 x 1080. Numbers confirmed against the product's stylesheets (sizes, offsets, z-order, timings).
- Shots 06 to 12 are at true CSS size (1 shot pixel = 1 CSS px at a 1920 x 1080 viewport). Shots 17 to 36 are scaled to about 0.9 (the capture was a larger viewport, roughly 2133 x 1200, reduced to 1920 x 1080); every pixel number below has been converted back to CSS px using the panel widths that the stylesheets fix (Data Layers 280, Display 272). Where a value is only seen in a shot it is tagged "(shot)"; where the stylesheet fixes it, no tag.
- All positions below assume a 1920 x 1080 viewport. "vh" = viewport height (10.8 px per 1 vh here).
- Wireframes use a 120-column by 34-row grid: 1 column is 16 px, 1 row is about 32 px. They show proportion, not detail.

---

## 1. Layout model in one paragraph

One full-viewport 3D globe sits at the bottom of the stack. Everything else floats above it as translucent dark-glass chrome pinned to the four corners and the bottom centre. There are no pages, no routes, no header bar and no footer bar: the screen is a "heads-up" frame. Left edge: a vertical rail of layer panels. Right edge: a vertical rail of display/context panels. Bottom centre: one wide dock holding two pop-up trays and the voice capsule. Top centre: five round buttons. Four corner text blocks (the "HUD") and two thin side strips fill the remaining margins with telemetry. Large central area (about 1280 x 760) stays free for the globe.

---

## 2. Region catalogue (default Tactical HUD, default theme, 1920 x 1080)

Coordinates are CSS px from the named screen edge. "Box" is left-top to right-bottom.

| # | Region | Anchor and offset | Size | Appearance in one line | Default state |
|---|---|---|---|---|---|
| 1 | Brand block | 36 left, 32 top | logo mark 52 wide; wordmark row 26 px type with 8 px letter spacing, about 325 wide; tagline row beneath at 11 px with 5 px spacing; block about 400 x 60 | glowing cyan globe-eye mark, spaced white capitals with the last word in cyan light weight, dim tagline | visible; non-interactive; narrower widths drop the words |
| 2 | Style indicator | 36 right, 32 top, right aligned | about 110 x 40 | small dim "ACTIVE STYLE" caption over a 16 px cyan value with 5 px spacing and soft glow (shot 06: NORMAL) | visible; hides at 720 px and below |
| 3 | Top-centre action row | centred, 32 top | 5 round buttons of 36 px, 8 px gaps = 212 x 36 (x 854 to 1066) | glass circles, 1 px hairline, 18 px outline icons | visible |
| 4 | Global loading strip | centred, 74 top | auto width, about 258 x 25 in shot 06; max viewport minus 32 | small dark-navy pill (7 px radius), cyan hairline, 8 to 9 px mono caps: grey-cyan label plus bold cyan layer names | hidden until work lasts 160 ms |
| 5 | Traffic-sync chip | centred, 112 top | about 170 x 24 | same pill as 4 with a percentage | hidden |
| 6 | Camera-frames chip | centred, 146 top | about 168 x 24 (shot 06: "LOADING FRAMES 612/3902") | same pill | hidden |
| 7 | Scene runtime caption | centred, 34 top | auto, text 11 px with 2 px spacing | black 45% plate with cyan hairline | hidden unless a scene plays |
| 8 | HUD top-left block | 36 left, about 151 top (14 vh) | about 400 x 120; summary line max 420 | no panel, floating cyan text with corner bracket glyph at its top-left; rows: classification line (12 px bold, 2.5 spacing), id line (13 px), mode word (16 px bold, 4 spacing), "SUMMARY" caption (9 px, 55%), one summary line (11 px, ellipsised) | on |
| 9 | HUD top-right block | 36 right, 151 top | about 250 x 45 | red record dot, UTC clock (13 px, 2 spacing), orbit/pass line (12 px, 70%) | on |
| 10 | HUD bottom-left block | 36 left, bottom edge 126 above screen bottom (2 vh + 6.5 rem) | about 270 x 45 | MGRS line then lat/lon in degrees-minutes-seconds, 13 px, bracket at bottom-left | on |
| 11 | HUD bottom-right block | 36 right, same bottom edge | about 270 x 65 | GSD/NIIRS, altitude/sun, AIS lines right aligned | on |
| 12 | HUD side strips | left 36 and right 36, vertically centred | rotated -90 and +90 degrees, 10 px, 50% | thin vertical telemetry (ONA, COLL clock; BAND, BITS, LVL) | on; hidden at 720 px and below |
| 13 | Left rail: Data Layers | 52 left, top about 285 (26 vh plus measurement) | 280 wide expanded, 176 wide collapsed; measured 605 tall (bottom about 890) | glass panel, 16 radius, scrolling list of layer rows | present; expanded when the viewer lands (shot 06); doc says compact on a brand-new profile |
| 14 | Left rail: Scenes | below Data Layers in the same stack, 8 gap | 360 wide expanded (shot 33 shows 242 in Cyber), 176 collapsed | glass panel with shot list | collapsed |
| 15 | Right rail: Display | 52 right (right edge at 1868), top about 285 | 272 wide expanded, 176 collapsed; rail container 330 wide; measured 577 tall (bottom about 862) in shot 06 | glass panel with a column of toggle rows and sliders, internal scrollbar | collapsed on a new profile; open in the shots |
| 16 | Right rail: CCTV | below Display, 8 gap | 330 wide (shots: about 327), preview 16:9 | glass panel with a dark gradient preview well | collapsed; open in shots 17 to 29 |
| 17 | Right rail: Street Level | below CCTV | 330 wide | glass panel, provider chips, segmented filters | collapsed; open in shots 17 to 32 |
| 18 | Right rail: Weather / Recent Imagery | below Street Level | 330 wide | glass cards | hidden until their layer is on |
| 19 | Right rail: Context | bottom of rail | 330 wide expanded, 176 collapsed | glass panel with mode tiles and contact list | collapsed (shown open in shots 25 to 31) |
| 20 | Bottom dock | centred, bottom 2 vh (22 px) | 474 x 68 in shot 06 (left wing 167, voice 140, right wing 167); capsule protrudes about 15 above | one dark teal-slate bar, 12 radius, three zones separated by hairlines | both wings compact |
| 21 | Dock tray: Location | opens upward from left wing; left edge about 35 left of dock edge | 544 wide (34 rem); height 51 (search) up to about 90 with landmark row; 20 above the dock | darker 94% slate popover, 6.7 radius, small downward notch at its bottom-left | closed |
| 22 | Dock tray: Visual presets | opens upward from right wing; mirrored | 544 wide | same popover, notch bottom-right; seven look tiles then a map-source chip row | closed |
| 23 | Voice capsule | centre of dock, z above wings | 140 x 83 (shot 06) | raised dark-teal capsule with cyan hairline and glow, round mic orbit 39 px on the left, tiny meters right | off |
| 24 | Voice card | above capsule, 11 above | 352 wide (22 rem) x up to 24 rem | dark card with captions and numbered plan | hidden |
| 25 | Voice error tray | above capsule, centred | 24 rem max | wine-red card, red hairline | hidden |
| 26 | Credits line | 24 left, bottom edge about 102 above screen bottom (2 vh + 5 rem) | single line about 440 wide, 11 px | logo + "Powered by Esri" + licence links, underlined, 70% white | always visible, even in clean view and recording |
| 27 | Toast | centred, bottom 100 | auto, about 143 x 35 (shot 12: "TILTED VIEW") | dark glass pill, 8 radius, cyan border, cyan 11 px caps with 2 spacing, soft cyan glow | hidden; 2 s display |
| 28 | Orbit badge | 36 right, 70 top | about 70 x 16 | cyan 10 px caps with rotating arrow glyph | hidden |
| 29 | Settings chip | 14 right, 14 bottom | about 130 x 28 | small slate pill with bolt icon and key-count label | only on a local server |
| 30 | First-run card | dead centre | 544 wide (34 rem), tall about 400 | near-black teal gradient card, 10 radius, bright cyan hairline, scan line across the top edge | appears once per browser session |
| 31 | Settings dialog | dead centre | 576 wide (36 rem) | same card family as 30 | closed |
| 32 | Clean-view exit | centred horizontally, 18 top | pill about 150 x 30 | dark pill with 25% white hairline, 10 px caps | only in clean view |
| 33 | Loading screen | full screen | full | page-colour background, centred logo 128 px pulsing, wordmark 18 px, status line 11 px | covers everything until first restore + at least 1 s |
| 34 | Safe-frame guide | centred | 90% of width at 16:9 (1728 x 972, shots 34 to 36) | 1 px white 28% rectangle plus rule-of-thirds lines | only while recording or scene playback |

Region 15 to 19 live in one right-hand rail whose container is 330 wide, right aligned, with 8 px gaps between panels, and rows align to its right edge; collapsed panels are 176 wide and flush right.

---

## 3. Z-order (back to front)

| Layer | Content | Notes |
|---|---|---|
| 0 | Globe canvas | whole viewport |
| 1 | Circular scope mask (vignette) | darkens everything outside a big keyhole circle; its edge is the soft dark ring seen in every shot |
| 2 | Celestial ring, HUD text layer | HUD is purely visual and does not take pointer events |
| 5 | Detection (label-allocation) blend surface | screen-blended against the globe |
| 6 | World label canvas | all in-world cards, call-outs, leader lines |
| 7 | Cockpit cloud pass | cockpit only |
| 90 | Credits line | |
| 95 | Safe-frame guide | |
| 100 | Brand block, style indicator, top-centre buttons, left rail, dock-adjacent panels | the promotable panel band runs 100 to 139: a panel that is touched moves up inside this band |
| 101 to 105 | Traffic chip (101), loading strip (102), scene caption (105) | |
| 110 | Right rail and Display | |
| 111 | Floating (lifted-out) CCTV or Street Level windows | above every docked panel |
| 141 | Expanded street-level viewer | dialog-like, over rails |
| 145 | Bottom dock | |
| 150 | Voice capsule (above dock) | when its card is open it rises to just above the rail band |
| 175 | First-run card | |
| 190 | Settings dialog | |
| 200 | Toast, credits lightbox | |
| 300 | Clean-view exit button | |
| 1000 | Loading screen | |

Reading: the chrome never covers the credit line (the rails are measured to stop above it), and nothing but the exit button survives in clean view, plus the credit line.

---

## 4. How regions react

### 4.1 Left rail
- The rail is a column with 8 px gaps; each expanded panel gets an allocated height from a measuring pass that subtracts the HUD blocks, the credits line and the dock; the panel's list scrolls internally while its header stays fixed (sticky).
- Collapsing Data Layers shrinks it to a 176 x about 42 launcher with "+" (shot 33 top-left of Cyber). Expanding any collapsed panel below it pushes the others up only if it needs the room; panels below an expanded one are hidden entirely when the rail is in focus mode (shot 33: Data Layers collapsed to a strip while Scenes is expanded).
- Expanded width: Data Layers 280; Scenes and CCTV 360 (default theme); Cyber theme clamps all to about 242 (shot 19, 33).
- Top edge of the whole rail drops to the first free row under the brand block and HUD top-left block; in Cyber the rail starts at about 220 because that theme's HUD box is shorter.

### 4.2 Right rail
- Container 330 wide, right edge 52 from the screen; its top is aligned with the left rail's settled top; max height roughly 70 vh minus HUD clearance.
- Default theme: Display may stay open next to one contextual panel; opening a contextual panel (CCTV, Context) can collapse the other contextual one. The shots (17 to 29) nevertheless show Display, CCTV, Street Level and Context all drawn at once, each squeezed; the rail then clips (see 4.6). UNVERIFIED whether that is a deliberate "tail" layout or a transient state.
- When the combined height exceeds the corridor, the rail does not scroll as a whole; instead each open panel gets a shorter allocated height and scrolls internally (Display shows only HUD, layout, dense in shot 17; Display shows SCOPE to CLEAN UI in shot 31 after the user scrolled it).
- Collapsed panels are 176 wide and flush right; collapsed panels stack as a column of header-only plates (Cyber, shot 19: DISPLAY, CCTV, STREET LEVEL, CONTEXT, each about 42 tall, 8 apart, at x 1685 to 1863).
- Cyber theme: one-at-a-time; opening one collapses its peers.

### 4.3 Bottom dock
- Both wings are compact. Hover (mouse only) 140 ms opens the tray upward; leaving closes after 420 ms unless pinned.
- A tray never pushes other chrome; it overlays the globe and any HUD blocks above the dock. The pin control is a 26 px round button hanging off the tray's top corner (left tray: top-left; right tray: top-right; shots 10, 11, 24 show the right-hand circle with a bell-like pin glyph at the tray's top-right).
- Pinned trays stack upward: second pinned tray sits above the first by its height plus gap, and the voice card/error tray lift by the pinned stack height.
- Opening one tray makes the voice capsule and other wing keep their widths on desktop (they shrink only under 900 px).
- The credit line and HUD bottom blocks are fixed above the dock so a closed tray never covers them; an open tray does.

### 4.4 HUD
- Tactical: four corner blocks with brackets, no bars. Operator: framed boxes (1 px white 16% border, 8 radius, black 36% fill) plus a top bar and a bottom bar (shots 17, 22 to 36). Minimal: only top-left and bottom-left, low and small. Cyber: red frame variants.
- In Operator the top bar spans about 646 x 24 at x 637 to 1283, y 109 to 132 (shot 17): left text "TOP SECRET // SI-TK // NOFORN", centre id, right "PAGE 1/1"; bottom bar about 610 wide at y 895 to 916 carries MGRS and lat/lon. The right rail is then drawn lower so it clears the top-right box.

### 4.5 Modes that remove chrome
| Mode | What disappears | What stays |
|---|---|---|
| Clean view | brand, indicator, actions, status chips, both rails, dock, HUD, first-run, settings chip | globe, credit line, a small centred exit pill (top 18) |
| Recording or scene playback (shots 34 to 36) | all panels and dock, brand, top buttons, status chips | HUD (minimal-style, amber in the CRT look), safe-frame rectangle with thirds, credits, a caption at top-centre; the four round action buttons are removed too |
| Cockpit | left/right panels, CCTV, Context | cockpit overlays, its own rails |
| Loading | all of it | loading screen |

### 4.6 Clipping problems visible in the shots
- Shot 25 and 29: the Context panel is cut at the bottom edge of the rail, so the contact list (the roster) is not visible; only the mode tiles, the COCKPIT and SEARCH NEARBY SITES buttons, the window line and the top of PREVIOUS / FOCUS / NEXT show. The panel has a plain light scroll thumb that does not match the cyan scrollbars elsewhere.
- Shot 29 and 30: the left Data Layers list is clipped after "Cameras / Recent".
- Shot 27: the Display panel keeps only its top 4 rows because CCTV and Street Level took the room.

---

## 5. State look (what each state looks like on screen)

| State | Where | What the user sees |
|---|---|---|
| App loading | whole screen | black-blue screen, logo breathing (scale 0.98 to 1.02, opacity 0.72 to 1, 2 s loop), wordmark, a dim status line that pulses 0.4 to 1 opacity; fades out over 0.8 s |
| Layer loading | layer row | grey chip text "ENABLING" (shot 06) then cyan "LOADING"; meta line text; after 160 ms the top strip shows "LOADING LIVE DATA" plus names |
| Refreshing | strip | same strip, label "REFRESHING LIVE DATA" (shots 11, 12, 22 onwards) |
| Long wait | strip | after 30 s the strip changes to a longer-wait skin |
| Success | strip | lingers 2.2 s |
| Error | strip | red-salmon border and text (shot 28: "LOAD FAILED" as a small red-bordered pill under the top buttons), lingers 5 s |
| Retry | strip | amber border and text |
| Cancelled | strip | amber text, centred |
| Acquiring (shared target) | strip | cyan border with a soft outer cyan glow, stays until resolved |
| Empty list | panels | dashed 1 px grey-white 14% rounded box with centred 9 px mono hint (scene shot list: "capture a shot" message); contact roster shows a standby note |
| Layer off | row | chip "OFF" in dim grey, count dash, meta "source - never" |
| Layer on | row | chip "ON" cyan outline with cyan wash and faint glow; meta "source - Ns/m/h ago" |
| Degraded | row chip | orange text and border (stale, partial, fallback use amber/yellow; degraded uses orange) |
| Fallback | row | amber chip "FALLBACK" with count at left (shot 22: count 247 then chip); meta names the source and radius ("adsb.lol", "250nm") |
| Unavailable | row | red chip "UNAVAILABLE", cannot toggle; meta names the missing key (shot 06: key not set; shot 32: "KEY REQUIRED") |
| Stale object | in-world card | the word "STALE" appended to the card title (shots 27, 30: "BAW615 - 1700 ft - 152 kts - STALE") |
| Camera frame loading | CCTV preview | shimmer sweep across the well every 1.4 s with centred "ACQUIRING FRAME" caption 8 px, 2.2 spacing |
| Camera frame error | CCTV badge | badge turns red |
| Street level key | panel header | amber outlined pill "KEY REQUIRED" (shot 32), filters dimmed 45% |
| Voice error | dock | capsule border turns wine red; a red tray appears above with header, message, hint and a "DISMISS" button |
| Toast | bottom centre | see region 27 |
| Storage blocked, key service missing | | chip removed or silent |
| Scene running | left panel | all buttons but STOP at 45% opacity; progress pill fills |
| Empty globe (no layers) | | just terrain, the HUD and the empty panels; the shot-06 look with many labels is the "everything loaded" case |

---

## 6. Wireframes

Legend for all frames: `#` = glass panel edge, `.` = globe visible, `[ ]` = control, `*` = marker/label on the globe. Frame is 120 columns (1920 px) by 34 rows (1080 px).

### 6.1 Default globe view (shot 06, 10)

```
 0         1         2         3         4         5         6         7         8         9        10        11
 012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789
 (o) G O D ' S  E Y E  V I E W                    (x)(link)(tilt)(N)(globe)                            ACTIVE STYLE
     NO PLACE LEFT BEHIND                          [ LOADING LIVE DATA  LIVE FLIGHTS ]                       NORMAL
                                                    [ LOADING FRAMES 612/3902 ]
                                                                                                          
  TOP SECRET // SI-TK // NOFORN                                                              REC 2026-10-15 11:48:40Z
  KH11-4179 OPS-4155                                                                           ORB: 47351 PASS: ...
  NORMAL                                                                                                    
  SUMMARY / one-line region text                                                                            
  ..                                                                                                        
   ##############################                                                  ##############################
   # DATA LAYERS            [-] #                                                  # DISPLAY             [>]  #
   # MOVEMENT                   #               globe / map                         # [ HUD            ]       #
   # sat      CelesTrak [OFF]  #                                                    # LAYOUT [Tactical v]        #
   # flights  Flights   [ENAB.]#                    * label cards                    # [ DENSE          ]       #
   # mil.fl.  ...       [OFF]  #                    * camera tags                    #  density ---o--- 75%        #
   # ADS-B               [OFF]  #                                                    #  ALLOC [ELASTIC|WEIGHTED]  #
   # vessels  UNAVAIL.  [!]     #                                                    #  fade ---o--- 7%           #
   # traffic             [OFF]  #                                                    # [ 3D ] [ SCOPE ] [ DRAW ]  #
   # transit             [ON]   #                                                    # [ CELESTIAL ] [ CLEAN UI ] #
   # bike share          [ON]   #                                                    # [ BLOOM ] [ SHARPEN ]      #
   # CAMERAS                    #                                                    ##############################
   # cameras 3.9K       [LOAD]  #                                                                      
   ##############################                                      GSD / NIIRS
  MGRS 31U BT 9321 1062                                                                ALT / SUN  AIS --
  51 30 32.04 N  000 01 12.00 W
  (cesium) Powered by Esri . (c) OpenStreetMap  Data attribution
                                     ###############[ LOCATION ]#(mic)#[ VISUAL PRESETS ]###############
                                     #  location: --  |  ON/OFF  |  Style: NORMAL        #
                                     ##########################################################
```

Notes: left panel x 52 to 332, right panel x 1597 to 1868, both start at y about 285. The dock is the only element touching the lower edge. In shot 10 (whole-Earth view) the dock tray for Location is open above the dock: a 544 x 51 bar with magnifier circle, "Search any location..." placeholder and five city chips, ringed by a faint circular scope; a floating pin circle sits at its right end.

### 6.2 Tracked-flight view with card (shots 26, 27, 29)

```
  (brand)                          (x)(link)(tilt)(N)(globe)                                  ACTIVE STYLE
                                  [ REFRESHING LIVE DATA  LIVE FLIGHTS . MILITARY FLIGHTS ]
                     [=== TOP SECRET // SI-TK // NOFORN ====== KH11-4165 ======= PAGE 1/1 ===]   (operator bar)
  +--------------------------+                                                    +----------------------+
  | TOP SECRET // ...        |                                                    | REC 12:15:02Z        |
  | NORMAL  /  summary       |                                                    | ORB / PASS           |
  +--------------------------+                                                    +----------------------+
   ########################                     +----------------------------+     ##########################
   # DATA LAYERS       [-]#                     | BAW615 . 1700 ft . 81 kts  |     # DISPLAY           [>]  #
   # ...                  #                     | British Airways . A320     |     # [HUD] LAYOUT Operator  #
   # Live Flights 830 [FALLBACK]                  | BDS -> LHR                 |     # [DENSE]                #
   # Military Fl. 204 [ON]#                     +-------------+--------------+     ########################
   # Live Vessels [UNAVAILABLE]                             [  /\  ]  <- aircraft in 4-corner reticle     ##########################
   # ...                  #                     ----cyan trail line to the left----   # CCTV             [>]  #
   ########################                                                           #  [ dark preview 16:9 ] #
  +--------------+                         near-contact plates: "SHT6Y . 0.9 km"       ##########################
  | MGRS ...     |                         (small 8 px plates with arrow glyph)       # STREET LEVEL  [OFF]    #
  +--------------+                     [ MGRS 30U XC 7803 0428   LAT ... LON ... ]    # providers [MAPILLARY]  #
  (cesium) credits                                                                     # CONTEXT  (radio) [>]   #
                                                                                       # [CONTACTS][SPACE MISS.]#
                           [ LOCATION ] (mic) [ VISUAL PRESETS ]                       # [COCKPIT][SEARCH NEARBY]#
                                                                                       # BAW615 . 250 km window #
```

Notes: the aircraft is drawn as a large flat silhouette with a 2 px cyan fill when tracked and white-grey otherwise, inside a four-corner bracket (about 44 px). The card sits above it with a short vertical leader; the card is a dark plate with a bright cyan top edge when selected. A thin cyan line (the track) runs from the aircraft out to the left or bottom of the screen. Operator HUD adds the banner strip across the top and the boxed corner blocks. The right rail is cramped: four panels at once.

### 6.3 Layers panel open, scrolled (shots 06, 10, 22, 32)

```
 ##########################################
 # DATA LAYERS                       [-]  #   header 14px padding, hairline divider fades to the right
 #----------------------------------------#
 # MOVEMENT                               #   group caption 10px caps, hairline above each group after the first
 #  [ico] Satellites        --   [ OFF ]  #   row: 16px icon, 13px name, count 11px mono, chip 22px tall
 #        CelesTrak . never               #   meta line 9px dim, indented 26
 #  [ico] Live Flights  247  [FALLBACK ]  #
 #        FALLBACK . adsb.lol . 250nm     #
 #  [ico] Military Flights 184 [ ON ]     #
 #  [ico] Live Vessels    [UNAVAILABLE]   #   row tinted on hover/voice focus
 #        AISSTREAM key not set           #
 # CAMERAS                                #
 #  [ico] Cameras 3.9K   [ LOADING ]      #
 #  [ico] Street Level  [UNAVAILABLE]     #
 # INFRASTRUCTURE                         #
 #  [ico] Mapped ALPR Cameras  [ ON ]     #
 #        meta, 3 lines, [SHOW NEAREST]   #   sub-controls appear only when ON
 #        * Camera badges 6               #
 # EVENTS ...                          ||  #   6px cyan scrollbar thumb at the right edge
 ##########################################
```

280 wide; list scrolls, header sticks; panel inner padding 14 top, 16 sides, 12 bottom.

### 6.4 Display panel open (shots 06, 22)

```
 ############################################
 # DISPLAY                              [>] #   collapse square 20px
 # +--------------------------------------+ #
 # | (o) HUD                       (cyan) | #   toggle row 38px high, 8px radius
 # +--------------------------------------+ #
 #  LAYOUT [ Tactical                  v ]  #   select 24px high
 # +--------------------------------------+ #
 # | [] DENSE                     (green) | #   detect toggle uses green when on
 # +--------------------------------------+ #
 #  DENSITY  --------o-----------  75%      #   3px track, 10px thumb, % value in cyan
 #  ALLOCATION [ ELASTIC | WEIGHTED ]       #   segmented 22px, active = solid cyan fill, black text
 #  FADE     ---o----------------   7%      #
 #  OUTSIDE  o------------------    1%      #
 # | 3D |  MODELS [PROXIMITY|ALL]            #
 # | SCOPE |  FEATHER ------------o 100%     #
 # | DRAW | | CELESTIAL | | CLEAN UI |       #   idle rows: dim label, hairline box
 # | BLOOM | | SHARPEN ----o-- 49% |         #
 ############################################
```

Panel: 272 wide, 8 px inner padding, 7 px gap between rows, internal scroll (cyan thin scrollbar at x 1862).

### 6.5 Scenes panel (shot 33, Cyber / FLIR look)

```
 ## DATA LAYERS [+] ##                 <- collapsed strip, 176 x 42
 ###################################
 # SCENES                      [-]  #
 # [ Nepal flood...  v ] [NEW][DEL] #   dropdown + two buttons
 # [CAPTURE SHOT][UPDATE SHOT]       #
 # +-------------------------------+#
 # | Global Incident Context  [LOAD][DEL]   active row: cyan border + 8% cyan fill
 # | NORMAL . OFF . 4.0s + 0.9s    |#
 # +-------------------------------+#
 # | Nepal-Focused Globe Rotation  |#   list max 160 tall, min 96, scrolls
 # | ...                           |#
 # [ START ][ STOP ]                 #
 # [ NEXT ]                          #
 # [EXPORT PRESETS][ IMPORT ]        #   IMPORT highlighted
 # [ RUN LOG ]  (disabled)           #
 # (======      ) progress pill 18px #
 # Ready                             #
 # [EDIT DETAILS] [SHARE SCENE]      #
 ###################################
```

### 6.6 Draw panel (shots 30, 31)

```
 ##########################################
 # SCOPE  (cyan row)                      #   Display scrolled so SCOPE is first visible row
 #  FEATHER ---------------------o 100%   #
 # +--------------------------------------+
 # | DRAW  (active)                       |#
 # |  SHAPE [ AREA | LINE | PIN ]          |#   AREA active, solid cyan
 # |  [ Label (optional)     ] [Primar v ] |#   text input flex + colour select 74 wide
 # |  hint line 8px: "click vertices..."    |#
 # +--------------------------------------+
 # [ CELESTIAL ] [ CLEAN UI ] ...          #
 ##########################################
```

On the globe the finished area is an amber outline with a translucent amber fill and a dark label chip ("Heathrow stand area", mono 10 px) with a leader dot at one vertex (shot 31). A larger outer rectangle outlines the same polygon's bounding tilt (shot 30/31 show a thin amber frame about 700 x 600 around the filled shape).

### 6.7 Street-level viewer

No shot shows an opened viewer (shot 32 is the key-required state). From styles: when an image opens it appears first inside the Street Level panel (16:10 well, min height 180), and "EXPAND" lifts it to a big dialog.

```
 Docked (inside the 330 rail panel)            Expanded
 +----------------------------------+        +----------------------------------------------------------+
 | STREET LEVEL        [ON]  [<]    |        | [FIT|FILL] [FOLLOW] [SHRINK . ESC]                   (x)  |   bar 8px pad
 | providers (o MAPILLARY)(o GOOGLE)|        +----------------------------------------------------------+
 | IMAGERY [ALL|360|FLAT] SINCE --o |        |                                                          |
 | legend: --- coverage colours     |        |                    photo / panorama                      |
 | coverage line / [OPEN STREET VIEW]|       |                                                          |
 | +------------------------------+ |        |                                                          |
 | | [EXPAND][FIT/FILL][FOLLOW] (x)| |       +----------------------------------------------------------+
 | |      photo 16:10             | |        | author . date . provider link                            |
 | | author . date . link         | |        +----------------------------------------------------------+
 | +------------------------------+ |        4 vw left/right, 5 vh top, 9 vh bottom, 12 radius, cyan hairline
 +----------------------------------+
```

Docked viewer: 12 radius, 1 px hairline, black well; top bar dark 85% with tool buttons 8 px caps; caption strip black 70% with provider link in a green accent. Spinner 28 px, 0.9 s rotation, on a 35% black scrim while loading. Expanded: the whole thing is near-full-screen over the globe, z above rails.

### 6.8 CCTV panel (shots 17, 25, 26, 27)

```
 ##########################################
 # CCTV                              [>]  #   header only when collapsed (176 x 42)
 # [ CCTV OFF ] [ NEAREST ]               #
 # [ PREV ] [ camera v ] [ NEXT ]         #
 # [FOCUS] [COVERAGE] [AUTO HOP]          #
 # +------------------------------------+ #
 # |  dark teal gradient well 16:9      | #   shimmer + "ACQUIRING FRAME" when loading
 # |  (empty in the screenshots)        | #   min height 150
 # +------------------------------------+ #
 # (source badge pill)  meta line         #
 # [PROJECTION] [CAL . CURATED]           #
 # [ calibration block: HDG PITCH FOV ... ]#
 # scene summary typed in 10px            #
 ##########################################
```

In the shots only the header and the empty well are visible because the rail clips the rest; the well is a dark gradient with a faint cyan radial glow top-left, 10 radius, 1 px hairline.

### 6.9 Contacts roster (Context panel; shots 25, 26, 29, 30)

```
 ##########################################
 # CONTEXT                (radio)  [>]    #
 # [ (o) CONTACTS ] [ rocket SPACE MISSIONS ]     two tiles, equal width, 28 high
 # [ plane COCKPIT ] [ SEARCH NEARBY SITES ] [>>] 
 # UAL935 . 250 km FLIGHT / VESSEL WINDOW   9px
 # [ PREVIOUS ] [ FOCUS ] [ NEXT ]          3 equal buttons
 # --- cohort rows (below the fold in shots) ---
 #  CIVIL FLIGHTS  12  source . reason
 #    > BAW99XC  1.6 km  BRG 233
 #    > SWR3CT   1.2 km  BRG 217
 #  MILITARY FLIGHTS  2 ...
 #  VESSELS  -- ...
 #  MAPPED SITES  5 ...
 # footnote: absence of data is not evidence of absence
 ##########################################
```

The roster rows (cohort heading plus up to three entries per page, pages auto-rotate every 10 s) are described from the behaviour notes; the shots never show them because the panel is cut off. Each entry is a full-width button with a leading rank number, a mono callsign, distance and bearing; unavailable entries are plain text.

---

## 7. What makes it feel the way it does

1. **Everything is a floating pane over one picture.** No bar, no frame: panels are 72%-opaque near-black blue glass with a 24 px blur and slight saturation boost, so the globe bleeds through every surface. The user never feels "in an app", but "inside an instrument".
2. **A strict outer margin of 36 to 52 px.** Corner text blocks at 36, rails at 52, brand at 36: the screen reads as a framed viewfinder. The dark scope vignette and circular ring reinforce a lens.
3. **Tiny monospace caps with wide letter spacing** (8 to 11 px, 1 to 8 px spacing) on a single accent colour; only the layer rows break into 13 px sans names.
4. **One accent (cyan, #00d4ff) with a second family for status** (green for dense/detect, amber for fallback and warnings, orange for degraded, red for unavailable and errors). Nothing else is saturated.
5. **Glow rather than shadow.** Active controls get 10 to 20 px cyan glows; panels sit on a faint radial glow beneath them (opacity about 25%).
6. **Density is high.** Row pitch of 45 to 50 px in rails, 9 px meta lines, 4 px chip radii. Information over comfort.
7. **Motion is quiet**: 150 ms hovers (1 px lift, 10% scale on top buttons), 300 ms panel transitions, 180 ms tray fades, one ambient pulse (loading logo) and one ambient shimmer (camera frame).
8. **The dock is a stage**: a raised voice capsule that overlaps the dock by 15 px is the only element that breaks the baseline.

---

## 8. Kawn improvements

- **Mobile first.** The reference collapses to two scroll bands at 720 px; Kawn should use a bottom sheet with peek/half/full snap points, a thumb-reach action bar, 44 px minimum targets, trays as sheets. Desktop keeps the left/right/bottom model below.
- **Arabic and right-to-left.** Mirror the whole frame: layer rail right, context rail left, dock wings swapped, collapse chevrons flipped, slider fill reversed, brand block to the right edge. HUD blocks mirror too. Letter-spaced capitals do not work for Arabic: use normal tracking and a minimum 14 px text size for rows (rows in the reference are 13 px sans plus 9 px meta).
- **Light theme and a calm civic tone.** Replace the "TOP SECRET" strip and ISR vocabulary (MGRS, NIIRS, ONA, orbit/pass) with city-readable telemetry (district, elevation, local time, data freshness). Offer a light glass variant (white 78% surface, 8% black hairline) and a high-contrast variant; keep the same layout.
- **Type size and contrast.** Raise the smallest sizes from 8 and 9 px to at least 12 px for anything a user must read; keep tiny text only for ornament. Check contrast of 30% and 50% text (the reference's dimmest tier fails AA).
- **Fix the clipping.** The reference lets the Context roster fall off the bottom of the rail (shots 25 to 30). Kawn should reserve a minimum height for the list, scroll the rail rather than the panels, and keep one primary panel open at a time on laptops.
- **Single scrollbar style.** The reference mixes cyan thin bars, a light native bar (Context) and hidden bars (dock row). Standardise.
- **Reduce simultaneous panels.** Never show four rail panels at once; use tabs in one context panel (Contacts / Layers detail / Camera) with the layer rail left.
- **Predictable top row.** Keep the five round buttons but add visible text labels on hover and in a first-run tour; add tooltips with shortcuts.
- **Shortcuts and focus.** Keep the white 2 px focus ring; ensure no region sits under the HUD text.
- **Loading strip and toast** become one status area (live region) with severity icon, not colour only.
- **Credits** stay bottom-left but become a small button that opens a lightbox; the reference's long inline credit string (shot 32) grows with every provider and collides with the HUD bottom block.
- **Scenes** rename to "Tours" or "Stories" and add a minimal timeline bar instead of a button column.

---

## UNVERIFIED

- Exact vertical size of the left rail panels (measured from shots; actual value comes from a runtime measuring pass).
- Whether the right rail really shows four open panels at once by design (shots 17 to 29) or only after the user opened them; the stylesheets suggest Display + one contextual panel.
- Appearance of the opened street-level viewer and the roster rows (no shot; built from styles and behaviour notes).
- First-run card and settings dialog appearance in a screenshot (geometry from styles only).
- Operator HUD at other widths; Minimal HUD placement in a shot.
- Exact top position difference between shots 06 to 12 and 17 onwards (brand and buttons appear about 5 px higher in the later shots after un-scaling; likely a viewport difference).
- Cockpit mode layout (not in the listed shots).
- Dock behaviour at pinned state (no shot).

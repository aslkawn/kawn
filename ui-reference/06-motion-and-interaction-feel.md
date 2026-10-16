# 06 - Motion and interaction feel of the original app

## Purpose
Give a designer numbers for timing, easing, feedback and sound so Kawn can reproduce the same sense of responsiveness without copying it. Sources: stylesheet and script review (primary for numbers), unit tests that pin reduced-motion behaviour, and live confirmation of computed transition values on the local copy (panel open/close read from the running page). Hover/press were not exercised with a physical pointer, so those entries come from the styles.

## 1. Global timing tokens
| Token role | Value | Easing |
|---|---|---|
| Fast transition (hover, colour, small transforms) | 150 ms | cubic-bezier 0.4, 0, 0.2, 1 (standard material-style ease) |
| Smooth transition (panels, large surfaces) | 300 ms | same curve |
| Dock hover (border, background, shadow) | 160 ms | ease |
| Fades of small labels, overlays | 120-250 ms | ease or linear |
So the app lives on two beats: about 150 ms for feedback and 300 ms for movement.

## 2. Panels and trays
| Interaction | Duration | Easing | What moves | Confirmed live |
|---|---|---|---|---|
| Layers / Scenes panel open or close | 300 ms | standard curve | opacity, visibility and transform together; panel position (top/bottom) also eases at 150 ms when neighbours reflow | yes: computed transition read as opacity+visibility+transform 0.3 s |
| Panel height changes (max-height, top offset) | 150 ms | standard | neighbours slide when a panel opens above them | from styles |
| Right-column Display/CCTV/Street Level cards | "all" transition at default timing | default | open/close is an immediate-feeling snap (computed transition on the panel was a catch-all, with no explicit duration, so it uses the browser default of 0 s) | yes |
| Dock popover (Location, Visual Presets tray) opening | opacity 180 ms ease, transform 220 ms with a springy-out curve (0.2, 0.8, 0.2, 1), starts offset down about 0.55 rem and scaled to 98.5 percent; visibility flips after 180 ms on close so it can fade out | | from styles |
| Dock tray reveal on first show | 180-220 ms ease-out | | from styles |
| Dock buttons hover | border, background, shadow 160 ms ease | | from styles |
| Dock label icon recolour | 160 ms ease | | from styles |
| Cockpit brief card enter | 220 ms ease-out | | from styles |
| Cockpit width/radius morph (expandable controls) | 180 ms ease | | from styles |
| Cockpit number readouts | digits roll up or down in 200 ms (0.2, 0.75, 0.25, 1) | | from styles |
| Voice card appear | 180 ms springy-out | | from styles |
| Pointer reticle appear | 220 ms springy-out | | from styles |
| Slider fills, tuner needle | 80-100 ms linear (needle moves left with 180 ms ease-out when snapping) | | from styles |
| Splitflap value change (airport-board style flip on status text) | 190 ms per flip in and out, 500 ms total width settle | | from styles |
| Annotation / embedded leader-line sequence | exit 160-300 ms ease-in, enter 240-420 ms ease-out, body enters after a 510 ms delay | staggered | from styles |
| First-run card fade | opacity 220 ms ease; scroll-linked fade of the card edge while scrolling | | from styles |

## 3. Ambient and looping animation
| Element | Period | Style |
|---|---|---|
| Logo pulse | 2 s ease-in-out infinite | glow breathing |
| Live indicator dot ("cockpit live") | 1.6 s ease-in-out infinite | opacity pulse |
| Altitude pointer | 1.8 s | gentle bob |
| Radio-title broadcast glow | 1.8 s | |
| Detection pulse | 2 s | ring pulse |
| Loading chip spinner | 0.9 s linear | rotation |
| CCTV frame shimmer | 1.4 s linear | placeholder sweep |
| Mission thrust wave | 0.9 s ease-in-out | |
| Voice level bars | 680 ms ease-in-out alternate | |
| Voice scan / lock | 2.8 s linear / 1.45 s ease-in-out | scanning bar then lock pulse |
| Sonar sweep (cyber look) | 5.2 s linear | radar sweep |
| Cockpit instruments | drift 36-48 s linear, rotate 24 s, trim 8 s, support 6 s, breathe 4.8 s, bands 10-13 s | slow, near-subliminal |
Forty keyframe sets in total. The ambient slow ones are what make the HUD feel alive when you are not touching anything.

## 4. Loading feedback ("strip"/chip)
| Phase | Timing |
|---|---|
| Delay before a loading indicator appears | 160 ms (avoids flicker for fast loads) |
| Successful completion stays visible | 2.2 s |
| Failure stays visible | 5 s (or persists until dismissed for permanent faults) |
| "Taking a long time" threshold | 30 s |
| Traffic sync confirmation flash | 1.5 s, no progress number shown when settled |
| Progress | integer percent 0-100, shown only while real progress exists |
Observed on load in the live copy: the globe imagery resolves while the first-run card is already up; a blurry low-res tile patch on both sides of the viewfinder resolves over several seconds (visible in the first capture), so imagery fades by tile swap, not by a managed fade.

## 5. Toasts and notices
I found no stand-alone toast stylesheet; transient notices reuse the loading chip pattern (appear after a short delay, dwell 2.2 s for success or 5 s for failure, fade). Exact fade timings: UNVERIFIED (likely the 120-250 ms opacity fades above).

## 6. Camera motion
| Movement | Duration | Notes |
|---|---|---|
| Fly to a searched place (default helper) | 3.0 s | |
| Opening flight to the home city | 4.0 s | |
| Quick re-orient (short hops) | 0.9 s | |
| Reset globe, north-up, tilt | not read from source; feel is a short ease (about 1-2 s) | UNVERIFIED value |
| Reduced motion | roll and altitude shaping flatten; verbs that rely on a leading gaze are dropped; tests assert the camera path stops the moment the user takes control | tests |
The load-time alt changed from 25 km to 600 m after dismissing the first-run card in my capture, so choosing/dismissing performs an automatic descent to the street (a flight of several seconds).
User input cancels automatic flights immediately (tested behaviour).

## 7. Hover, press, focus and cursor
- Hover: most surfaces brighten border and background over 150-160 ms; the glass card gets a lighter border (about 8 percent white to 15 percent white) and brighter fill. Rows in the layer list get a background tint on hover over 150 ms. Mission roster items get a tint and a leading marker.
- Press: only three active-state rules exist; buttons lift by 1-2 px on hover (translate up) and drop back on press. There is no scale-down press feedback and no ripple. On touch there is no hover, so the first feedback on a tap is the state change itself.
- Focus: about 80 focus-visible rules, so keyboard focus is widely styled (cyan outline family).
- Cursor map: pointer on 83 rules; default on 11 (labels, headers); not-allowed on 10 (disabled switches); wait on 8 (busy); grab/grabbing on panel drag handles; crosshair on 2 (scope/draw); ew-resize on 5 (sliders, tuner), ns-resize and diagonal resize on the resizable panels; text on one input. Globe itself shows the default Cesium grab hand (inferred).
- Drag: panels have drag handles (grab then grabbing) and resize corners; sliders and the radio tuner disable browser touch panning on themselves so they can be dragged with a finger.

## 8. Scrolling behaviour
- Scrollbars are custom: thin cyan thumb at about 40-50 percent opacity on a nearly transparent track, visible in the Layers and Display panels (a bright cyan thumb is seen at the right edge of the Layers panel in the capture).
- Scroll containment: 10 scrollable areas use "contain" so wheel/touch scrolling does not leak to the globe or page.
- Smooth scrolling is enabled in three places; auto in one.
- Layers list scrolls inside a fixed-height panel; the page body itself never scrolls.
- Mouse wheel over the globe zooms; over a panel it scrolls the panel (via the containment above).

## 9. Reduced motion
Twelve reduced-motion blocks cover: first-run card, status/split-flap (duration forced to 1 ms), voice card and visualiser, dock sliding, cyber effects (sonar, scan), cockpit instruments, bhote/annotation sequence, layers panel, foundation. Typical treatment: animations set to none and transitions to none (the dialog code even avoids waiting on a transition-end that would never fire). Scripts also query the preference for the logo gaze, sonar, cloud effects, radio needle, split-flap and camera verbs. The globe's own camera flights are flattened rather than cancelled.

## 10. Sound cues
| Cue | Spec |
|---|---|
| Voice "still working" earcon | two soft sine notes, 880 Hz then 1320 Hz 90 ms later; each note ramps in over 12 ms and decays exponentially over 90 ms; peak gain 0.035 (very quiet); repeats every 1.1 s; only plays when the microphone is muted (so it is never mistaken for the user's speech) |
| Radio tuning static | band-limited noise (low-pass at about 1.65 kHz), gain ramps to its target in 25 ms, level scales at 3 percent of the user's volume, capped; played only while tuning between stations |
| Radio audio streams | actual radio audio when a station is chosen |
| Spoken assistant | synthesised voice replies |
No click, success or error sounds for ordinary UI; the app is silent unless voice or radio is on.

## What makes it feel the way it does
- Two durations (150/300 ms) with one ease curve give a consistent, slightly mechanical snap; nothing bounces.
- Slow ambient loops (24-48 s) keep the interface alive without drawing the eye.
- Delaying the loading indicator by 160 ms and holding results for 2.2 s makes status feel calm rather than flashing.
- Flip-board digits and rolling numbers give data a physical, instrument-like quality.
- Silence by default makes the few sounds feel meaningful.

## Kawn improvements
1. Keep the 150/300 ms pair and a single ease, but make the sheet open spring-free and interruptible (reversing mid-motion should retract smoothly).
2. Press feedback on touch: 96 percent scale or a tint within 80 ms, since hover does not exist on phones.
3. Replace pure-opacity panel snaps in the right column with the same 300 ms slide as the left, for consistency.
4. Reduced-motion: honour the OS setting everywhere and add an in-app motion switch; replace camera flights with a 150 ms crossfade; stop all ambient loops; keep status changes instant rather than flipping.
5. Camera flights: cap at 2.5 s, always cancellable by touch, with a visible "cancel" only if longer than 1.5 s.
6. Arabic/RTL: sheets and trays slide from the start edge; number-roll direction and slider fill direction mirror; flip-board animation is disabled for Arabic text (disconnected shapes).
7. Light theme: do not rely on glow pulses for liveness; use a small solid dot that fades.
8. Accessibility: announce loading/finished states politely to screen readers; never use motion alone to convey status; all sounds optional with a caption.
9. Calmer civic tone: drop sonar sweep and scan lines by default; keep only loading and live indicators.
10. Sound: provide one short soft confirmation for completed saves/reports if wanted, off by default, and keep the "never play while the mic is open" rule.

## UNVERIFIED
- Real hover and press timings in the running app (read from styles only).
- Duration of reset-globe, north-up and tilt moves.
- Toast fade timings (no dedicated toast styles found).
- Whether panel drag or resize shows live motion effects.
- Behaviour of the tracking flight camera (no live contact available).
- Any sound produced by the map itself or other layers (searched only the voice and radio code).

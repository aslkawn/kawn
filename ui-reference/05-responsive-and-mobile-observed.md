# 05 - Responsive and mobile behaviour of the original app (observed)

## Purpose
Record how the original app's interface behaves at six viewport sizes so Kawn can reproduce what works and avoid what does not. Evidence: live captures taken in the user's local keyless copy at 1920x1080, 1440x900, 1024x768, 768x1024, 390x844 and 360x740 (initial/first-run, after dismissal, Data Layers open, Display open), element measurements read from the live page, and stylesheet/test review for breakpoints and safe-area use. The capture files live in the wallA-notes live-shots folder and are referenced by name below, not copied. A tracked-flight capture was skipped: keyless mode provides no live contacts within the time budget.

Method note: all measurements are CSS pixels, taken with a desktop (mouse) browser emulation at the stated size, device pixel ratio 1. No touch emulation, so touch behaviour is inferred, not observed.

## 1. Breakpoints found in the styles
| Width/height rule | What it governs (observed or inferred) |
|---|---|
| max-width 980 | dock label layout starts tightening |
| max-width 960 or max-height 700 | compact dock/cockpit variants |
| max-width 900 | dock trays and sliding dock rearrange (compact) |
| max-width 760 | cockpit view switcher moves up, width capped to viewport minus margin |
| max-width 720 | main phone layout: title shrinks, top actions pin top-right, style indicator hidden, panels span the width, layers panel resized |
| max-width 620 | controls and dock trays go phone-sized |
| max-width 575 / 520 | further small-phone trimming (second phone rule block) |
| max-height 640 / 620 | short-screen trims |
| min-width 961 with height 650+ / under 650; min-width 761 with height 820 or less | desktop height variants |
| prefers-reduced-motion | 12 blocks, see doc 06 |
Notable absences: no pointer-coarse or hover-none query anywhere; no viewport-fit setting in the page head (only width=device-width, initial-scale=1), so notch/home-indicator insets are almost never non-zero; the safe-area inset is read in only three places (top-right action cluster on phone, cockpit view switcher bottom offset twice). The dynamic viewport unit appears only in the first-run card and provider settings dialog max heights.

## 2. Per-width layout (what I saw)
| Width | Shell | Left column | Right column | Bottom dock | Notes |
|---|---|---|---|---|---|
| 1920x1080 | Circular viewfinder vignette in the globe centre; large letter-spaced title top-left with eye logo and tagline; five round action buttons centred at top (y 28-70); style readout top-right | Intel-style header text (classification line, product id, mode word, summary) at x 59, y 160-260; collapsed Data Layers and Scenes cards at x 52-228, y 285-392 | Display, CCTV, Street Level, Context cards stacked at x 1692-1868, y 285-508 | Location, mic, Visual Presets cluster centred, x 722-1198, y 996-1056 | All HUD corner readouts visible (REC time, orbit/pass, MGRS, GSD, altitude, sun elevation, vertical band/bits/level texts) |
| 1440x900 | Same composition, scaled: panels pinned at 52 px left and about 52 px from the right | Layers open = 280 x 451 | Display toggle at x 1212 | same | Panel max height tracks the viewport (451 of 900) |
| 1024x768 | Title is partly overdrawn by the five action buttons (title reads "GOD'S EYE VIE" and tagline is clipped at "NO PLACE LEFT BEHI") ; HUD readouts begin to collide with the right column (REC line overlaps top readout) | Layers open = 280 x 325, list scrolls inside, panel stops above the intel corner text | Four right cards at x 797-971, 174 wide | Dock x 291-733, labels fully legible | Layers panel shows only 4 of 9 movement rows without scrolling |
| 768x1024 | Title and action buttons overlap more strongly (title cut after "EY"); style readout still top-right | Layers open = 280 x 562 | Display open = 272 wide at x 540 beside Layers (x 52-332): both panels open at once without overlapping, centre of map is left clear (about 200 px) | Dock cluster is cramped: mic card overlaps the Location and Visual Presets labels | Both panels fit but cover about 80 percent of the width |
| 390x844 | Logo only (title text removed), five round buttons right-aligned at top (36 px each, 44 px pitch, x 162-374); style indicator hidden; corner HUD text mostly hidden but a stray cyan mode word and summary line remain mid-left and a translucent cyan readout string overprints the dock at the bottom | Data Layers and Scenes become full-width bars (x 16-374, 56 px tall, y 70-130/130-190) stacked top; open Layers = 358 x 286 anchored under the bars, internal scroll | Display, CCTV, Street Level, Context become full-width stacked bars lower down (y ~430-650); the Display bar is right-aligned and shorter | Dock is 380 wide but the mic card sits on top of the two side buttons: labels read "L" and "ESETS" (clipped) | Display panel opens to 343 x 674 starting at y 430, so it runs to y 1104 against an 844 screen; its lower 260 px (more toggles) are off-screen; its internal scroll appears only within the visible part and the page behind does not scroll |
| 360x740 | Same as 390 with 30 px less width; top buttons start at x 132 | same stacking, bars 328 wide | Display bar x 168-344 | Same overlap; dock text clipped | Opening Display did not visibly expand it in my capture (the cluster of four bars just reflowed up by about 50 px), so its open state at 360 is UNVERIFIED |

## 3. First-run card by width
Card is a centred dialog headed "Choose your first view" with four choices: LIVE CONTACTS, SPACE MISSIONS, ENVIRONMENTAL, EXPLORE MANUALLY, a "Don't show this again" checkbox and a tip line.
| Width | Card box (x, y, w, h) | Notes |
|---|---|---|
| 1920x1080 | 689, 313, 542 x 454 | Row buttons 495 x 56 each; "ESC TO DISMISS" hint shown on the right; map visible all around |
| 390x844 | 16, 345, 358 x 405 | Rows 316 wide, 48 high (the Environmental row is 56 because its subtitle wraps to two lines); sits in the lower 55 percent of the screen so the top action buttons and the layers bars stay visible and tappable behind it; the "ESC TO DISMISS" hint is removed; subtitle text 10-11 px |
| 360x740 | similar, 16 px margins | not separately read; first card rows at 360 were enumerated at 48 px high |
Dismissal by keyboard (Escape) works and the card does not return on reload in the same browser profile; no touch-friendly close button was visible on the phone card (UNVERIFIED whether the choices themselves are the only exit).

## 4. Touch targets measured (phone, 390 wide)
| Control | Size px | vs 44 px minimum |
|---|---|---|
| Five top round buttons (clear layers, share, tilt, north, reset globe) | 36 x 36, 8 px gap | fails by 8 px |
| Panel expand/collapse square buttons | 22 x 22 (right-column ones 20 x 20) | fails badly (half) |
| Dock "Location" and "Visual Presets" toggles (the visible button element) | 28 x 22 | fails; the wider card around them may also be tappable (UNVERIFIED) |
| Mic button | 42 x 66 (the voice card is about 140 x 90) | passes |
| Voice tier chip ("STD") | 25 x 12 | fails badly |
| Context radio toggle | 28 x 28 | fails |
| Layer on/off switches | 43 x 20 | fails on height |
| First-run choice rows | 316 x 48 | passes |
| First-run checkbox | 13 x 13 | fails badly |
| Attribution links | 10 px text, 13 px tall | fails |
Counts at every width (24 interactive elements visible with the first-run card up): 20 of 24 are under 44 px in at least one dimension; the smallest dimension found is 12 px. Sizes do not change between desktop and phone for the small controls: the phone gets no larger hit areas.

## 5. Text sizes seen on phone
| Element | Size |
|---|---|
| Panel title labels (DATA LAYERS, SCENES, DISPLAY) | about 9-10 px, wide letter-spacing, low contrast against dark glass |
| Layer row names | about 14-15 px |
| Layer row subtitle (provider, "never") | about 9 px |
| Layer switch label (OFF) | 9 px |
| Display slider labels (DENSITY, FADE, OUTSIDE) | 8 px |
| Voice status text | 6.4 px (tier chip), 8-10 px for the rest |
| Attribution | 10 px |
| First-run title | about 22 px; row title about 12 px letter-spaced caps; row subtitle about 10 px |
Most utility text is 8-10 px, below any usable reading minimum on a phone held at normal distance.

## 6. Safe areas and orientation
- Only the top-right action cluster and the cockpit switcher honour the safe-area inset; the left logo, the stacked bars, the dock and the first-run card do not. With no viewport-fit setting the insets evaluate to zero in a browser anyway, so a phone in full-screen/installed mode would put bars under the notch and the dock under the home indicator (inferred).
- Landscape phone (about 740 x 360) was not captured; two max-height rules (620/640) exist for it. UNVERIFIED how it looks.
- Tablet portrait 768x1024 keeps the desktop two-column scheme with narrower panels.
- No orientation-specific query was found.

## 7. Performance feel
- Initial load: globe imagery appears within about 3 s in the local copy; the first-run card is up before the sharpest imagery resolves.
- At every size the page does not scroll horizontally (document width equals viewport width).
- Render is a continuous full-screen 3D view with a heavy vignette and scan-style effects; on phone the same effects stay on (no reduced render mode observed), which will cost battery. Console showed 1-2 errors and 1 warning on load in keyless mode (unspecified provider failures, expected without keys).
- Panel open/close happens in 0.3 s; no jank was visible in stills.

## 8. What breaks or is unusable on phone (observed)
1. Tap targets: 22 px panel toggles and 36 px action buttons.
2. Display panel taller than the screen with no way to see its bottom (starts at about half height and overflows by 260 px).
3. Dock: mic card hides the neighbouring controls' labels ("L", "ESETS"); the controls are readable only by guessing.
4. Eight to ten px text for essential state (layer status, slider values).
5. Stray HUD text (mode word, summary line, cyan readout) printed over the map and the dock with no background, unreadable on bright imagery.
6. Panels stacked as eight bars take 55 percent of the height even when everything is collapsed, leaving the map mostly covered by chrome.
7. Opening a panel does not close the others, so two or three can stack and push content off screen.
8. Title overlaps the action buttons from 1024 down to 768.
9. Attribution links are under the dock and about 13 px tall.
10. No pinch/gesture documentation or touch-specific hints (the tip line mentions a mic button only).

## What makes it feel the way it does
Everything is built as floating glass cards on a full-bleed globe; on desktop this reads as a cockpit. On phone the same cards are simply stretched to full width and stacked, so the cockpit becomes a column of title bars, and the density that is charming on a 27-inch screen is the main thing that fails.

## Kawn improvements
1. Mobile first layout: map full-bleed; one bottom sheet with three snap heights (peek 72 px, half, full), one panel at a time; a bottom tab bar (Layers, Display, Search, Location) instead of eight bars.
2. Minimum 44 x 44 px targets (48 recommended) with at least 8 px gaps; checkbox and switch hit areas expand beyond visible size.
3. Text floor of 12 px for anything essential, 14-16 px for row names; value readouts 12 px minimum; avoid wide-tracked caps below 11 px.
4. Honour all four safe-area insets, set viewport-fit cover, use dynamic viewport units for sheet heights; cap every panel to visible height with internal scrolling and a visible scrollbar/shadow cue.
5. Desktop-only effects (vignette, scan text, corner readouts) removed or off by default on phone; offer a "calm" mode default for civic use.
6. Dock must never overlap its siblings: at narrow width, drop labels (icons only) or move to the tab bar.
7. Landscape phone: side rail instead of bottom bar.
8. Arabic/RTL: mirror the whole layout (action cluster to the left, sheet handle centre, panel toggles on the start edge); letter-spaced uppercase styling must be dropped for Arabic since tracking breaks joined letters; icons that imply direction (arrows, chevrons) flip.
9. Light theme: the translucent dark glass needs a solid-enough light surface (about 90 percent opaque) so text over bright imagery keeps 4.5:1 contrast.
10. Accessibility: visible focus ring on every control, role/label for icon-only buttons, escape/close affordance on every sheet that is also a visible button.
11. Calmer tone: first-run copy and chrome should avoid secret-agency styling for a public-service product.

## Prioritised mobile defects Kawn must avoid
| Priority | Defect |
|---|---|
| P0 | Tap targets 20-36 px; essential controls (panel toggle, layer switch) hard to hit |
| P0 | A panel taller than the screen with its end unreachable |
| P0 | Overlapping/clipped dock labels |
| P1 | 8-10 px essential text; stray overlay text on the map |
| P1 | Stacked bars consuming over half the screen; several panels open at once |
| P1 | No safe-area handling for logo, dock, sheets |
| P2 | Heavy effects always on; battery/performance |
| P2 | Title colliding with action cluster on tablet portrait |
| P2 | No coarse-pointer or hover-less styling (hover-only affordances) |
| P3 | No landscape-phone layout |

## UNVERIFIED
- True touch behaviour (gestures, long press, scroll chaining) - no touch emulation.
- Landscape phone appearance.
- Display panel open state at 360 wide.
- Whether the full dock cards (not only the 28 x 22 labels) are tappable.
- Tracked-flight layout at any width (needs live data).
- Behaviour with a real notch or in installed/standalone mode.
- Exact first-run card size at 360.

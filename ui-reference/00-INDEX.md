# UI reference index (how the original looks and feels; clean-room)

Method: screenshots (36 stills of the original) viewed one by one, a live run of the owner's local copy at six viewport sizes (24 captures, kept outside kawn/),
and stylesheet numbers read for measurements only. Values (hex, px, ms) are recorded as design facts; no code, selectors or copied text. Screenshots are NOT stored here.

| # | File | Covers |
|---|---|---|
| 01 | `01-visual-language.md` | Colour by role, type scale, icons, spacing, radii, glow/blur, cosmetic overlays, brand |
| 02 | `02-looks-hud-and-modes.md` | 7 looks, 4 HUD layouts with positions, scope, detection look, cockpit, recording, Clean UI |
| 03 | `03-screen-anatomy-and-layout.md` | 34 regions, z-order, state table, 9 ASCII wireframes |
| 04 | `04-components-and-states.md` | Component catalogue with state matrix |
| 05 | `05-responsive-and-mobile-observed.md` | Per-breakpoint behaviour, measured touch targets, mobile defects Kawn must avoid |
| 06 | `06-motion-and-interaction-feel.md` | Transition timings/easings, feedback, sound, reduced motion |

## Headline findings
- Look: near-black blue canvas, translucent glass panels (blur ~24 px), one electric-cyan accent, monospaced uppercase labels; military/sci-fi tone.
- Layout: top-centre action row, left layer rail, right display/context rail, bottom dock with trays; HUD corners overlay the globe.
- Mobile is not designed: no phone layout beyond stacking into scroll bands; 20 of 24 controls under 44 px; essential text 8-10 px; one panel overflows a phone screen by ~260 px.
- Motion: one 150/300 ms pair, one ease curve; almost silent.

## Known gaps (UNVERIFIED in files)
No Minimal-HUD or static cockpit shot; shots 17-36 are at ~0.9 UI scale; no real touch or landscape-phone test; opened street-level viewer, first-run card, settings dialog, voice card and timeline never appear in a still; animated looks cannot be judged from stills; per-layer marker hex colours estimated.

## Next
Kawn's own UI design (personas, mobile-first, Arabic/RTL, light/dark, new layers) goes in `../ui-design/` and must not copy this look 1:1.

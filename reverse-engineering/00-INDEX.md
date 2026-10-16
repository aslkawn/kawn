# Reverse-engineering index (behaviour-level, clean-room)

Subject: the open-source browser 3D-globe live-data app studied for Kawn. Method: static reading of source, docs and tests by
parallel analyst agents (nothing run, nothing installed from the app), plus the earlier live walkthrough notes. No code,
file names, identifiers or copied wording. Public provider names and numeric behaviour values are included on purpose.

| # | File | Covers |
|---|---|---|
| 01 | `01-ui-controls-and-panels.md` | Every panel, rail, dock, first-run, settings, responsive, embed, a11y |
| 02 | `02-keyboard-shortcuts-and-shareable-state.md` | Shortcuts, gestures, share-link state, persistence |
| 03 | `03-layers-movement-and-events.md` | Flights, military, ADS-B, GNSS, vessels, satellites, launches, transit, bikes, traffic, quakes, fires |
| 04 | `04-layers-infrastructure-weather-cameras.md` | Cables, data centres, dams, installations, ALPR, CCTV, imagery, street level, radio, directions, weather, flood scene |
| 05 | `05-visual-looks-and-rendering.md` | Basemaps, terrain, 7 looks, bloom/sharpen, 3D models, performance, design language |
| 06 | `06-camera-tracking-cockpit-contacts.md` | Camera model, tracking, cockpit, contacts roster |
| 07 | `07-hud-detection-scope-annotations.md` | HUD layouts, detection, scope, drawing, recording |
| 08 | `08-voice-ai-and-tools.md` | Voice lifecycle, ~30 actions, 25 query tools, analyst answers, cost, MCP |
| 09 | `09-director-scenes-sharing.md` | Scene system, import/export, bundled scenes |
| 10 | `10-backend-api-and-data-sources.md` | Server duties, ~40 upstream sources + licence notes, caching, SSRF controls |
| 11 | `11-config-keys-costs-and-privacy.md` | Keys, costs, outbound destinations, telemetry, install modes, tests |
| 13 | `13-feature-catalog-master.md` + `13a-13f` census files | 662-feature master list; per-domain censuses with technique-level explanations, gap write-ups and corrections |
| 12 | `12-external-urls.md` | Every external host and sample endpoint the app uses, by category, with Kawn UAE notes |

Older material: `../specs/01-04` (summary specs), `../docs/05-06` (inventory, design), `/Users/imdadareeph/Desktop/42/gev/FEATURES.md` (live walkthrough).

## Tool notes
- REA (analysis tool) was run on the app source: 1,442 files parsed, 0 failures. It targets Electron apps, so output was mainly an
  import graph; its "endpoint" nodes were test fixtures. Raw output (227 MB) lives outside this folder in `wallA-notes/` and is NOT part of the clean-room set.
- GitNexus index exists for navigation; not needed for these docs.

## Status
Pass 1 (draft) and pass 2 (revision) done 2026-10-16. Every doc 01-11 was re-verified claim by claim against source and tests by an independent
reviewer, carries confidence tags and a "Revision log" section, and lists what stays UNVERIFIED. The REA skill method (checklist, evidence tiers,
finding ledger, final recheck) was followed. Private ledgers (with real paths) live outside this folder in `wallA-notes/ledgers/`.

## Headline corrections found in pass 2
- Display panel starts collapsed; Sharpen defaults on at 49%; page shortcuts ignore Ctrl/Cmd; C key has three meanings; Escape runs in three tiers.
- No transit filter UI, no earthquake click card, no satellite-pass UI (voice/tool only); launches = past 30 days only; OpenSky default is OAuth.
- Cables: click only flies; UAE has 16 data centres and 0 dams; CCTV auto-hop off by default.
- Voice: 30 actions + 25 query tools; MCP 30 tools (29 model-visible); scene format 6 is current.
- Detection labels are single-line; HUD text sits below detection; cargo-ship model unused; Clean UI Escape does not exit.
- Backend: not every proxy is cross-site gated; bike-share relay takes an allow-listed browser URL; ~562 test files; no telemetry found.
- Abu Dhabi is labelled "AFRICA" by the region label: Kawn must own its region logic.

## Biggest remaining gaps
Mobile/touch on real devices; live timings and pixel layout; engine-default gestures; UAE coverage of Mapillary/KartaView/radio;
no network capture (telemetry "none" is from source reading); OpenSky quota exhaustion behaviour.

## Next
Specs 01-04 and docs/05 are being corrected from these docs. Then hand the builder session its reading list. Implementation waits for the owner's go.

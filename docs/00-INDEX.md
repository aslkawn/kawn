# Kawn docs index

Reading order for a human (you). The Wall B builder session reads only `../specs/*`, `../brand.json`, `../KICKOFF.md`
(plus `docs/05` and `docs/06` if you choose to hand them over; they are behaviour/design only, no code).

## What already existed in this folder (audited 2026-10-16)
| Path | What | Status |
|---|---|---|
| `brand.json` | Single source of truth for name (Kawn / كون / "See the whole city.") | done, domain empty |
| `KICKOFF.md` | Instructions for the clean-room builder session | done |
| `specs/01-product-ux.md` | Users, screens, modes, first-run, interactions, a11y, states | done (142 lines) |
| `specs/02-layers.md` | Every data layer behaviour + UAE relevance ranking | done (208 lines) |
| `specs/03-backend-data.md` | Data sources, proxy, keys, caching | done, thin on weather + server |
| `specs/04-voice-maps-camera.md` | Voice, basemaps, camera, director | done, thin on director/annotations/tools |

Missing before this pass: design system / theme, one-page feature list, REA tooling notes, roadmap. Added below.

## Added in this pass
| File | Purpose |
|---|---|
| `05-feature-inventory.md` | Every God's Eye View feature, grouped, with Kawn keep/skip/adapt verdict |
| `06-design-system.md` | Observed look & feel of the original + proposed Kawn palette/type/motion |
| `07-rea-and-gitnexus.md` | What REA and GitNexus are, how to use them safely for Wall A |
| `08-roadmap.md` | MVP -> v1 plan and open decisions |

## Clean-room reminder
No source code, file names, identifiers or copied wording from the original. Design facts (colour roles, type,
radii) are recorded as observations; Kawn should pick its own final values (see 06 section 3).

## Update 2026-10-16 (after reverse-engineering revision)
- `../reverse-engineering/` (00-INDEX + 01-11) is the detailed, verified behaviour reference; specs 01-04 and docs 05-06 were corrected from it.
- Builder reading list suggestion: KICKOFF.md, brand.json, specs/01-04, docs/05, docs/08, docs/06 sections 2-4. Reverse-engineering docs are optional deep reference (owner decides).
- Keep OUT of the builder session: docs/07 (names the original-analysis tools) and docs/06 section 1 ("Observed" values of the original).

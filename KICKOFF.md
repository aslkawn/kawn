# Wall B kickoff (paste into a FRESH Claude session in a NEW empty folder)

Rules: read ONLY these in kawn/: KICKOFF.md, brand.json, specs/01-04, docs/05-feature-inventory.md, docs/06-design-system.md (sections 2-4 only), docs/08-roadmap.md, and reverse-engineering/ (behaviour reference; do not read reverse-engineering/12 until you need a data-source list). Never open, clone, or search the original
God's Eye View repo or the REA notes. Build an original product from behavior specs,
using a different stack (e.g. MapLibre GL / deck.gl, not Cesium).

## Naming (configurable in ONE place)
The product name lives only in `brand.json` (copy it into the new project root).
- Never hardcode the name, slug, tagline or domain anywhere else.
- Code: load it once (a single brand module/constant that imports brand.json); all UI text, page titles,
  meta tags, manifest, package name, env-var prefix, log prefixes, i18n strings (Arabic + English)
  and docs use it via that module/placeholder, e.g. {{brand.name}}.
- Build time: package name, HTML title, manifest and README header are generated from brand.json by one small script.
- Arabic name/transliteration gets its own field in brand.json (add "name_ar") if it differs.
- Renaming = edit brand.json, run the build. Add a check that fails if the old name string appears in src.
- Arabic note: "Kawn" (كون) means universe; keep the Arabic form in brand.json too.

## Product
Live city intelligence for Dubai, Abu Dhabi, Sharjah.
Arabic + English, RTL, mobile-first. Personas: investor, tourist, government planner.
MVP: map + 3 layers (flights around DXB/DWC/AUH, metro/bus, real estate), city switcher.
Skip: CCTV, plate readers, military, fire perimeters. Keys stay server-side.
Verify every data source licence before use. Plan first; ask before installing anything.

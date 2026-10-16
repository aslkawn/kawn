# Owner decisions log (binding for design and build)

Newest first. Source: product owner, in chat. Builder must follow these over any spec text that says otherwise.

## 2026-10-16: default view and locations
1. **Default view = Scope view, feather 100%.** On first load the app opens with Scope ON (spotlight/keyhole around the view centre; detection labels sharp inside, fading smoothly toward the edges and into a soft vignette) and feather at its maximum (100% = smoothest fade, no hard edge). Reference: the owner's screenshot of the original in Scope view with the feather slider at 100% (London, about 28 km altitude, top-down, normal look, dense detection at 75%).
2. **Default location = Dubai**, not a US city. First camera position: Dubai, top-down, city scale. (Original defaulted to Austin, USA.)
3. **Location tray buttons** (replace the original's US/world chips):
   - **All 7 emirates:** Abu Dhabi, Dubai, Sharjah, Ajman, Umm Al Quwain, Ras Al Khaimah, Fujairah.
   - **Important airports, Dubai:** DXB (Dubai International), DWC (Al Maktoum International). Also recommended for the UAE set: AUH (Abu Dhabi), SHJ (Sharjah).
   - **Important airports, London:** LHR (Heathrow), LGW (Gatwick), STN (Stansted), LCY (London City), LTN (Luton). London is a secondary, non-UAE group.
   - Each button flies the camera there; search still works for any place (bilingual Arabic/English).

## Approximate fly-to targets (verify before use; city-scale framing to be tuned)
| Button | Lat | Lon |
|---|---|---|
| Abu Dhabi | 24.454 | 54.377 |
| Dubai | 25.205 | 55.271 |
| Sharjah | 25.346 | 55.421 |
| Ajman | 25.405 | 55.514 |
| Umm Al Quwain | 25.565 | 55.555 |
| Ras Al Khaimah | 25.790 | 55.943 |
| Fujairah | 25.129 | 56.327 |
| DXB | 25.253 | 55.364 |
| DWC | 24.896 | 55.161 |
| AUH | 24.433 | 54.651 |
| SHJ | 25.329 | 55.517 |
| LHR | 51.470 | -0.454 |
| LGW | 51.154 | -0.182 |
| STN | 51.886 | 0.239 |
| LCY | 51.505 | 0.055 |
| LTN | 51.875 | -0.368 |

## Design consequences to resolve in the UI design pass
- Scope as default means the centre of the map is the hero: put the most important live data (flights, metro, key POIs, real-estate hotspots) in the centre zone; edges stay calm. Define what "centre" means when side panels are open and on phones (use the free map area, not the full window).
- Keep a visible, one-tap way to turn Scope off and to change feather (Scope is the default, not forced).
- Scope reuses the original's "keyhole + feather" behaviour (doc: reverse-engineering/07); the visual treatment must be Kawn's own.
- Location tray needs grouping and Arabic labels: Emirates / Dubai airports / London airports; RTL order; large touch targets.
- Dubai airports and London airports are chosen because flights are the flagship live layer (DXB/DWC arrival and departure traffic; LHR as the busiest comparison and the original's own showcase).

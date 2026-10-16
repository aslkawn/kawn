# Design system

## 1. Observed in the original (reference only)
Self-described mood: "dark sci-fi, Apple meets Blade Runner".
| Role | Observed |
|---|---|
| Page background | near-black blue `#0a0a0f` |
| Panel surface | translucent dark glass, ~72% opacity of `#0c0c14`, backdrop blur ~24px + saturate 1.4 |
| Opaque menus | `#12121c` |
| Panel border | 1px white at 8% (15% on hover) |
| Primary accent | electric cyan `#00d4ff` (dim 15% fill, 40% glow); world-card accent `#6be8ff`; HUD colour follows the look (cyan, green NVG, white FLIR, amber CRT) |
| Warm secondary | military tier amber `#ffb347`; slow-traffic amber `#f0b23e` |
| Alert | traffic-jam red `#e05252`; recording dot `#ff3333` |
| Text | primary `#e8eaed`, secondary 50%, dim 30% |
| Type | JetBrains Mono for nearly all UI/labels (uppercase, tracked), Inter for prose; Material Symbols icons |
| Shape | panel radius 16px, button 10px, pills 999px, many HUD elements 0-radius with thin lines |
| Motion | 150ms / 300ms, cubic-bezier(0.4,0,0.2,1); `prefers-reduced-motion` honoured |
| Focus | 2px light outline everywhere, never removed |
| Layout | left layer rail, right rail (Display, Context, etc.), bottom command dock, top-centre action row; cockpit mode re-lays HUD corners |
| Breakpoints | 980, 900, 760, 720, 620, 575, 520 px; at 720 and below both rails become edge-to-edge scroll bands (no bottom sheets) |
| Sensor skins | seven full-screen looks: Normal, CRT, NVG (green), FLIR/Ironbow, Anime, Noir, Snow |
| Layer chips | one colour per layer, status dot, text status line |

Weakness seen: tiny mono caps text, weak mobile, no RTL, military tone.

## 2. Kawn direction
Tone: calm, premium, civic-tech. "Control room for a city" rather than "weapons HUD".

## 3. Proposed Kawn tokens (starting point, tune in browser)
| Token | Dark (default) | Light |
|---|---|---|
| bg | `#0b0f14` | `#f6f4ef` |
| surface (glass) | rgba(16,22,30,.72) + blur 20 | rgba(255,255,255,.78) |
| border | white 10% | black 8% |
| text | `#eaeef2` / 55% / 35% | `#14181d` / 60% / 40% |
| accent (Kawn teal) | `#2ee6c5` | `#0a9f87` |
| accent-2 (desert gold) | `#e0b15a` | `#a97a1f` |
| warn / danger / ok | `#ffb454` / `#ff6b6b` / `#5be39b` | darker equivalents |
Layer colour set (colour-blind safe, distinct): flights sky `#58b6ff`, transit teal `#2ee6c5`, real estate gold `#e0b15a`,
traffic coral `#ff7a59`, vessels violet `#a58bff`, weather ice `#9fe3ff`.
Type: **IBM Plex Sans Arabic** (AR+Latin, one family, good dense data) for UI; **IBM Plex Mono** only for numerals/IDs.
Min body 14px, labels 12px, no all-caps tracking on Arabic.
Shape: radius 14 panels, 10 buttons, 999 chips. Motion 150/300ms, reduced-motion respected.
Icons: Lucide or Phosphor (MIT). Numerals: toggle Western/Eastern Arabic.
Layout: mobile-first bottom sheet + layer chips row; desktop left rail + right context + bottom timeline; true RTL mirroring.
Themes: Dark (default), Light, High-contrast; Looks (map skins): Day, Night, Heat, Blueprint, Noir.
Accessibility: WCAG AA contrast, visible focus ring, keyboard map controls, no colour-only meaning.

## 4. Config
All tokens as CSS variables in one file generated/overridable; name and tagline from `brand.json`.

# Texture System

## Texture Permission (single source of truth)

Texture permission is decided by `direction.layout` — **never** by a palette or a recovery module. Every module that wants to emit texture language must resolve it here first.

Three tiers, most permissive to least:

| Tier | What it covers | Allowed where |
|------|----------------|---------------|
| **PRINT** | Print defects as visible pattern: risograph grain, xerox halftone, halftone degradation, scan noise, film grain, ink bleed, misregistration, aged paper mottling | `zine` **only** |
| **SURFACE** | Material character of the substrate itself: cotton paper, natural fibre, matte board, lived-in warmth. Described as what the ground *is*, never as an overlay pattern or visible dot/grain structure | CLEAN layouts whose style DNA rates Texture (or an equivalent material dimension) ★★★ or higher: Apartamento ★★★★★, COS ★★★★★ *Material calm*, Kinfolk ★★★★, MUJI ★★★, Brutalist ★★★ |
| **FLAT** | Zero texture language of any kind | `photo-abstract-diptych` abstract panel ground, `interface-asset`, `website-hero` copy-safe area, `product-editorial` background |

Everything not listed under PRINT is **CLEAN**: `photo-abstract-diptych`, `poster`, `magazine-cover`, `gallery-print`, `editorial-spread`, `campaign-poster`, `brand-key-visual`, `product-editorial`, `website-hero`, `social-asset`, `interface-asset`, `moodboard`.

**CLEAN is the default.** On a CLEAN layout a PRINT clause is dropped, not downgraded — the requesting module falls back to contrast, spacing, mark scale, and hierarchy.

Contract consequences:

- CLEAN → `design_tokens.texture` holds SURFACE tokens only (`cotton_paper`, `natural_material`, `matte_board`) or is `[]`
- CLEAN → `recoveries` must not contain `riso_texture`
- CLEAN → no adapter may emit "riso", "halftone", "grain", "noise", "xerox", "scan", or "mottling"
- FLAT → `design_tokens.texture` is `[]` and no texture words appear at all
- Styles with no Texture / material dimension (Swiss ★★, Monocle, Purple, POPEYE, Wallpaper\*) get no SURFACE tokens — flat matte grounds

## Editorial (photo-abstract) — FLAT panel

- Flat ivory panel — **no** gradient, shadow, grain, or paper stain on the panel ground
- Depth comes from contrast, interval, and mark scale

## Zine (gc-minimal) — PRINT

- Aged paper mottling, scan fibers
- Xerox softness, risograph grain, halftone degradation
- Letterpress ink bleed, misregistration
- Flat orthographic scanned-paper view — no 3D mockup

## Panter compensation

Panter is a **colour** compensation. It never promotes a layout to a higher texture tier.

- CLEAN / FLAT layouts: compensate with warm/cool conflict hues, the high-chroma anchor, wider tonal separation, spacing, and mark scale. No texture clause.
- `zine`: riso halftone may additionally be applied to the abstract marks, where texture density supplements chroma complexity.

## Lighting texture

- Diffuse, matte, low-to-medium contrast
- No cinematic rim light, no glossy reflections

# Presentation Deck Layout

For multi-page slide sets where every page must read as one publication. Plan the set with [../prompts/series.md](../prompts/series.md); this file supplies the per-page DNA.

Default aspect ratio 16:9. Use 4:3 only when the user names a legacy projector.

## Page Roles

| Role | Focal | Type | Whitespace | Notes |
|------|-------|------|------------|-------|
| `title` | 55% | 25% | 20% | The only page with display-scale type |
| `section` | 35% | 20% | 45% | Divider. Near-textless, carries the palette |
| `content` | 45% | 25% | 30% | Repeats N times — rotate composition family |
| `data` | 60% | 15% | 25% | Chart area stays flat and legible; no texture |
| `closing` | 40% | 20% | 40% | Mirrors `title` composition, inverted |

## Structure

- One visual system across the deck: locked palette, typeface, texture tier ([../prompts/visual-memory.md](../prompts/visual-memory.md))
- Rotate composition family across `content` pages — left-anchor, right-anchor, centered, full-bleed — no family more than twice in a row
- Keep a consistent margin grid on every page; the deck reads as a template the moment margins drift
- Copy-safe area on all four edges for projector overscan
- The color anchor appears on every page at 0.5–2% canvas, same hue

## Texture

One tier for the whole deck — a tier is a property of the system, not of a page. **FLAT** is the default, and is mandatory whenever the deck contains a `data` page. A deck with no `data` page may run SURFACE on every page when the style DNA rates Texture ★★★+. PRINT is never valid in a deck. See [../assets/texture.md](../assets/texture.md).

## Compiler anchor

```
Presentation page, 16:9, single consistent editorial system, generous margin
grid, copy-safe edges, one focal visual, restrained type hierarchy, flat
legible ground, one small saturated color anchor, no chart junk, no mockup
frame, no watermark, no fake UI.
```

## Blocked

Zine variation recipes, riso or scan defects, magazine cover furniture (barcode, cover lines), heavy full-bleed photography behind body copy.

# Vintage Travel Poster — Preset

A place reduced to an ink poster: saturated flat colour fields, hard-edged stacked shapes, and the destination name set large as part of the composition. Mid-century travel and exhibition poster lineage.

The ground here is **ink, not paper** — that is the whole point, and the reason this preset exists alongside [ivory-postcard](ivory-postcard.md).

## Locked DNA

```yaml
memory_version: "1.0"
memory_id: vintage-travel-poster
version: "1.0"
established_from: preset
source: preset

locked:
  ground: saturated
  render_mode: graphic
  style: travel-poster
  visual_language: Poster Graphic
  intended_layouts: [poster, campaign-poster, social-asset, magazine-cover]
  abstraction_level: full-abstract
  palette: [deep teal, burnt orange, ochre, cream, ink navy]   # 4–6 flat inks
  typography: "condensed geometric sans, wordmark scale, letterspaced; one secondary line max"
  texture_tier: SURFACE                 # matte poster stock only — no grain, no halftone
  atmosphere: optimistic, graphic, mid-century
  composition:
    horizon: strong                     # or a strong diagonal
    subject_scale: large
    type_ratio: 0.20-0.30               # the name is structural, not a caption
    whitespace_ratio: 0.10

blocked_layouts: [zine, photo-abstract-diptych, interface-asset, product-editorial, website-hero, presentation-deck]
                                        # zine = PRINT; the rest are FLAT-ground layouts that fight SURFACE

free: [layout, aspect_ratio, title, subtitle, recoveries]

hard_constraints:
  - "Every colour is a flat area — no gradient modelling, no airbrush"
  - "Four to six inks. A seventh colour is a rejection, not a warning"
  - "The destination or event name is part of the composition, never captioned on top"
```

## Photo Policy

```yaml
photo_policy:
  fidelity: none
  reference_image: uploaded | none
  source_region: full-bleed
```

A source photo, if given, supplies the landmark silhouette and the horizon line. Nothing else survives.

## Avoids

Photographic surface detail · depth-of-field blur · airbrush or gradient modelling · riso grain, halftone dots, scan defects · drop shadows · more than six colours · distressed or "aged" overlay filters · invented logos · watermark

Note on ageing: the period feel comes from the ink set, the flat areas, and the type — **not** from a grunge texture laid over the top. PRINT-tier defects are `zine` only ([../assets/texture.md](../assets/texture.md)).

## Compiler anchor

```
Period travel poster, saturated flat ink ground covering the full canvas.
Landscape or landmark reduced to bold stacked flat colour shapes with hard
edges and no gradient modelling; strong horizon or diagonal; long geometric
shadow. Destination name in condensed geometric sans at wordmark scale,
letterspaced, integrated into the composition; one secondary line maximum.
Four to six flat saturated inks — deep teal, burnt orange, ochre, cream, ink
navy. Matte poster stock. Avoid photographic detail, depth-of-field blur,
gradients, halftone or riso grain, drop shadows, distress overlays, watermark.
```

## Notes

Style DNA: [../styles/travel-poster.md](../styles/travel-poster.md). `texture_tier: SURFACE` is legal because that style rates Texture ★★★ and the token describes matte stock — substrate character, not an overlay pattern.

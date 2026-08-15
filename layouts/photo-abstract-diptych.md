# Photo-Abstract Diptych Layout

Two zones on one vertical canvas: the photograph, untouched, and a flat panel below it carrying an abstraction of the same scene's structure.

The panel is not a caption, a thumbnail, or a decorative border. It is the photograph's spatial logic restated with the fewest marks that still carry it — which is why this layout is the only one where `render_mode: photo-plus-graphic` and `photo_policy.fidelity: required` are both mandatory.

## DNA

| Element | Ratio |
|---------|-------|
| Photograph | 60–68% |
| Abstract panel | 25–33% |
| Title margin | 5–10% |

Adapt to the source aspect: a horizontal photograph takes the lower end and gives the panel room; a tall subject takes the upper end. Never a mechanical half-and-half — the split is a proportion decision, not a default.

## Required Spec Values

```yaml
direction:
  layout: photo-abstract-diptych
  render_mode: photo-plus-graphic
  abstraction_level: relationship-first | identity-cue
design_tokens:
  ground: paper-light
  texture_tier: FLAT
photo_policy:
  fidelity: required
  reference_image: uploaded
  source_region: upper | principal
```

All five are hard requirements, not defaults. `fidelity: required` puts `photo redraw` in `avoids` automatically ([../spec/editorial-spec.schema.md](../spec/editorial-spec.schema.md)).

## The Photograph Zone

Scale and crop only. No redraw, filter, grade, retouch, extension, or outpainting — the photograph arrives as it is and leaves as it is.

Crop only as much as the join needs. If a crop would cost the subject, change the ratio instead.

## What the Panel Is Built From

The panel is compiled from `image_report.spatial` ([../prompts/analyzer.md](../prompts/analyzer.md)) — the same six fields every run already records:

| Spatial field | Becomes |
|---------------|---------|
| `arrangement` | Where the marks sit relative to each other |
| `overlaps` | Which marks occlude which |
| `relative_scale` | Mark sizes against one another |
| `ground_plane` | The panel's implied baseline, or its deliberate absence |
| `light_direction` | Which side the weight falls on |
| `element_kinds` | **The mark vocabulary** — one mark type per kind, no more |

`element_kinds` is the whole answer to "how many kinds of mark". A scene the Analyzer read as `{chair: 3, table: 1, planter: 1}` gets three mark types, repeated as often as the arrangement needs. Nothing is invented that the spatial report did not record, and nothing recorded is dropped because a count felt high.

This is `form_types` from [../assets/scene-construction.md](../assets/scene-construction.md) applied to a panel instead of a full canvas.

## Abstraction Level Decides Recognition

The enum already carries this decision — do not add a separate rule:

| `abstraction_level` | The panel keeps |
|---------------------|-----------------|
| `relationship-first` | Direction, interval, density, hierarchy, occlusion. No object outlines survive |
| `identity-cue` | The above, plus the single silhouette that makes this subject *this* subject — its outer profile only, never what is on its surface |

`full-abstract` is not valid here. A panel with no traceable relationship to the photograph above it makes the diptych two unrelated images.

Surface detail is always dropped: windows, masonry, railings, patterns, texture, small hardware. Recognition comes from profile, scale, and position.

## Panel Ground

Flat `paper-light`, uniform edge to edge. `texture_tier: FLAT` means literally zero — no gradient, vignette, shadow, seam, grain, fibre, stain, or scan artefact ([../assets/texture.md](../assets/texture.md)).

Atmosphere comes from interval, asymmetry, scale contrast, and how much of the panel stays empty. Not from surface noise. A panel that needs texture to feel like something is a panel whose marks are not carrying their weight.

## Palette

Extract from the photograph only, then reduce — see [../assets/palette.md](../assets/palette.md). One dominant role, one structural dark, one light neutral, and at most one accent that genuinely exists in the source.

When the Analyzer flags low saturation or low contrast, the panel switches to Panter compensation ([../recovery/contrast.md](../recovery/contrast.md)) — warm and cool poles pushed apart, one small high-chroma anchor. Panter is colour only; it never buys the panel a texture tier.

## Title

Per [../assets/typography.md](../assets/typography.md): restrained editorial serif, short, on the panel — never over the photograph, never inside the marks.

Draw the title from something the spatial report actually recorded — a relationship, a direction, the light. A title that could sit on any photograph is not doing work.

## Join

Photo and panel meet on a clean straight edge. No torn paper, frame, drop shadow, card, tape, or mockup. The two zones are one canvas, not two objects on a table.

## Compiler anchor

```
Vertical editorial diptych on one canvas. Upper region: the supplied
photograph reproduced exactly — scaled and cropped only, no redraw, filter,
grade, retouch, or extension. Lower region: a flat warm ivory panel, uniform
edge to edge, no gradient or grain. On the panel, restate the photograph's
structure as flat marks — one mark type per element kind the scene contains,
repeated as the arrangement requires; keep their relative positions, sizes,
and overlaps; drop all surface detail. {abstraction_clause}. Palette extracted
from the photograph and reduced: one dominant, one structural dark, one light
neutral, at most one accent that exists in the source. Short restrained serif
title on the panel only. Straight clean join between the two regions. Avoid
photo redraw, filtering, posterisation, vector tracing, thumbnail repetition
of the photo, illustrated faces or limbs, invented objects, decorative
symmetry, panel texture or grain, torn edges, drop shadows, mockup framing,
watermark, extra text.
```

`{abstraction_clause}` resolves from `direction.abstraction_level`:

- `relationship-first` → `keep only direction, interval, density, and hierarchy; no object outlines`
- `identity-cue` → `keep the subject's outer silhouette and nothing on its surface`

## Blocked

Zine variation recipes and PRINT-tier defects · deep perspective recession in the panel · illustrated faces, limbs, or clothing · realistic material, metallic highlight, or volume on panel marks · regularised icon spacing · any second photograph · text anywhere but the panel title

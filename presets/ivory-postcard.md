# Ivory Postcard — Preset

The repo's original hardcoded look, sealed as a preset: a photograph reconstructed as a minimal editorial illustration on ivory paper — three to five simplified symbolic forms, flat opaque paint, and a large quiet field.

This is **not** a photo filter. The workflow reselects the subject, deletes detail, and rebuilds the composition.

Every example image in the READMEs is this preset's output.

## Locked DNA

```yaml
memory_version: "1.0"
memory_id: ivory-postcard
version: "1.0"
established_from: preset
source: preset

locked:
  ground: paper-light                   # warm ivory #F5F0E6
  render_mode: painterly
  style: muji
  visual_language: Museum          # Museum row lists Swiss / MUJI
  intended_layouts: [poster, gallery-print, social-asset, magazine-cover, moodboard]
  abstraction_level: full-abstract
  palette: [warm ivory, charcoal brown, one muted earth tone, one cool neutral]
  typography: "one small fine serif title at the lower margin, or none"
  texture_tier: FLAT
  atmosphere: quiet, still, contemplative
  composition:
    subject_scale: small
    subject_position: upper-centre
    whitespace_ratio: 0.55              # minimum, not a target
    form_count: 3-5                     # simplified symbolic forms

blocked_layouts: [zine]            # PRINT tier would break the flat ivory ground

free: [layout, aspect_ratio, title, subtitle, recoveries]

hard_constraints:
  - "The source photograph is a content reference only — never preserved, never filtered"
  - "At most one small MUTED chroma focus. MUJI DNA rejects saturated accents — this is not a Panter anchor"
```

## Photo Policy

Deliberately the opposite of the `photo-abstract-diptych` lineage. There, `photo_policy.fidelity: required` and the source region is untouched. Here:

```yaml
photo_policy:
  fidelity: none
  reference_image: uploaded
  source_region: full-bleed
```

The photo supplies *facts* — which forms exist, how they relate — and nothing else. Do not preserve its light, perspective, or material.

## Avoids

Photographic realism · translucent overlays · true perspective · overhead wires · dense window grids · fine lattice structures · glossy gradients · neon · magazine-cover furniture (frames, barcodes, cover lines) · invented logos · watermark · any additional text

## Compiler anchor

```
Minimal editorial illustration on a warm ivory paper ground, flat and uniform,
no grain or stain. Recompose the scene from three to five simplified symbolic
forms; subject reduced and placed upper-centre with at least 55 percent quiet
empty field. Opaque flat paint-like colour areas with slightly irregular
hand-drawn edges and zero photographic surface detail. Palette: warm ivory,
charcoal brown, one muted earth tone, one cool neutral, plus at most one small
muted chroma focus — never a saturated anchor. Type optional: a single small fine serif title at the lower
margin. Avoid photographic realism, translucent overlays, true perspective,
wires, dense window grids, glossy gradients, neon, cover furniture, watermark,
extra text.
```

## Style-lock clause for reference photos

When results still look like a softened photograph, append:

```
This photograph is a content reference only. Do not preserve its photographic
detail, light, perspective, or material. Select only the most recognisable cues
and redraw them as separate flat editorial marks. Do not make a magazine cover.
```

## Notes

Before this file existed, these values were the engine's de facto default — `assets/palette.md` labelled warm ivory the "default panel," `styles/swiss.md` hardcoded `flat ivory ground`, and the READMEs' quick prompt pinned the paper and the paint. That made ivory the fallback whenever `ground` went undecided, which was every run, because no `ground` field existed.

The values are unchanged. What changed is that they are now **a named choice** rather than the only reachable outcome.

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
  intended_layouts: [poster, gallery-print, social-asset, moodboard]
  abstraction_level: relationship-first   # NOT full-abstract — see Notes
  palette: [warm ivory, charcoal brown, one muted earth tone, one cool neutral]
  typography: "one small fine serif title at the lower margin, or none"
  texture_tier: FLAT
  atmosphere: quiet, still, contemplative
  composition:
    subject_scale: small
    subject_position: upper-centre
    whitespace_ratio: 0.55              # minimum, not a target
    form_types: 3-5                     # KINDS of form, repeated as the scene needs — not a cap on objects
    projection: flat-elevation          # one viewpoint, held across every object
    ground_plane: shared                # everything stands on one surface
    light: single direction, flat contact shadow on every object
    relations: preserved                # positions, relative scale, overlaps survive the reduction

blocked_layouts: [zine]            # PRINT tier would break the flat ivory ground

free: [layout, aspect_ratio, title, subtitle, recoveries]

hard_constraints:
  - "The source photograph is a content reference only — never preserved, never filtered"
  - "At most one small MUTED chroma focus. MUJI DNA rejects saturated accents — this is not a Panter anchor"
  - "Detail is removed; spatial relationships are not. Objects that clustered still cluster ([../assets/scene-construction.md](../assets/scene-construction.md))"
  - "Every object touching the ground casts a flat contact shadow in one shared direction"
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

Photographic realism · translucent overlays · deep perspective recession and converging vanishing points · objects floating with no contact shadow · two viewpoints inside one object · a row of isolated objects where the source had a cluster · overhead wires · dense window grids · fine lattice structures · glossy gradients · neon · magazine-cover furniture (frames, barcodes, cover lines) · invented logos · watermark · any additional text

Note that **deep recession** is the avoid, not projection itself. A viewpoint-less image is what floats — see [../assets/scene-construction.md](../assets/scene-construction.md).

## Compiler anchor

```
Minimal editorial illustration on a warm ivory paper ground, flat and uniform,
no grain or stain. Rebuild the scene using three to five kinds of simplified
form, repeated as often as the scene needs; keep the arrangement the
photograph had — objects that clustered still cluster, overlaps and relative
sizes survive. Single flat elevation viewpoint held across every object, never
two angles within one object. All objects rest on one shared ground plane; one
light from the upper left, and every object casts a flat contact shadow in
that direction, one or two steps darker than the ground, never black. Subject
reduced and placed upper-centre with at least 55 percent quiet empty field.
Opaque paint-like colour areas, dabbed and brush-made, with slightly uneven
edges and faint tonal variation inside each shape; repeated elements differ
slightly from one another; no vector-clean outlines and zero photographic
surface detail. Palette: warm ivory, charcoal brown, one muted earth tone, one
cool neutral, plus at most one small muted chroma focus — never a saturated
anchor. Type optional: a single small fine serif title at the lower margin.
Avoid photographic realism, translucent overlays, deep perspective recession
and converging vanishing points, objects floating with no contact shadow, a
row of isolated objects where the source had a cluster, wires, dense window
grids, glossy gradients, neon, cover furniture, watermark, extra text.
```

## Style-lock clause for reference photos

When results still look like a softened photograph, append:

```
This photograph is a content reference only. Do not preserve its photographic
detail, light, or material. Keep its arrangement: what stood next to what, what
overlapped what, and how big things were relative to each other. Redraw the
recognisable cues as flat editorial marks on one shared ground plane, one
viewpoint, one light. Do not make a magazine cover.
```

## Notes

**`abstraction_level` was `full-abstract`, and that was wrong.** `full-abstract` licenses the engine to abandon what the photograph said about space, which is exactly how a courtyard of chairs around a table comes back as three unrelated pieces of furniture in a row. This preset reduces *detail*, not *relationships* — that is `relationship-first`. The three failure modes it fixes, all visible in real output: floating objects with no contact shadow, two viewpoints inside one object, and a cluster flattened into a row. See [../assets/scene-construction.md](../assets/scene-construction.md).

**`form_count: 3-5` was also wrong**, for the same reason: read as a cap on objects it deletes the scene. It is a cap on the *vocabulary* — three to five kinds of form, repeated as needed. Renamed `form_types`.

**«Avoid true perspective» was the third cause.** It told the model to drop projection entirely rather than to avoid deep recession, and a viewpoint-less image is what floats. The anchor now names the projection it wants instead of only naming what it does not.

Before this file existed, these values were the engine's de facto default — `assets/palette.md` labelled warm ivory the "default panel," `styles/swiss.md` hardcoded `flat ivory ground`, and the READMEs' quick prompt pinned the paper and the paint. That made ivory the fallback whenever `ground` went undecided, which was every run, because no `ground` field existed.

The values are unchanged. What changed is that they are now **a named choice** rather than the only reachable outcome.

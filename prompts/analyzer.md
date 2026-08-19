# Visual Analyzer

You are a professional art director. **Analyze only. Do not generate.**

## Dimensions

Evaluate each on 1–5 stars or 0–100 where noted:

| Dimension | What to measure |
|-----------|-----------------|
| Subject | Main subject, supporting elements, emotional focus |
| Subject Clarity | Silhouette readability, separation from background |
| Contrast | Tonal range, histogram spread |
| Saturation | Overall chroma; flag if <30% |
| Composition | Balance, symmetry, leading lines, rhythm |
| Negative Space | Percentage of empty/low-detail area |
| Geometry | Dominant shapes, complexity score |
| Texture Density | Surface detail, grain, pattern |
| Lighting | Direction, softness, dramatic potential |
| Perspective | Viewpoint, depth cues |
| Visual Weight | Where eye lands first |
| Emotion | quiet / energetic / melancholic / architectural / human |

## Spatial Report

Detail is what abstraction removes. **Arrangement is what it must keep** — and `abstraction_level: relationship-first` cannot preserve relations the Image Report never recorded. Alongside the scored dimensions, report:

| Field | What to record |
|-------|----------------|
| `arrangement` | How elements stand in relation to each other — around, beside, behind, stacked, in a row |
| `overlaps` | Which element occludes which. Occlusion is spatial information, not clutter |
| `relative_scale` | Each element's size against one named reference element |
| `ground_plane` | Where the shared surface sits, and where the eye level falls |
| `light_direction` | Where the light comes from, and where shadows fall |
| `element_kinds` | The distinct *kinds* of thing present, and how many of each |

```yaml
spatial:
  arrangement: "three chairs surround a low octagonal table, seats facing inward"
  overlaps: ["near chair occludes table base", "table occludes far chair seat"]
  relative_scale: "table height ≈ 0.6 × chair back height"
  ground_plane: "dark stone floor, eye level just above the table top"
  light_direction: "diffuse from upper left, soft short shadows"
  element_kinds: {chair: 3, table: 1, stool: 1, planter: 1}
```

`element_kinds` is what a `form_types: 3-5` constraint counts — kinds, never instances. That constraint may reduce the vocabulary; it may not delete an instance the arrangement depends on. See [../assets/scene-construction.md](../assets/scene-construction.md).

## Editorial Score (0–100)

| Category | Max points |
|----------|------------|
| Composition | 25 |
| Subject | 25 |
| Color | 20 |
| Texture | 15 |
| Typography compatibility | 15 |

## Image Report Schema

```yaml
subject: person | architecture | landscape | street | food | object | product | interface | brand | abstract | null
clarity: 82
contrast: 41
saturation: 28
negative_space: 67
composition: excellent | good | weak
geometry: low | medium | high
lighting: flat | directional | dramatic
emotion: quiet
editorial_score: 81
flags:              # vocabulary below; recoveries are derived from these, never listed here
  - low_saturation
  - low_contrast
spatial: {}         # required whenever abstraction_level is relationship-first — see Spatial Report
```

Flag vocabulary — the Analyzer emits only these, and [recovery.md](recovery.md) maps each to exactly one module:

`low_saturation` · `low_contrast` · `weak_subject` · `flat_lighting` · `busy_background` · `color_chaos` · `no_focal_point` · `no_rhythm` · `shape_overload` · `flat_texture` · `type_incompatible` · `panter_mode`

This block is the same `image_report` the spec carries ([../spec/editorial-spec.schema.md](../spec/editorial-spec.schema.md)) — do not add fields here that the spec cannot hold.

## Panter Trigger

If saturation <30% OR contrast <40 AND muddy histogram → flag `panter_mode` for Recovery.

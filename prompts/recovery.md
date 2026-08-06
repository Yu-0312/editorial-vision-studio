# Recovery Engine

Repair visual weakness **without destroying original identity**. Recovery is modular — load atomic fixes from [recovery/](../recovery/).

## Application Order

1. Read Image Report flags
2. Map each flag to one recovery module
3. Apply minimum set needed (avoid stacking redundant fixes)
4. Drop any texture clause the layout does not permit — see [../assets/texture.md](../assets/texture.md) → Texture Permission
5. Pass recovery list to Compiler as explicit prompt clauses

## Texture Gate

Texture permission belongs to `direction.layout`, not to a recovery module. Only `zine` is PRINT; every other layout is CLEAN, and a few targets are FLAT. On CLEAN layouts strip `riso_texture` from `recoveries`, keep only SURFACE tokens in `design_tokens.texture`, and substitute contrast, spacing, and mark scale.

## Flag → Module Map

| Flag | Module |
|------|--------|
| low_contrast, low_saturation | [contrast.md](../recovery/contrast.md) (Panter Mode) |
| weak_subject | [subject.md](../recovery/subject.md) |
| flat_lighting | [lighting.md](../recovery/lighting.md) |
| busy_background | [background.md](../recovery/background.md) |
| color_chaos | [palette.md](../recovery/palette.md) |
| no_focal_point | [focus.md](../recovery/focus.md) |
| no_rhythm | [composition.md](../recovery/composition.md) |
| shape_overload | [geometry.md](../recovery/geometry.md) |
| flat_texture | [texture.md](../recovery/texture.md) — PRINT layouts only; no-op on CLEAN |
| type_incompatible | [typography.md](../recovery/typography.md) |

## Panter Mode (Critical)

For gray/muddy inputs (saturation <30%). Panter is a **colour** compensation — it never grants texture permission.

1. **Discard** dull extracted grays as panel colors
2. **Extract conflict hues**: warmest pixel → 75% sat; coolest → 70% sat
3. **High-sat anchor**: ~8% panel area, brightness +40 vs panel average; default vermilion or cobalt if achromatic highlights
4. **Non-colour compensation**: widen tonal separation, interval, and mark scale to replace missing chroma complexity
5. **Texture overlay**: only on PRINT layouts (`zine`), riso halftone on abstract marks. On CLEAN layouts — including `photo-abstract-diptych` — emit no texture clause

## Reconstruction Threshold

Editorial Score <50 → Concept Reconstruction: increase abstraction, reduce literal photo dependency, rebuild hierarchy from geometry/emotion only.

# Ground & Render Mode

The two axes that decide what an image is *made of*. Both are **required with no default** in [../spec/editorial-spec.schema.md](../spec/editorial-spec.schema.md).

Enum values are contract tokens, not prompt words. Never write `paper-light` or `photo-plus-graphic` into a prompt — every adapter's `{ground}` and `{render_mode}` slot resolves through the tables below first.

## Why these are required

Before these fields existed, the engine had a palette, a texture tier, and a style — but nothing that named the field the marks sit on. With no field to set, every run inherited whatever the repo said most often, which was warm ivory paper. A required field with no fallback cannot be skipped; that is the entire mechanism.

`ground` and `render_mode` are independent of `abstraction_level`. Medium and distance-from-source are different questions: a `photographic` image can still be `full-abstract`.

## Ground → prompt language

| Token | Renders as | Palette behaviour | Default texture tier |
|-------|-----------|-------------------|----------------------|
| `paper-light` | warm ivory / cream / off-white paper field, flat and uniform | marks sit dark on light | FLAT, or SURFACE when style DNA rates Texture ★★★+ |
| `neutral-gray` | mid-tone gray or concrete field | marks read both directions; needs a value anchor | FLAT or SURFACE |
| `dark` | near-black, deep charcoal, or deep neutral field | marks sit light on dark; accents gain chroma | FLAT |
| `saturated` | a single flat chromatic field covering the canvas — the ground *is* ink | palette must include the ground hue as a member | FLAT or SURFACE |
| `full-bleed-photo` | the photograph is the canvas; no field behind it | palette is graded, not composed | **always FLAT** ([texture.md](texture.md)) |
| `duotone` | two-ink field, one light one dark, no third value | exactly two hues plus their blend | FLAT |

Rules:

- The ground is stated **first** in every compiled prompt ([../prompts/compiler.md](../prompts/compiler.md)). An unstated ground is the single most common way an image drifts back to paper white.
- `saturated` and `duotone` require the ground hue to appear in `design_tokens.palette`. A ground that is not in the palette is a rejection.
- On `dark` and `saturated`, invert the value logic of the style DNA rather than abandoning it — a Swiss grid on a dark ground is still a Swiss grid.
- Recovery modules never change the ground. Panter compensates chroma inside whatever ground is set ([../recovery/contrast.md](../recovery/contrast.md)).

## Render mode → prompt language

| Token | Renders as | Never contains |
|-------|-----------|----------------|
| `photographic` | a photograph: real light, real depth of field, real material | flat paint areas, vector edges, "illustration" |
| `photo-plus-graphic` | a preserved photographic region plus flat graphic marks in separate zones | painterly blending between the two; the boundary stays hard |
| `graphic` | flat colour areas, hard edges, no gradient modelling, vector-like | photographic surface detail, depth-of-field blur, airbrush |
| `painterly` | opaque flat paint-like areas with slightly irregular hand-drawn edges | photographic detail, gloss, true perspective |
| `mixed` | a declared combination — say which regions are which | an undeclared blend; `mixed` without a zone map is a rejection |

Rules:

- `photographic` forbids `8K`, `masterpiece`, `photorealistic` and the rest of the banned list. Photographic is a medium, not a quality claim ([../prompts/compiler.md](../prompts/compiler.md)).
- `graphic` and `painterly` forbid depth-of-field language entirely — no bokeh, no shallow focus.
- `photo-plus-graphic` is the `photo-abstract-diptych` lineage and implies `photo_policy.fidelity: required`.
- A prompt that describes one medium while the spec declares another is a Reviewer rejection, not a warning ([../prompts/reviewer.md](../prompts/reviewer.md)).

## Worked substitutions

| Spec | `{ground}` → | `{render_mode}` → |
|------|-------------|-------------------|
| `paper-light` + `painterly` | `warm ivory paper ground, flat and uniform` | `opaque flat paint-like colour areas, slightly irregular hand-drawn edges` |
| `saturated` + `graphic` | `flat deep-teal ink field covering the canvas` | `bold flat colour shapes, hard edges, no gradient modelling` |
| `full-bleed-photo` + `photographic` | `the photograph fills the frame; no field behind it` | `real light, shallow depth of field, realistic shadows` |
| `dark` + `photo-plus-graphic` | `near-black ground` | `preserved photographic region above, flat graphic panel below, hard boundary` |

## Preset defaults

Each shipped preset pins both axes — see [../presets/registry.md](../presets/registry.md). The three ship with three different grounds on purpose: it is what gives the Art Direction candidate-diversity rule something to draw on.

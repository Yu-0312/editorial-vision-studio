# VisualMemory — Cross-Image Consistency Contract

Carries a locked visual DNA across runs. Logic and lock policy: [../prompts/visual-memory.md](../prompts/visual-memory.md).

## Schema

```yaml
memory_version: "1.0"

memory_id: tokyo-series
version: "1.0"                            # presets bump this on edit; manifests record the version used
established_from: tokyo-tower-editorial-r1   # run_id, "brand_input", or "preset"
source: generated | user_provided | preset

locked:
  ground: paper-light | neutral-gray | dark | saturated | full-bleed-photo | duotone
  render_mode: photographic | photo-plus-graphic | graphic | painterly | mixed
  style: swiss
  visual_language: Architectural
  palette: [warm ivory, charcoal, muted red]
  typography: "grotesk title, sans metadata"
  texture_tier: FLAT | SURFACE | PRINT
  atmosphere: quiet contemporary            # soft-lock
  # spatial_plan — presets only, field by field (projection, ground_plane,
  #   light_direction, arrangement, overlaps, relative_scale); keys must match
  #   direction.spatial_plan exactly. See validation rules below

blocked_layouts: [zine]                     # required whenever `layout` is free

free:
  - layout
  - composition
  - abstraction_level
  - aspect_ratio
  - title
  - recoveries

hard_constraints: []                        # brand mode only
  # - "Never invent a logo mark"
  # - "Never alter supplied brand hues"

runs:
  - tokyo-tower-editorial-r1
  - tokyo-street-editorial-r1

forked_from: null                           # memory_id when this is a fork
```

## Validation Rules

- `locked.style` and `locked.visual_language` must be consistent with the Visual Language catalog ([../prompts/visual-language.md](../prompts/visual-language.md)).
- `locked.texture_tier` must be a single tier. A memory that permits two tiers is not a memory.
- `locked.texture_tier: PRINT` implies every run in the series uses `layout: zine`. Reject any other layout while that lock holds — see [../assets/texture.md](../assets/texture.md).
- A field may not appear in both `locked` and `free`.
- `composition` may never be locked when `source: generated` or `user_provided`. Locking it produces a template, not a system — four images with the subject in the same corner.
- `source: preset` **may** lock `composition`, `aspect_ratio`, `layout`, and `spatial_plan` (field by field). A preset is avowedly a template; that is what it is for. See [../presets/registry.md](../presets/registry.md).
- `locked.spatial_plan` keys must be exactly `direction.spatial_plan`'s keys — `projection`, `ground_plane`, `light_direction`, `arrangement`, `overlaps`, `relative_scale`. A spatial constraint written under `composition` under a different name (`camera`, `light`, `relations`) is a schema violation: the Reviewer's locked-field check cannot see it, and it silently duplicates a field the spec already owns.
- Spatial-lock precedence: on a photo run, `direction.spatial_plan` starts as the verbatim copy of `image_report.spatial`, then each locked spatial field **overrides the copy** — a locked `projection: flat-elevation` beats the photographed perspective, and that is the point. Unlocked spatial fields keep the photographed values. On a theme-only run, unlocked spatial fields are authored by the Planner as usual; a preset that locks `projection` but not `light_direction` still gets a Planner-authored light.
- `spatial_plan` sits outside the locked/free bookkeeping below: it is required on every run regardless, so a memory may lock none, some, or all of its fields while the rest follows the normal source rules.
- Every field in the canonical `free` list must appear in either `locked` or `free`. A field in neither can be set by nobody — `layout` and `abstraction_level` are the two that go missing most often, and both are required in every spec.
- A memory that leaves `layout` free must list `blocked_layouts`, because texture permission is decided by `direction.layout` ([../assets/texture.md](../assets/texture.md)) and an unrestricted memory could select `zine` and pull PRINT into a FLAT system.
- `locked.ground` and `locked.render_mode` are mandatory in every memory. They are the two axes that have no safe default.
- `source: user_provided` (brand mode) forbids soft-locks: `atmosphere` becomes a hard lock, and mutation requires explicit user instruction.
- `established_from` must reference a run that passed QC — `quality.overall ≥ 0.85` **and** no dimension below 0.60 — unless `source` is `user_provided` or `preset`, in which case it is the literal `brand_input` or `preset`.
- Changing any `locked` value requires a **fork**: new `memory_id`, `forked_from` set. Never mutate in place.
- Exception: `source: preset` is an authored definition with no run history to invalidate, so it may be edited in place — but **bump `version`**. Fork instead when the change would break a series already built on it.

## Enforcement Points

| Layer | Enforces |
|-------|----------|
| [../prompts/art-direction.md](../prompts/art-direction.md) | Auto-commits to locked DNA; offers no candidates |
| [../prompts/planner.md](../prompts/planner.md) | Sets `free` fields only |
| [../prompts/compiler.md](../prompts/compiler.md) | Compiles locked palette/typography/texture verbatim |
| [../prompts/reviewer.md](../prompts/reviewer.md) | Rejects any GenerationRequest that contradicts a lock |
| [../prompts/series.md](../prompts/series.md) | Cross-image consistency scoring |
| [../presets/registry.md](../presets/registry.md) | Ships authored memories with `source: preset` |

## Minimal Example

```yaml
memory_version: "1.0"
memory_id: acme-2026
version: "1.0"
established_from: brand_input
source: user_provided
locked:
  ground: paper-light
  render_mode: photo-plus-graphic
  style: cos
  visual_language: Brand System Calm
  palette: [ivory, ink black, signal red]
  typography: "brand grotesk, tight tracking"
  texture_tier: FLAT
  atmosphere: controlled premium
blocked_layouts: [zine]
free: [layout, composition, abstraction_level, aspect_ratio, title, recoveries]
hard_constraints:
  - "Never invent a logo mark"
  - "Never alter supplied brand hues"
runs: []
```

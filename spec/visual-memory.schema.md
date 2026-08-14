# VisualMemory — Cross-Image Consistency Contract

Carries a locked visual DNA across runs. Logic and lock policy: [../prompts/visual-memory.md](../prompts/visual-memory.md).

## Schema

```yaml
memory_version: "1.0"

memory_id: tokyo-series
established_from: tokyo-tower-editorial-r1   # run_id, or "brand_input"
source: generated | user_provided

locked:
  style: swiss
  visual_language: Architectural
  palette: [warm ivory, charcoal, muted red]
  typography: "grotesk title, sans metadata"
  texture_tier: FLAT | SURFACE | PRINT
  atmosphere: quiet contemporary            # soft-lock

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
- `composition` may never be locked. Locking it produces a template, not a system.
- `source: user_provided` (brand mode) forbids soft-locks: `atmosphere` becomes a hard lock, and mutation requires explicit user instruction.
- `established_from` must reference a run that passed QC — `quality.overall ≥ 0.85` **and** no dimension below 0.60 — unless `source: user_provided`.
- Changing any `locked` value requires a **fork**: new `memory_id`, `forked_from` set. Never mutate in place.

## Enforcement Points

| Layer | Enforces |
|-------|----------|
| [../prompts/art-direction.md](../prompts/art-direction.md) | Auto-commits to locked DNA; offers no candidates |
| [../prompts/planner.md](../prompts/planner.md) | Sets `free` fields only |
| [../prompts/compiler.md](../prompts/compiler.md) | Compiles locked palette/typography/texture verbatim |
| [../prompts/reviewer.md](../prompts/reviewer.md) | Rejects any GenerationRequest that contradicts a lock |
| [../prompts/series.md](../prompts/series.md) | Cross-image consistency scoring |

## Minimal Example

```yaml
memory_version: "1.0"
memory_id: acme-2026
established_from: brand_input
source: user_provided
locked:
  style: cos
  visual_language: Brand System Calm
  palette: [ivory, ink black, signal red]
  typography: "brand grotesk, tight tracking"
  texture_tier: FLAT
  atmosphere: controlled premium
free: [layout, composition, abstraction_level, aspect_ratio, title, recoveries]
hard_constraints:
  - "Never invent a logo mark"
  - "Never alter supplied brand hues"
runs: []
```

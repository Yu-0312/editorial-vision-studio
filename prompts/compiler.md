# Prompt Compiler (Orchestrator)

The Compiler is a **two-phase orchestrator**. It does not embed model syntax.

## Phase 1: Build VisionSpec / EditorialSpec (model-agnostic)

Aggregate outputs from Decision Engine into [spec/editorial-spec.schema.md](../spec/editorial-spec.schema.md).

Pull modules from:

- Layout: [layouts/](../layouts/)
- Style DNA: [styles/](../styles/)
- Assets: [assets/typography.md](../assets/typography.md), [assets/palette.md](../assets/palette.md), [assets/texture.md](../assets/texture.md)
- Recovery clauses: [recovery/](../recovery/)
- Conditional: [references/photo-abstract-prompt.*](../references/) when diptych; [assets/variation-engine.md](../assets/variation-engine.md) when zine

**Compiler never analyzes the image.** It validates VisionSpec / EditorialSpec against schema rules.

## Phase 2: Route to Model Adapter (swappable)

1. Resolve `target.model` via [adapters/registry.md](../adapters/registry.md)
2. Load `adapters/{model}.md`
3. Translate VisionSpec / EditorialSpec → `GenerationRequest`
4. Pass to Prompt Reviewer

| Model | Adapter |
|-------|---------|
| gpt-image | [adapters/gpt-image.md](../adapters/gpt-image.md) |
| flux | [adapters/flux.md](../adapters/flux.md) |
| ideogram | [adapters/ideogram.md](../adapters/ideogram.md) |
| generic | [adapters/generic.md](../adapters/generic.md) |

User override: `model: flux` in request → sets `target.model` before Phase 2.

## Prompt Hygiene (all adapters)

**Write:** concrete visual constraints — `Large negative space`, `One serif title`, `Warm ivory background`

**Never write:** beautiful, professional, minimal, elegant, high quality, award winning, stunning, masterpiece

## Renderability Gate

Compile only information that can change final pixels. Convert the brief into this order:

1. canvas and surface
2. attention geometry and negative-space budget
3. one primary image anchor and its treatment
4. typography or copy-safe behavior
5. palette, texture, lighting, and explicit avoids

Exclude source paths, planning rationale, sample-specific copy, and generic checklist language. Keep exact in-image text short; image models are unreliable with long text. For zines, enforce the selected variation recipe and make its saturated anchor visible at thumbnail scale.

## Adapter Output Contract

Every adapter must emit:

```yaml
generation_request:
  model: string
  prompt: string
  negative_prompt: string | null
  aspect_ratio: string
  reference_image: keep | edit | none
  extra_params: {}
```

Reviewer validates `generation_request`, not raw EditorialSpec.

## When User Switches Model Only

If user says "same direction, but generate with Flux":

1. **Reuse VisionSpec / EditorialSpec** — do not re-run Analyzer/Planner
2. Re-run Phase 2 with new adapter only
3. Reviewer + Evaluator as normal

## Conditional References

| Condition | Read |
|-----------|------|
| layout = photo-abstract diptych | [references/photo-abstract-prompt.en.md](../references/photo-abstract-prompt.en.md) |
| layout = zine | [assets/variation-engine.md](../assets/variation-engine.md) |
| intent = Event Campaign | [layouts/campaign-poster.md](../layouts/campaign-poster.md) |
| intent = Branding | [layouts/brand-key-visual.md](../layouts/brand-key-visual.md) |
| intent = Product / Object | [layouts/product-editorial.md](../layouts/product-editorial.md) |
| intent = Digital Product | [layouts/website-hero.md](../layouts/website-hero.md) |
| layout = social-asset | [layouts/social-asset.md](../layouts/social-asset.md) |
| layout = moodboard | [layouts/moodboard.md](../layouts/moodboard.md) |
| layout = interface-asset | [layouts/interface-asset.md](../layouts/interface-asset.md) |
| architecture overview | [reference/architecture.md](../reference/architecture.md) |

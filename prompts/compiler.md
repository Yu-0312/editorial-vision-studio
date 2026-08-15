# Prompt Compiler (Orchestrator)

The Compiler is a **two-phase orchestrator**. It does not embed model syntax.

## Phase 1: Build VisionSpec / EditorialSpec (model-agnostic)

Aggregate outputs from Decision Engine into [spec/editorial-spec.schema.md](../spec/editorial-spec.schema.md).

Pull modules from:

- Layout: [layouts/](../layouts/)
- Style DNA: [styles/](../styles/)
- Assets: [assets/ground.md](../assets/ground.md), [assets/scene-construction.md](../assets/scene-construction.md), [assets/typography.md](../assets/typography.md), [assets/palette.md](../assets/palette.md), [assets/texture.md](../assets/texture.md)
- Recovery clauses: [recovery/](../recovery/)
- Conditional: [layouts/photo-abstract-diptych.md](../layouts/photo-abstract-diptych.md) when diptych; [assets/variation-engine.md](../assets/variation-engine.md) when zine

**Compiler never analyzes the image.** It validates VisionSpec / EditorialSpec against schema rules.

**Compiler never re-decides.** When `memory_id` is set, locked palette, typography, style, and texture tier are compiled verbatim — the Compiler has no discretion over them ([../spec/visual-memory.schema.md](../spec/visual-memory.schema.md)).

**Compiler does correct schema violations.** Resolving `design_tokens.texture_tier` against [assets/texture.md](../assets/texture.md), and keeping `photo_policy.fidelity` consistent with `avoids`, are validation fixes it already owns — this is what [iteration.md](iteration.md) means when it names the Compiler as a responsible layer. Creative fields (style, palette, layout, direction) stay untouched.

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

1. canvas and surface — state `design_tokens.ground` explicitly and first, resolved to prose via [assets/ground.md](../assets/ground.md); never emit the enum token. An unstated ground is the single most common way an image drifts back to paper white
2. **space** — projection, shared ground plane, light direction and contact shadows, and the source's preserved arrangement ([assets/scene-construction.md](../assets/scene-construction.md)). Skipping this is what turns a reduced scene into floating clip art
3. attention geometry and negative-space budget
4. one primary image anchor and its treatment
5. typography or copy-safe behavior
6. palette, texture, lighting, and explicit avoids

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

Reviewer validates `generation_request` against the spec it was compiled from, and may correct the request — never the spec.

## When User Switches Model Only

If user says "same direction, but generate with Flux":

1. **Reuse VisionSpec / EditorialSpec** — do not re-run Analyzer/Planner
2. Re-run Phase 2 with new adapter only
3. Reviewer + Evaluator as normal

## Phase 3: Emit Manifest

The Compiler **owns** [../spec/visual-manifest.schema.md](../spec/visual-manifest.schema.md): it opens the manifest when Phase 1 first compiles a spec, the Evaluator appends the quality vector, the Iteration Engine appends each pass, and the Compiler finalizes it when the run settles. This is what makes "regenerate that one" and "try it in Flux" cheap instead of a full re-run.

Keep the manifest internal unless the user asks for it or the run is part of a series.

## Conditional References

| Condition | Read |
|-----------|------|
| layout = photo-abstract diptych | [layouts/photo-abstract-diptych.md](../layouts/photo-abstract-diptych.md) |
| layout = zine | [assets/variation-engine.md](../assets/variation-engine.md) |
| intent = Event Campaign | [layouts/campaign-poster.md](../layouts/campaign-poster.md) |
| intent = Branding | [layouts/brand-key-visual.md](../layouts/brand-key-visual.md) |
| intent = Product / Object | [layouts/product-editorial.md](../layouts/product-editorial.md) |
| intent = Digital Product | [layouts/website-hero.md](../layouts/website-hero.md) |
| layout = social-asset | [layouts/social-asset.md](../layouts/social-asset.md) |
| layout = moodboard | [layouts/moodboard.md](../layouts/moodboard.md) |
| layout = interface-asset | [layouts/interface-asset.md](../layouts/interface-asset.md) |
| layout = presentation-deck | [layouts/presentation-deck.md](../layouts/presentation-deck.md) |
| `series_id` set | [series.md](series.md) |
| `preset` set | [presets/registry.md](../presets/registry.md) |
| `style_gate.reason: freeform` | [style-brief.md](style-brief.md) |
| `memory_id` set, or brand assets supplied | [visual-memory.md](visual-memory.md) |
| architecture overview | [reference/architecture.md](../reference/architecture.md) |

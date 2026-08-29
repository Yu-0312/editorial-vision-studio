# Model Adapter Registry

Select adapter by `target.model` in VisionSpec / EditorialSpec or user request.

| Model ID | Adapter file | Best for | Reference image |
|----------|--------------|----------|-----------------|
| `gpt-image` | [gpt-image.md](gpt-image.md) | Photo editing, diptych with upload | **Required** when photo_policy.fidelity=required |
| `flux` | [flux.md](flux.md) | Atmospheric editorial, texture | Optional img2img |
| `ideogram` | [ideogram.md](ideogram.md) | Typography-heavy covers, campaign type | Optional |
| `generic` | [generic.md](generic.md) | Unknown / fallback | As available |

## Auto-Detection

| Signal | Default model |
|--------|---------------|
| User uploads photo + diptych/cover | `gpt-image` |
| User mentions Flux / fal / BFL | `flux` |
| User mentions Ideogram / poster text | `ideogram` |
| Theme-only zine, no photo | `flux` or `ideogram` |
| Website hero / interface asset | `flux` or `generic` |
| Product or brand visual with uploaded reference | `gpt-image` |
| Brand/campaign/social asset with important text | `ideogram` |
| Moodboard / concept atmosphere | `flux` |
| Unspecified | `generic` |

User override always wins: `model: flux`

## Adapter Selection Flow

```
1. Decision Engine completes → VisionSpec / EditorialSpec
2. Resolve target.model (user > auto-detect > generic)
3. Load adapters/{model}.md
4. Translate VisionSpec / EditorialSpec → GenerationRequest
5. Prompt Reviewer validates output — legality
6. Prompt Optimizer rewrites wording against the adapter's `optimizer_contract`
7. Route to image API
```

## GenerationRequest (adapter output)

All adapters emit:

```yaml
generation_request:
  model: string
  prompt: string
  negative_prompt: string | null
  aspect_ratio: string
  reference_image: keep | edit | none
  extra_params: {}   # model-specific, documented per adapter
```

Every adapter also declares an `optimizer_contract` block — sentence budget, negative-prompt support, emphasis order, clause density. [../prompts/optimizer.md](../prompts/optimizer.md) reads it and holds no per-model knowledge of its own, which is what keeps a new model from touching a shared layer.

| Model ID | Budget | Negatives | Emphasis |
|----------|--------|-----------|----------|
| `gpt-image` | none | inline (paragraph 4) | ground-first |
| `flux` | 3 sentences | `negative_prompt` field | ground-first |
| `ideogram` | 4 sentences | `negative_prompt` field | **title-first** |
| `generic` | none | `negative_prompt` field | ground-first |

## Extending

Copy [_template.md](_template.md), register here — the `optimizer_contract` block included. Do **not** fork the Decision Engine.

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
5. Prompt Reviewer validates output
6. Route to image API
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

## Extending

Copy [_template.md](_template.md), register here. Do **not** fork the Decision Engine.

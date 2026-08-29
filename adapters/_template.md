# Model Adapter Template

Copy to `adapters/your-model.md` and register in [registry.md](registry.md).

## Model ID

`your-model-id`

## Strengths

- [What this model does best in editorial workflow]

## Reference Image Policy

| photo_policy.fidelity | Action |
|-----------------------|--------|
| `required` | [edit / img2img / warn / unsupported] |
| `optional` | [loose reference / low-strength img2img / text only] |
| `none` | [text-to-image] |

## Prompt Shape

[Describe optimal prompt structure for this model]

## Negative Prompt

[Separate field? Inline? Not supported?]

## Optimizer Contract

Model facts [../prompts/optimizer.md](../prompts/optimizer.md) reads. Required — an adapter without this block makes `compress`, `route_negatives`, and `ground_first` unrunnable.

```yaml
optimizer_contract:
  sentence_budget: null           # int, or null for no stated limit
  negative_prompt: field          # field | inline | unsupported
  emphasis_order: ground-first    # ground-first | subject-first | title-first
  clause_density: bound           # terse | bound | narrative — what this model responds to
```

`emphasis_order` is the clause this model weights most heavily, and it is **not** a preference — it is why `ground_first` fires differently per model. Declare it from the model's documented behaviour, not from taste.

## VisionSpec / EditorialSpec → Prompt Mapping

| Spec field | This model's dialect |
|------------|---------------------|
| `direction.title` | |
| `design_tokens.palette` | |
| `recoveries: *` | |

## Default Params

```yaml
extra_params:
  key: value
```

## Example GenerationRequest

```yaml
generation_request:
  model: your-model-id
  prompt: |
    ...
  negative_prompt: "..."
  aspect_ratio: "3:4"
  reference_image: none | edit | keep
  extra_params: {}
```

## Notes

- Adapters translate VisionSpec / EditorialSpec only — never re-run Analyzer or Planner
- Do not fork style/layout/recovery modules per model

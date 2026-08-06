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

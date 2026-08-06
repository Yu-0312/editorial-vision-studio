# Model Adapter: Generic (Fallback)

Use when model is unknown or user has not specified a backend.

## Policy

- Emit **both** a natural-language prompt (GPT shape) and a compact variant (Flux shape)
- Use standard negative prompt list
- Document which dialect user should paste into their tool

## Prompt Shape

Primary: 4-paragraph GPT Image shape from [../prompts/compiler.md](../prompts/compiler.md)

Also append:

```
--- Compact variant (Flux/Ideogram) ---
{2-sentence dense version}
```

## Reference Image

Follow EditorialSpec `photo_policy` — note in output:

```yaml
reference_image_note: "Upload source photo as reference for edit/img2img if your model supports it"
```

## Default Params

```yaml
extra_params: {}
```

## Example GenerationRequest

```yaml
generation_request:
  model: generic
  prompt: |
    [4-paragraph full prompt]
    --- Compact variant ---
    Vertical 3:4 ivory editorial poster, preserved upper photo, abstract lower panel,
    flat uniform panel ground, serif "After Rain", quiet mood.
  negative_prompt: "glossy, commercial, HDR, cinematic, 3D, neon, mockup, watermark"
  aspect_ratio: "3:4"
  reference_image: keep
  extra_params: {}
```

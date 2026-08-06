# Model Adapter: GPT Image

For OpenAI GPT Image / gpt-4o image generation / ChatGPT image tools.

## Strengths

- Reference image editing with high fidelity
- Best choice for **photo-abstract diptych** (preserve upload, edit compose)
- Natural language prompts, 4-paragraph shape

## Reference Image Policy

| photo_policy.fidelity | Action |
|-----------------------|--------|
| `required` | Pass uploaded image as reference; prompt must state "preserve photograph exactly" |
| `optional` | Use uploaded image as loose reference if available; prompt may say "inspired by reference" |
| `none` | Text-to-image only |

## Prompt Shape

Four compact paragraphs (see [../prompts/compiler.md](../prompts/compiler.md)):

1. Canvas + aspect + negative space
2. Subject / photo region + abstract derivation
3. Typography + palette + texture + recoveries
4. Atmosphere + inline avoid list (`Avoid: …`)

## Negative Prompt

GPT Image: embed avoids in paragraph 4 as prose list. Separate `negative_prompt` field optional.

```
Avoid: glossy ad, cinematic lighting, 3D, neon, photo redraw, mockup frame, watermark.
```

## VisionSpec / EditorialSpec → Prompt Mapping

| Spec field | Prompt clause |
|------------|---------------|
| `direction.aspect_ratio` | "Vertical {ratio} canvas" |
| `composition.photo_ratio` | "Upper {N}% preserves uploaded photograph exactly" |
| `composition.abstract_ratio` | "Lower {N}% abstract memory panel" |
| `direction.layout: website-hero` | "Wide hero image with clear copy-safe negative space" |
| `direction.layout: brand-key-visual` | "Adaptable brand key visual with one core visual sign" |
| `direction.layout: product-editorial` | "Product/object silhouette remains readable and proportionally correct" |
| `direction.layout: interface-asset` | "Clean symbolic app asset with UI-safe whitespace" |
| `direction.title` | "One small {typography} title \"{title}\"" |
| `design_tokens.palette` | Join as "muted {a}, {b}, {c} palette" |
| `recoveries: panter_mode` | Panter clauses from [../recovery/contrast.md](../recovery/contrast.md) — use the CLEAN variant unless layout is `zine` |
| `photo_policy.fidelity: required` | Mandatory fidelity sentence in paragraph 2 |

## Default Params

```yaml
extra_params:
  quality: high
  temperature: 0.65
```

## Example GenerationRequest

```yaml
generation_request:
  model: gpt-image
  prompt: |
    Vertical editorial poster, 3:4 canvas, warm ivory uninterrupted background.
    Upper 65% preserves uploaded street photograph exactly — no retouching or restyling.
    Lower 25% sparse abstract memory panel: compensated warm amber and cool slate blocks.
    One thin Helvetica caption "After Rain". Panel ground flat and uniform, no grain or paper texture.
    Cobalt anchor 8% at lower-right. Flat view, no mockup.
    Avoid: glossy ad, cinematic lighting, photo redraw, watermark.
  negative_prompt: null
  aspect_ratio: "3:4"
  reference_image: edit
  extra_params:
    quality: high
```

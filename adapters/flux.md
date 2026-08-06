# Model Adapter: Flux

For Black Forest Labs Flux (Flux Pro, Flux Dev, fal.ai, Replicate, etc.).

## Strengths

- Atmospheric texture, paper grain, riso/xerox defects — usable only when the layout permits it ([../assets/texture.md](../assets/texture.md))
- Strong zine and gallery moods
- Shorter prompts often outperform long essays

## Reference Image Policy

| photo_policy.fidelity | Action |
|-----------------------|--------|
| `required` | **Warn**: Flux img2img varies by host; prefer GPT Image for strict diptych fidelity. If forced, use img2img strength ≤0.35 on composed layout only |
| `optional` | Use text-to-image or low-strength img2img; treat reference as mood/composition, not identity lock |
| `none` | Text-to-image — ideal default for Flux |

## Prompt Shape

**Compact 2–3 sentences.** Flux responds to dense visual nouns, not paragraphs.

```
{aspect} aged ivory editorial poster, {photo_or_subject clause}, {palette} palette,
{focal texture}, {atmosphere}, {typography hint}, {recovery keyword}
```

## Negative Prompt

Use dedicated `negative_prompt` field — Flux supports explicit negatives:

```
glossy, commercial, HDR, cinematic, 3D, neon, mockup, watermark, oversaturated,
blurry text, ugly typography, low quality, jpeg artifacts
```

Add `photo redraw, altered faces` when fidelity=required.

## VisionSpec / EditorialSpec → Prompt Mapping

| Spec field | Flux dialect |
|------------|--------------|
| `design_tokens.texture` | Lead with material. PRINT tokens ("risograph grain", "xerox halftone") only on `zine`; CLEAN layouts get SURFACE wording ("cotton paper surface") at most; FLAT targets get none ([../assets/texture.md](../assets/texture.md)) |
| `design_tokens.palette` | 2–3 color nouns max |
| `direction.title` | Append: `small serif text "{title}"` — keep short |
| `recoveries: panter_mode` | "high saturation cobalt anchor, warm-cool contrast blocks" — colour only, add grain words only if layout is `zine` |
| `direction.layout: zine` | "70% negative space, tiny lower-left cluster" |
| `direction.layout: moodboard` | "curated material fragments, color samples, coherent style frame" |
| `direction.layout: website-hero` | "wide hero image, calm negative space for copy" |
| `direction.layout: social-asset` | "mobile-readable focal composition, strong crop" |
| `direction.layout: interface-asset` | "simple symbolic object, UI-safe whitespace" |
| Long avoid lists | Move to negative_prompt, not inline |

## Default Params

```yaml
extra_params:
  guidance: 3.5      # Flux Dev; Pro may differ
  steps: 28
  prompt_upsampling: false
```

## Example GenerationRequest

```yaml
generation_request:
  model: flux
  prompt: |
    Vertical 3:4 warm ivory editorial poster, upper street photo preserved,
    lower abstract geometric panel, flat uniform ground, warm amber and slate blocks,
    small serif "After Rain", quiet urban mood, flat scan view.
  negative_prompt: "glossy, commercial, HDR, cinematic, 3D, neon, mockup, watermark, photo redraw"
  aspect_ratio: "3:4"
  reference_image: none
  extra_params:
    guidance: 3.5
    steps: 28
```

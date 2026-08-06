# Model Adapter: Ideogram

For Ideogram v2/v3 — typography-forward editorial posters and covers.

## Strengths

- Readable poster text, campaign titles, magazine mastheads
- Bold layout hierarchy with type as graphic element
- Good for Campaign Poster, Magazine Cover when title matters

## Reference Image Policy

| photo_policy.fidelity | Action |
|-----------------------|--------|
| `required` | Use Ideogram **Edit** or **Remix** with reference; state "keep photo region unchanged" early in prompt |
| `optional` | Use reference only for loose composition/style if helpful |
| `none` | Text-to-image — leverage typography strength |

## Prompt Shape

**Lead with typography hierarchy**, then visual scene:

```
Editorial {layout}: "{title}" in {typography style}, {aspect ratio},
{subject/visual description}, {palette}, {texture}, {atmosphere}.
Text must be legible and correctly spelled.
```

Ideogram prioritizes text clauses — put `direction.title` in **first sentence**.

## Negative Prompt

```
misspelled text, garbled letters, watermark, logo, stock photo badge,
glossy advertisement, 3D render, neon, cluttered layout, low resolution
```

## VisionSpec / EditorialSpec → Prompt Mapping

| Spec field | Ideogram dialect |
|------------|------------------|
| `direction.title` | **First** — quoted exact string, spell-check critical |
| `direction.layout: campaign-poster` | "event campaign poster with bold hierarchy" |
| `direction.layout: magazine-cover` | "magazine cover masthead" |
| `direction.layout: brand-key-visual` | "brand key visual with restrained readable hierarchy" |
| `direction.layout: social-asset` | "social campaign asset with mobile-readable type" |
| `direction.layout: website-hero` | "web hero graphic with copy-safe negative space" |
| `design_tokens.typography` | Explicit: "elegant Garamond serif masthead" |
| `recoveries` | Shorter visual clauses; avoid overloading text instructions |
| `direction.subtitle` | Second line, smaller scale |

## Typography Emphasis Rule

When `type_ratio > 0.15`, prepend:

```
Typography-forward editorial design. Primary text: "{title}". Text must be sharp and readable.
```

## Default Params

```yaml
extra_params:
  magic_prompt: false    # keep EditorialSpec control
  style_type: Design     # or Auto for campaign
  rendering: Default
```

## Example GenerationRequest

```yaml
generation_request:
  model: ideogram
  prompt: |
    Typography-forward magazine cover: "Quiet Hours" in light Garamond serif masthead,
    vertical 3:4, portrait in lower 40%, large cream negative space above,
    warm sage palette, soft cotton paper texture, calm editorial mood.
    Text must be legible and correctly spelled.
  negative_prompt: "misspelled text, garbled letters, watermark, glossy ad, 3D, neon, cluttered"
  aspect_ratio: "3:4"
  reference_image: none
  extra_params:
    magic_prompt: false
    style_type: Design
```

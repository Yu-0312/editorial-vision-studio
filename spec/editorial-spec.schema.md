# VisionSpec / EditorialSpec — Model-Agnostic Contract

Emitted by the Decision Engine **before** Model Adapter runs.
All adapters consume this schema. Do not embed model-specific syntax here.

Use `EditorialSpec` for backward compatibility. For general tasks, treat the same schema as `VisionSpec`.

## Schema

```yaml
spec_version: "1.0"

# --- From Intent Engine ---
intent:
  goal: string
  family: Visual Concept | Art Book | Event Campaign | Gallery | Zine | Magazine | Branding | Product / Object | Digital Product | Social | Interface Asset
  allowed_outputs: [string]
  blocked_outputs: [string]
  language: en | zh  # user-facing summary language, not prompt language
  source_type: photo | theme | brand | product | interface | mixed

# --- From Visual Analyzer (null if theme-only / prompt-only) ---
image_report:
  subject: person | architecture | landscape | street | food | object | product | interface | brand | abstract | null
  clarity: 0-100
  contrast: 0-100
  saturation: 0-100
  negative_space: 0-100
  composition: excellent | good | weak
  emotion: string
  editorial_score: 0-100
  flags: [string]  # e.g. low_saturation, panter_mode

# --- From Visual Language + Planner ---
direction:
  visual_language: string
  layout: poster | magazine-cover | gallery-print | zine | editorial-spread | campaign-poster | photo-abstract-diptych | brand-key-visual | product-editorial | website-hero | social-asset | moodboard | interface-asset
  style: string  # swiss, kinfolk, muji, ...
  editorial_mode: premium | standard | compensation | reconstruction
  abstraction_level: relationship-first | identity-cue | full-abstract
  composition:
    photo_ratio: 0.0-1.0      # 0 if no photo
    abstract_ratio: 0.0-1.0
    type_ratio: 0.0-1.0
    whitespace_ratio: 0.0-1.0
  aspect_ratio: "3:4" | "2:3" | "3:5" | "1:1" | "16:9"
  title: string | null        # 2-5 word English title
  subtitle: string | null
  production_context: print | social | web | interface | prompt_only

# --- From Style + Assets modules ---
design_tokens:
  palette: [string]           # named hues, not hex-only
  typography: string          # e.g. "thin serif, caption scale"
  texture: [string]           # PRINT tokens (riso_halftone, xerox, scan_noise) only when direction.layout == zine
                              # CLEAN layouts: SURFACE tokens only (cotton_paper, natural_material, matte_board) or []
                              # FLAT targets: always []. See assets/texture.md
  atmosphere: string          # e.g. quiet cinematic
  brand_cues: [string]         # null/empty unless user provided brand or product cues

# --- From Recovery Engine ---
recoveries: [string]          # module IDs: panter_mode, silhouette_boost, ...
                              # riso_texture is valid only when direction.layout == zine

# --- Photo fidelity (photo-abstract-editorial lineage) ---
photo_policy:
  fidelity: required | optional | none     # required = never redraw source region
  reference_image: uploaded | none
  source_region: upper | principal | full-bleed

# --- Shared avoid list (model-agnostic) ---
avoids:
  - glossy commercial ad
  - cinematic HDR lighting
  - 3D render
  - neon
  - mockup device frame
  - watermark
  - photo redraw  # when fidelity=required

# --- Adapter routing (set by user or auto-detect) ---
target:
  model: gpt-image | flux | ideogram | generic
  prompt_only: false
```

## Validation Rules

- `recoveries` must match `image_report.flags` — no orphan recoveries
- If `photo_policy.fidelity = required`, `avoids` must include `photo redraw`
- If `layout = photo-abstract-diptych`, `photo_policy.fidelity` must be `required`
- If `editorial_mode = reconstruction`, `abstraction_level` should be `full-abstract`
- If `production_context = web`, preserve copy-safe negative space and avoid fake UI unless requested
- If `production_context = interface`, avoid fake text, fake controls, and unreadable UI details
- Sum of composition ratios ≈ 1.0 (±0.1)

## Example (minimal)

```yaml
spec_version: "1.0"
intent:
  goal: "editorial poster from gray street photo"
  family: Gallery
  allowed_outputs: [editorial_poster, photo_abstract_diptych]
  language: zh
image_report:
  subject: street
  clarity: 58
  contrast: 35
  saturation: 22
  negative_space: 40
  composition: good
  emotion: quiet
  editorial_score: 54
  flags: [low_saturation, low_contrast, panter_mode]
direction:
  visual_language: Architectural
  layout: photo-abstract-diptych
  style: swiss
  editorial_mode: compensation
  abstraction_level: relationship-first
  composition:
    photo_ratio: 0.65
    abstract_ratio: 0.25
    type_ratio: 0.05
    whitespace_ratio: 0.05
  aspect_ratio: "3:4"
  title: "After Rain"
  subtitle: null
design_tokens:
  palette: [warm ivory, compensated amber, cool slate, cobalt anchor]
  typography: "thin Helvetica caption"
  texture: []
  atmosphere: quiet urban
recoveries: [panter_mode, color_anchor]   # no riso_texture — photo-abstract-diptych is CLEAN
photo_policy:
  fidelity: required
  reference_image: uploaded
  source_region: upper
avoids: [glossy ad, cinematic lighting, photo redraw, mockup frame]
target:
  model: gpt-image
  prompt_only: false
```

# VisionSpec / EditorialSpec — Model-Agnostic Contract

Emitted by the Decision Engine **before** Model Adapter runs.
All adapters consume this schema. Do not embed model-specific syntax here.

Use `EditorialSpec` for backward compatibility. For general tasks, treat the same schema as `VisionSpec`.

## Schema

```yaml
spec_version: "1.1"

# --- From Intent Engine ---
intent:
  goal: string
  family: Visual Concept | Art Book | Event Campaign | Gallery | Zine | Magazine | Branding | Product / Object | Digital Product | Social | Interface Asset
  purpose: poster | editorial | social | portfolio | campaign | presentation | brand
  audience: general | student | professional | luxury | youth | internal
  emotion: string          # calm, nostalgic, futuristic, dramatic, confident, quiet
  platform: instagram | website | print | presentation | app
  subject: string          # what is depicted
  aspect_ratio: string     # requested or derived from platform
  inferred: [string]       # intent dimensions the engine filled in rather than read
  allowed_outputs: [string]   # layout ids, same vocabulary as direction.layout
  blocked_outputs: [string]   # layout ids, same vocabulary as direction.layout
  user_style_override: string | null   # "swiss", "kinfolk", … — makes Art Direction auto-commit
  language: en | zh  # user-facing summary language, not prompt language
  source_type: photo | theme | brand | product | interface | mixed

# --- From Art Direction Engine (prompts/art-direction.md) ---
art_direction:
  id: string               # A | B | C
  name: string             # "Swiss Editorial"
  thesis: string           # one sentence: what this direction argues the image is about
  fit_score: 0.0-1.0
  selection_mode: auto | user
  runner_up: string | null # candidate id — switchable without re-analysis

# --- Series / memory linkage (null for a standalone run) ---
preset: string | null      # preset id from presets/registry.md; resolves to a VisualMemory
style_gate:                # set by prompts/style-gate.md
  outcome: commit | offer  # THE ONLY VALUE Art Direction reads
  reason: preset_chosen | freeform | deferred | named_in_request | memory_active |
          series | unattended | direction_committed | proposals_requested | too_few_presets
  preset: string | null
  description: string | null   # the user's own words, verbatim, when reason is freeform
series_id: string | null   # set by prompts/series.md when the request is a set
memory_id: string | null   # set by prompts/visual-memory.md; locked fields become hard constraints

# --- From Visual Analyzer (null if theme-only / prompt-only) ---
image_report:
  subject: person | architecture | landscape | street | food | object | product | interface | brand | abstract | null
  clarity: 0-100
  contrast: 0-100
  saturation: 0-100
  negative_space: 0-100
  composition: excellent | good | weak
  geometry: low | medium | high      # read by prompts/visual-language.md
  lighting: flat | directional | dramatic
  emotion: string
  editorial_score: 0-100
  flags: [string]  # e.g. low_saturation, panter_mode
  spatial:         # what abstraction must preserve — see prompts/analyzer.md
    arrangement: string
    overlaps: [string]
    relative_scale: string
    ground_plane: string
    light_direction: string
    element_kinds: {}

# --- From Visual Language + Planner ---
direction:
  visual_language: string
  layout: poster | magazine-cover | gallery-print | zine | editorial-spread | campaign-poster | photo-abstract-diptych | brand-key-visual | product-editorial | website-hero | social-asset | moodboard | interface-asset | presentation-deck
  style: string  # swiss, kinfolk, muji, ...
  editorial_mode: premium | standard | compensation | reconstruction
  abstraction_level: relationship-first | identity-cue | full-abstract   # HOW ABSTRACT
  render_mode: photographic | photo-plus-graphic | graphic | painterly | mixed   # WHAT MEDIUM — required, no default
  composition:
    photo_ratio: 0.0-1.0      # 0 if no photo
    abstract_ratio: 0.0-1.0
    type_ratio: 0.0-1.0
    whitespace_ratio: 0.0-1.0
    form_types: integer | "3-5"   # KINDS of form, not instances — excluded from the ratio sum
  aspect_ratio: "3:4" | "2:3" | "3:5" | "1:1" | "4:5" | "9:16" | "16:9" | "4:3" | "A4"
  title: string | null        # 2-5 word English title
  subtitle: string | null
  production_context: print | social | web | interface | prompt_only

# --- From Style + Assets modules ---
design_tokens:
  ground: paper-light | neutral-gray | dark | saturated | full-bleed-photo | duotone
                              # the canvas field itself — required, no default. See assets/ground.md
  palette: [string]           # named hues, not hex-only
  typography: string          # e.g. "thin serif, caption scale"
  texture_tier: FLAT | SURFACE | PRINT   # resolved tier — the lockable field. See assets/texture.md
  texture: [string]           # PRINT tokens (riso_halftone, xerox, scan_noise) only when direction.layout == zine
                              # CLEAN layouts: SURFACE tokens only (cotton_paper, natural_material, matte_board) or []
                              # FLAT targets: always []. See assets/texture.md
  atmosphere: string          # e.g. quiet cinematic
  brand_cues: [string]         # null/empty unless user provided brand or product cues

# --- From Recovery Engine ---
recoveries: [string]          # module IDs: panter_mode, silhouette_boost, ...
                              # riso_texture is valid only when direction.layout == zine

# --- Photo fidelity ---
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

- `design_tokens.ground` and `direction.render_mode` are **required with no default**. An unset value is a rejection, not a fallback — this is what stops every run drifting to ivory paper
- `direction.render_mode` and `direction.abstraction_level` are independent: `render_mode` is the medium, `abstraction_level` is how far from the source it travels. A `photographic` image can still be `full-abstract`
- If `preset` is set, it resolves to a VisualMemory with `source: preset` and the same lock rules apply ([../presets/registry.md](../presets/registry.md))
- `design_tokens.ground: full-bleed-photo` requires either `photo_policy.reference_image: uploaded` (a supplied photograph) or `direction.render_mode: photographic` (the model generates one). One of the two, never neither
- `design_tokens.ground` values `saturated` and `duotone` require the ground hue to be a member of `design_tokens.palette` ([../assets/ground.md](../assets/ground.md))
- `style_gate.outcome` must be set before Art Direction runs. `commit` forbids offering candidates; `offer` requires it. There is no third behaviour
- `style_gate.reason: preset_chosen` requires `style_gate.preset` and `preset` to be the same non-null id
- `style_gate.reason: freeform` requires a non-null `style_gate.description`, and `art_direction.runner_up` must be null — a description commits to one reading
- On `reason: freeform`, `design_tokens.ground` and `direction.render_mode` must each trace to a cue in the description or to the single permitted clarifying question ([../prompts/style-brief.md](../prompts/style-brief.md)). Neither may be inferred silently
- If `direction.abstraction_level` is `relationship-first`, `image_report.spatial` must be non-null and the compiled prompt must restate `arrangement`, `ground_plane`, and `light_direction`. Relationships cannot be preserved if they were never recorded
- `direction.composition.form_types` counts **kinds** of form, never instances. It may reduce the vocabulary; it may not delete an element the arrangement depends on ([../assets/scene-construction.md](../assets/scene-construction.md))
- `art_direction.thesis` is required and must be traceable to a photo fact, brand cue, or stated goal — a direction with no argument is decoration
- `direction.layout` must appear in `intent.allowed_outputs` and must not appear in `intent.blocked_outputs`
- If `memory_id` is set, every field locked by that memory must match it exactly ([visual-memory.schema.md](visual-memory.schema.md)). A mismatch is a rejection, not a warning
- If `memory_id` is set, `art_direction.selection_mode` must be `auto` — memory runs do not re-pick a direction
- If `series_id` is set, `design_tokens` and `direction.style` must be identical across every spec sharing that id
- If `direction.layout = presentation-deck`, `design_tokens.texture_tier` must be `FLAT` or `SURFACE` — never `PRINT`
- `design_tokens.texture_tier` must agree with `design_tokens.texture`: `FLAT` → `[]`, `SURFACE` → SURFACE tokens only, `PRINT` → `direction.layout = zine`
- `intent.platform` and `direction.aspect_ratio` must be compatible (`instagram` → 1:1 / 4:5 / 9:16; `presentation` → 16:9 / 4:3; `print` → 2:3 / 3:4 / A4; `website` → 16:9; `app` → 1:1). In a series, `intent.platform` names the hero's platform and each output is checked against its own `direction.production_context` instead
- `intent.allowed_outputs`, `intent.blocked_outputs`, and `direction.layout` all use the same kebab-case layout vocabulary
- `recoveries` must match `image_report.flags` — no orphan recoveries
- If `photo_policy.fidelity = required`, `avoids` must include `photo redraw`
- If `layout = photo-abstract-diptych`, `photo_policy.fidelity` must be `required`
- If `editorial_mode = reconstruction`, `abstraction_level` should be `full-abstract` — except on `layout: photo-abstract-diptych`, where `full-abstract` is invalid and `relationship-first` is the floor ([../layouts/photo-abstract-diptych.md](../layouts/photo-abstract-diptych.md))
- If `production_context = web`, preserve copy-safe negative space and avoid fake UI unless requested
- If `production_context = interface`, avoid fake text, fake controls, and unreadable UI details
- Sum of composition **ratios** ≈ 1.0 (±0.1). `form_types` is a count, not a ratio, and is excluded from the sum

## Example (minimal)

```yaml
spec_version: "1.1"
intent:
  goal: "editorial poster from gray street photo"
  family: Gallery
  purpose: editorial
  audience: general
  emotion: quiet
  platform: print
  subject: "rain-wet street corner"
  aspect_ratio: "3:4"
  inferred: [audience, aspect_ratio]
  allowed_outputs: [poster, photo-abstract-diptych]
  blocked_outputs: [campaign-poster, interface-asset]
  language: zh
art_direction:
  id: A
  name: "Swiss Architectural"
  thesis: "The street is a grid interrupted by weather."
  fit_score: 0.87
  selection_mode: auto
  runner_up: B
preset: null
series_id: null
memory_id: null
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
  render_mode: photo-plus-graphic
  composition:
    photo_ratio: 0.65
    abstract_ratio: 0.25
    type_ratio: 0.05
    whitespace_ratio: 0.05
  aspect_ratio: "3:4"
  title: "After Rain"
  subtitle: null
design_tokens:
  ground: paper-light
  palette: [warm ivory, compensated amber, cool slate, cobalt anchor]
  typography: "thin Helvetica caption"
  texture_tier: FLAT
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

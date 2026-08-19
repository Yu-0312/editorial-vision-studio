# Intent Engine

Resolve user goal **before** pixel analysis or prompt compilation.

## Output Families

Families match the `intent.family` enum in [../spec/editorial-spec.schema.md](../spec/editorial-spec.schema.md) exactly. Allowed and blocked entries are **layout ids** from the same schema, not prose.

| Family | Allowed outputs | Blocked |
|--------|-----------------|---------|
| Visual Concept | moodboard, poster, editorial-spread | campaign-poster, interface-asset |
| Art Book | magazine-cover, gallery-print, poster | campaign-poster, social-asset |
| Event Campaign | campaign-poster, brand-key-visual, social-asset | gallery-print, zine |
| Gallery | gallery-print, photo-abstract-diptych, poster | campaign-poster, interface-asset |
| Zine | zine, poster, editorial-spread | brand-key-visual, website-hero |
| Magazine | magazine-cover, editorial-spread, poster | interface-asset, zine |
| Branding | brand-key-visual, product-editorial, social-asset, campaign-poster, website-hero | zine, photo-abstract-diptych |
| Product / Object | product-editorial, brand-key-visual, editorial-spread | zine, campaign-poster |
| Digital Product | website-hero, interface-asset, social-asset | gallery-print, zine |
| Social | social-asset, poster, magazine-cover | gallery-print, photo-abstract-diptych |
| Interface Asset | interface-asset, website-hero, moodboard | zine, gallery-print |

`presentation-deck` is allowed for any family when `intent.purpose = presentation`, and appears in no family's blocked list.

## Intent Dimensions

Format alone underdetermines the image. Resolve **six** dimensions before anything downstream runs — each one constrains a different layer.

| Dimension | Values | Constrains |
|-----------|--------|------------|
| `subject` | what is depicted | Analyzer, Planner |
| `purpose` | poster, editorial, social, portfolio, campaign, presentation, brand | allowed / blocked layouts |
| `audience` | general, student, professional, luxury, youth, internal | Art Direction fit score |
| `emotion` | calm, nostalgic, futuristic, dramatic, confident, quiet | Visual Language row |
| `platform` | instagram, website, print, presentation, app | copy-safe rules, texture tier |
| `aspect_ratio` | 3:4, 2:3, 3:5, 1:1, 4:5, 9:16, 16:9, 4:3, A4 | Planner ratios, crop survival |

Intent does **not** set `ground` or `render_mode` — those belong to Art Direction or a preset. But `platform` and `emotion` constrain them: a story frame rarely wants a paper ground, and `dramatic` rarely wants `paper-light`.

Infer, do not interrogate. Ask the user only when a dimension is both unresolvable and load-bearing — an unstated `platform` when the brief could be print or story is worth one question; an unstated `audience` on a personal zine is not.

Record which values were inferred, so the Art Direction fit score knows what it is grading against.

## Resolution Steps

1. Extract explicit format from user request
2. If ambiguous, infer from subject + context (event name → Campaign; "封面" → Magazine/Art Book)
3. Resolve the six intent dimensions above; mark each stated or inferred
4. Set `allowed_outputs[]` and `blocked_outputs[]` using the kebab-case layout vocabulary from [../spec/editorial-spec.schema.md](../spec/editorial-spec.schema.md)
5. Derive `aspect_ratio` from `platform` when not stated: `instagram` → 4:5 (9:16 for a story or reel), `presentation` → 16:9 (4:3 only for a legacy projector), `print` → 2:3, 3:4, or A4, `website` → 16:9, `app` → 1:1
6. Pass intent object to the [Style Gate](style-gate.md), then to Art Direction and Planner as a hard constraint

## Intent Object Schema

```yaml
intent:
  goal: "TEDx key visual"
  family: Event Campaign
  subject: "speaker silhouette on stage"
  purpose: campaign
  audience: professional
  emotion: confident
  platform: print
  aspect_ratio: "2:3"
  allowed_outputs: [campaign-poster, brand-key-visual, social-asset]
  blocked_outputs: [gallery-print, zine]
  user_style_override: null  # or "swiss", "kinfolk", etc.
  language: en  # title/copy language
  source_type: photo | theme | brand | product | interface | mixed
  inferred: [audience, aspect_ratio]   # dimensions the engine filled in

# scope ids sit at spec root, not inside intent — see ../spec/editorial-spec.schema.md
preset: null                           # string → presets/registry.md
series_id: null                        # string → prompts/series.md
memory_id: null                        # string → prompts/visual-memory.md
```

## Series and Continuation Detection

| Signal | Set |
|--------|-----|
| "一套", "全尺寸", "N 頁", "carousel", "all platform sizes" | `series_id` → [series.md](series.md) |
| "同一套視覺", "延續上一張", "same look as before", brand assets supplied | `memory_id` → [visual-memory.md](visual-memory.md) |
| `preset: <id>`, or a named look from [../presets/registry.md](../presets/registry.md) | `preset` |
| `style: swiss`, or any named style | `user_style_override` |
| A free-text description of a look (「暗一點、像雜誌」) | `style_gate.description` → [style-brief.md](style-brief.md) |
| None of the above | neither; single run |

Then hand off to the [Style Gate](style-gate.md), which decides from these values whether to show a menu. `series_id`, `memory_id`, `preset`, `user_style_override`, and a `style_gate.description` already present in the brief each make it skip the menu; none of them set is what makes the menu appear.

Detect this at Intent, not later. A series discovered at the Compiler has already wasted a full pipeline pass.

## Examples

**"幫我做一本藝術攝影集封面"**
→ family: Art Book → allowed: `magazine-cover`, `gallery-print`, `poster`

**"TEDx 主視覺"**
→ family: Event Campaign → allowed: `campaign-poster`, `brand-key-visual`, `social-asset`

**"幫我做一張 skincare brand launch hero image"**
→ family: Branding → allowed: `brand-key-visual`, `product-editorial`, `social-asset`, `campaign-poster`, `website-hero`

**"SaaS landing page hero art，安靜、可信、不是插畫感"**
→ family: Digital Product → allowed: `website-hero`, `social-asset`, `interface-asset`

**"做一組 app empty state 圖像方向"**
→ family: Interface Asset, `series_id` set → allowed: `interface-asset`, `moodboard`

**"一套 10 頁簡報"**
→ purpose: presentation, `series_id` set → allowed: `presentation-deck`

**"把這張灰圖做成編輯作品"**
→ family: Gallery (default) → Planner decides layout from Image Report

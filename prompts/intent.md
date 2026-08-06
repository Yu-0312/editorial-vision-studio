# Intent Engine

Resolve user goal **before** pixel analysis or prompt compilation.

## Output Families

| Family | Typical outputs | Block |
|--------|-----------------|-------|
| Visual Concept | concept image, moodboard, style frame, material board | production-ready layout claims |
| Commercial Editorial | campaign poster, key visual, social banner | gallery-only layouts |
| Branding | brand poster, brand key visual, lookbook spread, packaging mock, launch visual | zine diary layouts unless requested |
| Product / Object | product editorial, still life, hero image, catalog visual | human documentary layouts |
| Digital Product | website hero, app-store visual, onboarding art, landing-page image | print-only proportions |
| Gallery | gallery print, full bleed, diptych | heavy typography campaigns |
| Social Presentation | square/vertical social, story frame, carousel cover | book jacket proportions |
| Campaign | event poster, stage screen, multi-format KV | single-specimen zine |
| Art Book | cover, jacket, chapter opener | commercial CTA layouts |
| Magazine | cover, spread, feature opener | poster-only when cover requested |
| Zine | page, spread, cover | glossy ad mockups |
| Interface Asset | empty-state image, illustration direction, app background | photography fidelity constraints unless image provided |

## Resolution Steps

1. Extract explicit format from user request
2. If ambiguous, infer from subject + context (event name → Campaign; "封面" → Magazine/Art Book)
3. Set `allowed_layouts[]` and `blocked_layouts[]`
4. Pass intent object to Planner as hard constraint

## Intent Object Schema

```yaml
intent:
  goal: "TEDx key visual"
  family: Event Campaign
  allowed_outputs: [campaign_poster, key_visual, social_banner]
  blocked_outputs: [gallery_print, zine_page]
  user_style_override: null  # or "swiss", "kinfolk", etc.
  language: en  # title/copy language
  source_type: photo | theme | brand | product | interface | mixed
```

## Examples

**"幫我做一本藝術攝影集封面"**
→ family: Art Book → allowed: gallery cover, magazine cover, minimal cover, book jacket

**"TEDx 主視覺"**
→ family: Event Campaign → allowed: campaign poster, KV, social banner, stage screen

**"幫我做一張 skincare brand launch hero image"**
→ family: Branding or Product / Object → allowed: brand key visual, product editorial, website hero

**"SaaS landing page hero art，安靜、可信、不是插畫感"**
→ family: Digital Product → allowed: website hero, social banner, style frame

**"做一組 app empty state 圖像方向"**
→ family: Interface Asset → allowed: empty-state image, illustration direction, app background

**"把這張灰圖做成編輯作品"**
→ family: Gallery (default) → Planner decides layout from Image Report

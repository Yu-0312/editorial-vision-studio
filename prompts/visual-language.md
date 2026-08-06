# Visual Language Engine

Visual Language is **not** Style. It is the emotional/architectural intent that **derives** style, palette, layout, typography, texture, and lighting.

## Derivation Chain

```
User Intent + Image Report
    ↓
Visual Language (pick one primary)
    ↓
Style + Palette + Layout + Typography + Texture + Lighting
    ↓
Planner output
```

Never pick Style first unless user explicitly overrides (`style: kinfolk`).

## Language Catalog

| Visual Language | Signals in Image Report | Style | Layout bias | Palette | Typography | Texture | Lighting |
|-----------------|-------------------------|-------|-------------|---------|------------|---------|----------|
| Museum | quiet, high negative space, fine detail | Swiss | Gallery / Cover | Ivory, warm gray | Fine serif | Cotton paper | Soft side |
| Quiet Human | portrait, emotion, soft light | Kinfolk | Magazine Cover | Cream, sage, beige | Light serif | Cotton paper (SURFACE) | Diffuse |
| Architectural | geometry, lines, structure | Swiss / Brutalist | Poster | Neutral + accent | Grotesk | Matte concrete | Directional |
| Domestic Intimacy | interiors, objects, warmth | Apartamento | Spread | Terracotta, wood | Small serif | Lived-in | Window light |
| Indie Memory | nostalgia, sparse, diary | Zine | Zine | Paper + one chroma | Typewriter | Riso/xerox | Flat scan |
| Fashion Edge | portrait, drama, confidence | Purple | Magazine Cover | Deep neutral + vivid | Bold serif | Clean matte | Cinematic quiet |
| Urban Documentary | street, energy, story | POPEYE | Zine / Poster | Primary accents | Mixed sans | Print wear | Available light |
| Product Stillness | object, minimal, calm | MUJI / Wallpaper* / COS | Spread / Poster | White, gray, wood | Caption only | Natural material | Soft even |
| Campaign Bold | event, message, impact | Brutalist / Swiss | Campaign Poster | High contrast | Display grotesk | Flat ink | Hard graphic |
| Cosmopolitan Reportage | travel, city, design facts | Monocle | Spread / Poster | Navy, warm white | Sans hierarchy | Clean matte | Even studio |
| Brand System Calm | brand/product launch, premium restraint | COS / Swiss / Wallpaper* | Brand Key Visual | Off-white, black, one product hue | Small sans | Material surface | Soft controlled |
| Digital Trust | SaaS, app, technical product, calm clarity | Swiss / MUJI / COS | Website Hero / Interface Asset | White, cool gray, single signal color | UI-safe sans | Flat matte (FLAT) | Even luminous |
| Social Impact | social campaign, launch post, mobile crop | Swiss / Brutalist / POPEYE | Social Asset | High contrast + one anchor | Bold but sparse | Flat ink | Graphic |
| Concept Atmosphere | theme-only prompt, moodboard, style frame | User override / Generic | Moodboard / Zine | Derived from theme | Minimal labels | Material mix | Mood-led |

## Selection Rules

1. Match Image Report `emotion` + `subject` + `geometry` to closest language row
2. If Intent family = Event Campaign → prefer Campaign Bold unless photo is quiet landscape
3. If Intent family = Branding or Product / Object → prefer Brand System Calm or Product Stillness
4. If Intent family = Digital Product or Interface Asset → prefer Digital Trust
5. If no image is provided → derive from user adjectives, domain, audience, and output format
6. If Editorial Score <50 → shift toward Indie Memory or Architectural (abstraction-friendly)
7. If Panter flag → Indie Memory or Architectural (anchor-friendly). Texture still follows [assets/texture.md](../assets/texture.md): PRINT only if the resulting layout is `zine`

## Output Field

```yaml
visual_language: Quiet Human
derivation:
  style: kinfolk
  layout: magazine-cover
  palette: cream_sage_beige
  typography: light_serif_small
  texture: cotton_paper
  lighting: soft_diffuse
rationale: "Portrait with 72% negative space and quiet emotion"
```

# Visual Planner

Based on Intent + Image Report + Visual Language, select visual direction. **Do not write the final prompt.**

Run [visual-language.md](visual-language.md) first unless user set explicit `style:`.

## Subject → Layout Matrix

| Subject + Condition | Layout | Default Style |
|---------------------|--------|---------------|
| Portrait, negative space >50% | Magazine Cover | Kinfolk / Purple |
| Portrait, tight crop | Editorial Portrait | Apartamento |
| Architecture, strong geometry | Swiss Poster | Swiss |
| Landscape, quiet | Gallery Print | MUJI / Gallery |
| Street, human story | Documentary Zine | POPEYE |
| Food / product, minimal | Product Editorial | Wallpaper* |
| Product + launch context | Brand Key Visual | COS / Wallpaper* / Swiss |
| Brand system / campaign | Campaign Poster or Social Set | Swiss / Brutalist / Monocle |
| Digital product / SaaS | Website Hero or Interface Asset | Swiss / MUJI / COS |
| Theme-only mood | Concept Board or Zine | Flux texture / Ideogram type |
| High abstraction potential | Editorial Poster + abstract panel | Swiss / Brutalist |
| User: photo + abstraction diptych | Photo-Abstract Diptych | photo-abstract-editorial rules |
| Intent: Event Campaign | Campaign Poster | Swiss / Brutalist |

## Planner Output Schema

```yaml
visual_language: Quiet Human
layout: magazine-cover
style: kinfolk
typography: minimal serif, small scale
color_strategy: warm neutral extracted + one sage accent
abstraction_level: relationship-first  # or identity-cue / full-abstract
composition_strategy: upper photo 65%, lower abstract 25%, margin 10%
recovery_plan: [panter_mode, color_anchor]
editorial_mode: compensation  # premium | standard | compensation | reconstruction
production_context: print | social | web | interface | prompt_only
```

## Layout DNA Parameters

Load from [layouts/](../layouts/) and set ratios:

- Magazine Cover: image 65%, type 20%, whitespace 15%
- Poster: image 70%, geometry 20%, text 10%
- Gallery: image 92%, type 3%, whitespace 5%
- Website Hero: focal image 55%, negative/copy-safe area 35%, texture/atmosphere 10%
- Social Asset: focal image 60%, type/safe area 25%, brand cue 15%
- Interface Asset: focal object 45%, whitespace 45%, system color cue 10%

Adapt ratios to photo aspect — do not force mechanical 50/50 split.

For web/interface outputs, preserve copy-safe space and avoid generating fake UI unless the user explicitly asks for UI content.

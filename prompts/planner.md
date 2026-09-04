# Visual Planner

Based on Intent + Image Report + the **selected Art Direction**, set the execution parameters. **Do not write the final prompt.**

Run [visual-language.md](visual-language.md) then [art-direction.md](art-direction.md) first. Art Direction always runs — an explicit `style:` or an active memory makes it auto-commit, it does not skip the layer, because the spec requires an `art_direction` block on every run.

The Planner works **inside** the committed direction. It sets ratios, typography scale, abstraction level, and recovery plan — it does not re-pick style, palette, or visual language. If the direction itself is wrong, that is an Art Direction escalation ([iteration.md](iteration.md)), not a Planner override.

When `memory_id` or `preset` is set, fields locked by that memory are read-only. The Planner sets `free` fields only — see [visual-memory.md](visual-memory.md) and [../presets/registry.md](../presets/registry.md).

`design_tokens.ground` and `direction.render_mode` must both be set before the Planner hands off. They have no defaults; an unset value is a rejection, not ivory paper.

## Spatial Plan

`direction.spatial_plan` is required on every run. Where it comes from depends on whether a photograph was analyzed:

| Run type | Source |
|----------|--------|
| Photo | Copy `image_report.spatial` verbatim. The Planner does not re-observe the image — that is the Analyzer's job and re-deciding it here is a layer violation |
| Theme-only | The Planner authors it from `intent.subject`. Name one projection, one shared ground plane, one light direction, and the arrangement the subject implies |

**Locked spatial fields win.** When an active memory or preset locks `spatial_plan` fields, apply them over the result field by field — a locked `projection: flat-elevation` overrides a photographed deep perspective, and that is the point of the lock. The lock is an authored decision, not a re-observation. Unlocked fields keep their source: the report on a photo run, Planner authoring on a theme-only one. Locked keys must match the spec's own names ([../spec/visual-memory.schema.md](../spec/visual-memory.schema.md)).

Theme-only is not an excuse to leave it empty. A brief that says "an old Kyoto tea room" already implies a viewpoint, a floor, and a window the light comes through; deciding those here is what stops the model inventing three of each. Say the projection you want — «avoid perspective» is not a projection ([../assets/scene-construction.md](../assets/scene-construction.md)).

## Subject → Layout Matrix

| Subject + Condition | Layout | Default Style |
|---------------------|--------|---------------|
| Portrait, negative space >50% | Magazine Cover | Kinfolk / Purple |
| Portrait, tight crop | Editorial Spread | Apartamento |
| Architecture, strong geometry | Swiss Poster | Swiss |
| Landscape, quiet | Gallery Print | MUJI / Gallery |
| Street, human story | Documentary Zine | POPEYE |
| Food / product, minimal | Product Editorial | Wallpaper* |
| Product + launch context | Brand Key Visual | COS / Wallpaper* / Swiss |
| Brand system / campaign | Campaign Poster or Social Set | Swiss / Brutalist / Monocle |
| Digital product / SaaS | Website Hero or Interface Asset | Swiss / MUJI / COS |
| Multi-page slide set | Presentation Deck | Swiss / MUJI / Monocle |
| Theme-only mood | Concept Board or Zine | Flux texture / Ideogram type |
| High abstraction potential | Editorial Poster + abstract panel | Swiss / Brutalist |
| User: photo + abstraction diptych | Photo-Abstract Diptych | [layouts/photo-abstract-diptych.md](../layouts/photo-abstract-diptych.md) |
| Intent: Event Campaign | Campaign Poster | Swiss / Brutalist |

## Planner Output Schema

```yaml
visual_language: Quiet Human
layout: magazine-cover
style: kinfolk
typography: thin serif, caption scale   # never a banned adjective — see compiler.md
color_strategy: warm neutral extracted + one sage accent
abstraction_level: relationship-first  # or identity-cue / full-abstract
spatial_plan:
  projection: "flat elevation, held across every object"
  ground_plane: "one shared floor, eye level just above the table top"
  light_direction: "upper left, flat contact shadow under every grounded object"
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
- Presentation Deck: per page role — see [layouts/presentation-deck.md](../layouts/presentation-deck.md)

Adapt ratios to photo aspect — do not force mechanical 50/50 split.

For web/interface outputs, preserve copy-safe space and avoid generating fake UI unless the user explicitly asks for UI content.

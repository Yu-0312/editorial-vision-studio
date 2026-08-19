# Preset Registry

A **preset** is a shipped [VisualMemory](../spec/visual-memory.schema.md) with `source: preset` — a locked partial spec authored in this repo rather than established from a run.

Presets reuse the entire memory mechanism. Nothing new enforces them:

| Layer | Behaviour on a preset run |
|-------|---------------------------|
| [../prompts/art-direction.md](../prompts/art-direction.md) | Auto-commits to the preset DNA; offers no candidates |
| [../prompts/planner.md](../prompts/planner.md) | Sets `free` fields only |
| [../prompts/compiler.md](../prompts/compiler.md) | Compiles locked fields verbatim |
| [../prompts/reviewer.md](../prompts/reviewer.md) | Rejects any request that contradicts a lock |

## Available Presets

| Preset | Ground | Render mode | Intended layouts | Use when |
|--------|--------|-------------|------------------|----------|
| [ivory-postcard](ivory-postcard.md) | `paper-light` | `painterly` | poster, gallery-print, social-asset, moodboard | Quiet minimal editorial illustration from a photo; the repo's original hardcoded look |
| [vintage-travel-poster](vintage-travel-poster.md) | `saturated` | `graphic` | poster, campaign-poster, social-asset, magazine-cover | Period travel/exhibition poster; flat colour fields, bold type, no photographic surface |
| [papercraft-diorama-postcard](papercraft-diorama-postcard.md) | `full-bleed-photo` | `photographic` | social-asset | Photoreal papercraft diorama emerging from a postcard; social-first, 1:1 |

`intended_layouts` is what the [Style Gate](../prompts/style-gate.md) filters on. No preset currently covers `website-hero`, `interface-asset`, `presentation-deck`, `zine`, `editorial-spread`, `brand-key-visual`, `product-editorial`, or `photo-abstract-diptych` — briefs in those families skip the menu and go to Art Direction candidates.

Three presets, three different grounds. That is deliberate — see [../prompts/art-direction.md](../prompts/art-direction.md), where candidate directions must differ on `ground`.

## Invoking

The primary path is the [Style Gate](../prompts/style-gate.md): at the start of a run the user is shown these presets as a numbered menu plus 「讓 AI 提案」, and picks one. The chosen preset becomes the session's active Visual Memory, so the question is asked once, not once per image.

The shortcut skips the menu — for automation, series work, and users who already know what they want:

```
preset: ivory-postcard
```

Either way, a preset **removes the direction question entirely**. Intent and Analyzer still run; the preset supplies everything Art Direction would have decided. Art Direction must not ask again — that would be the user choosing a look twice.

## Preset vs. Series Memory

Same shape, different provenance and one different rule:

| | Preset | Series memory |
|---|--------|---------------|
| Provenance | Authored in this repo | Established from a passing run |
| Scope | Any project | One project |
| `composition` / `aspect_ratio` / `layout` lockable | **Yes** | No |
| Fork required to change | No — edit in place and bump `version`, unless the change would break a series already built on it | Yes |

A preset is avowedly a template, so it may lock composition. A series memory may not: locking composition across a series produces four images with the subject in the same corner, which reads as a template rather than a system.

## Adding a Preset

1. Copy [_template.md](_template.md) → `presets/your-preset.md`
2. Fill every locked field — a preset with an unset `ground`, `render_mode`, or `intended_layouts` is not a preset
3. Account for every field in the schema's canonical `free` list: each goes in `locked` or `free`. If `layout` stays free, list `blocked_layouts`
4. Set `version`, and bump it whenever you edit a locked value
5. Register it in the table above
6. No changes to Analyzer, Planner, Recovery, adapters, or style files

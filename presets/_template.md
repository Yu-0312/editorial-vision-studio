# {Preset Name} — Preset

One sentence: what this preset argues an image should look like, and when to reach for it.

## Locked DNA

```yaml
memory_version: "1.0"
memory_id: your-preset
version: "1.0"
established_from: preset
source: preset

locked:
  ground: paper-light | neutral-gray | dark | saturated | full-bleed-photo | duotone
  render_mode: photographic | photo-plus-graphic | graphic | painterly | mixed
  style: swiss | kinfolk | ...          # must have a styles/ file
  visual_language: ...                  # must be a row in prompts/visual-language.md
  intended_layouts: [...]               # what the Style Gate filters on — state the fit positively
  palette: [...]                        # named hues; hex only for an exact brand/ground value
  typography: "..."
  texture_tier: FLAT | SURFACE | PRINT
  atmosphere: "..."
  abstraction_level: relationship-first | identity-cue | full-abstract
  composition: {...}                    # presets may lock composition, aspect_ratio, and layout;
                                        # memories established from a run may not

blocked_layouts: [zine]                 # required whenever `layout` stays free

# every field in the schema's canonical free list must be in `locked` or here
free: [layout, aspect_ratio, title, subtitle, recoveries]

hard_constraints:
  - "..."
```

## Avoids

Concrete, visible things this preset must never produce. Not adjectives.

## Compiler anchor

```
One paragraph of renderable constraints, in the order from prompts/compiler.md:
canvas and surface, attention geometry and negative-space budget, one primary
image anchor and its treatment, typography or copy-safe behaviour, palette,
texture, lighting, and explicit avoids.
```

## Notes

Where this preset came from, and anything a future editor would otherwise get wrong.

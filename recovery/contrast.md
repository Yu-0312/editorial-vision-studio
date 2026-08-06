# Recovery: Contrast / Panter Mode

**Trigger:** saturation <30% OR contrast <40, muddy histogram

Panter is a **colour** compensation. Texture is not part of it — texture is gated separately by [../assets/texture.md](../assets/texture.md) → Texture Permission.

## Actions

1. Discard dull gray extractions as panel colors
2. Find warmest region → boost saturation to 75%
3. Find coolest region → boost saturation to 70%
4. Use as contrasting abstract panel hues
5. Add high-sat anchor block (~8% panel, brightness +40 vs average)
6. Widen tonal separation and mark scale to carry the remaining atmosphere
7. Texture: only if the layout is PRINT (`zine`) may riso halftone be added, and then to abstract marks only

## Compiler clauses

CLEAN layouts (default):

```
Abstract panel uses compensated warm-cool contrast hues, not extracted muddy grays;
small high-saturation vermilion or cobalt anchor at golden-ratio corner;
flat uniform panel ground with no grain, noise, or paper texture.
```

PRINT layouts (`zine`):

```
Abstract panel uses compensated warm-cool contrast hues, not extracted muddy grays;
small high-saturation vermilion or cobalt anchor at golden-ratio corner;
riso halftone within the abstract marks only, never over the panel ground.
```

## Never

Use original dirty grays as dominant panel colors without compensation.

Add grain, noise, or paper texture to a CLEAN layout — including the ivory panel of a `photo-abstract-diptych`. Use contrast, spacing, and mark scale for atmosphere instead.

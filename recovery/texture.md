# Recovery: Texture

**Trigger:** flat texture, lack of material depth

**Gate:** check [../assets/texture.md](../assets/texture.md) → Texture Permission first. This module is a no-op on CLEAN layouts.

## Actions — PRINT layouts (`zine`)

- Fine paper grain on ground
- Riso/xerox/halftone on abstract region
- Scan noise for zine layouts

## Actions — CLEAN layouts (all others)

Do **not** add grain, noise, halftone, or paper stain. At most name the substrate's own SURFACE character when the style DNA rates Texture ★★★+ (cotton paper, natural fibre, matte board). Otherwise recover material depth through:

- Wider tonal separation between ground and marks
- Larger interval / negative-space contrast
- Mark scale and edge-quality variation (hard block vs tapered block)

## Actions — FLAT targets

`photo-abstract-diptych` panel ground, `interface-asset`, `website-hero` copy-safe area, `product-editorial` background: no texture language at all, not even SURFACE.

## Compiler clauses

PRINT:

```
Fine matte paper grain on ivory ground,
Riso halftone texture on abstract panel region.
```

CLEAN:

```
Flat uniform ivory ground with no grain or paper texture;
material depth from tonal separation, interval, and mark scale.
```

## Never

Emit a texture clause when the selected layout is CLEAN, even if `flat_texture` was flagged.

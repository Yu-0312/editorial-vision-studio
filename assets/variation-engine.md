# Zine Variation Engine

From gc-minimal-zine-poster. Use when layout = zine and Planner needs visual diversity.

Before compiling, pick **one option per axis**. If recent outputs repeated the same combo, rotate.

## Layout Family

| ID | Description |
|----|-------------|
| center-fragment | Tiny central image, surrounding air |
| lower-left-float | Anchor lower-left, empty top |
| upper-right-block | Color/photo block upper-right, loose text |
| dual-panel | Two small overlapping panels, narrow gap |
| irregular-cutout | Torn organic shape carries image/type |
| type-led | Typography is main anchor |
| dot-orbit | Dots/letters orbit small subject |
| single-specimen | One isolated mark, almost no support |

## Image Anchor

tiny faded photo · torn-paper clipping · flat silhouette · solid color block · old printed illustration · object specimen · translucent geometric overlay · abstract texture window

## Typography Mode

fragmented floating letters · short phrase at image edge · archive microtext + date · diagonal scattered words · gray ghost text · headline-as-object letterpress · text inside color block · almost textless tiny caption

## Texture Mode

xerox softness · risograph grain · letterpress bleed · halftone degradation · film grain · scan noise · aged paper mottling · soft motion blur on selected text

## Mood Mode

quiet · summer · solitude · childhood · seaside · afternoon · rain · night walk

## Color Anchor Rule

- One fully saturated hue per image (cobalt default)
- Anchor: 0.8–2.5% canvas or 15–35% of cluster
- Never apply low saturation to the color anchor
- Batch rule: ≥60% images use colored subject/cutout/block

## Variation Output

```yaml
variation:
  layout_family: lower-left-float
  image_anchor: torn-paper clipping
  typography_mode: archive microtext
  texture_mode: risograph grain
  mood_mode: rain
  color_anchor: saturated cobalt risograph ink
```

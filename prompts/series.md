# Series Planner

One Visual System → **N deliverables**. Use when the request is a set, not an image.

Triggers: "一套 10 頁簡報", "campaign 全尺寸", "carousel 5 張", "封面＋內頁＋社群", "same visual, all platform sizes".

## Difference From Visual Memory

| | [visual-memory.md](visual-memory.md) | This file |
|---|---|---|
| Scope | Across sessions and requests | Within one request |
| Trigger | "now do the next one" | "do all of them" |
| Output | One image, consistent with prior runs | A planned set, emitted together |

A series **establishes** a memory. They compose: plan the set here, lock the DNA there.

## Pipeline Position

```
Intent → Style Gate → Analyzer → Visual Language → Art Direction
    ↓
Series Planner   ← you are here
    ↓
for each output: Planner → Compiler → Adapter → Reviewer → Generate
    ↓
Series QC (cross-image consistency)
```

Run Intent, the Style Gate, Analyzer, Visual Language, and Art Direction **once** for the whole series. Only the Planner and downstream run per output. Every output after the hero skips the gate with `reason: series`.

If a preset is active — named, or chosen at the gate — the series **adopts it as the system** rather than establishing a new memory. `memory_id` is the preset's id, `system` is copied from its locked DNA, and no new memory is emitted on pass. Only a series with no preset establishes one of its own.

## Series Spec

```yaml
series:
  series_id: tedx-2026-kv
  memory_id: tedx-2026          # emitted to visual-memory once the hero passes QC.
                                # When a preset is active this is the preset id instead,
                                # and nothing new is emitted.
  system:                        # locked across every output
    style: swiss
    visual_language: Campaign Bold
    palette: [off-white, ink, signal red]
    typography: "display grotesk, tight"
    texture_tier: FLAT
  outputs:
    - id: kv-print
      layout: campaign-poster
      aspect_ratio: "2:3"
      production_context: print
      role: hero
    - id: kv-social-square
      layout: social-asset
      aspect_ratio: "1:1"
      production_context: social
      role: derivative
    - id: kv-story
      layout: social-asset
      aspect_ratio: "9:16"
      production_context: social
      role: derivative
    - id: kv-stage
      layout: campaign-poster
      aspect_ratio: "16:9"
      production_context: web
      role: derivative
  consistency_target: 0.85
```

## Roles

| Role | Meaning | Freedom |
|------|---------|---------|
| `hero` | The image the system is derived from. Generate first. | Full Planner freedom within the direction |
| `derivative` | Re-crop / re-proportion of the hero idea | Layout ratios and copy only |
| `variant` | Same system, different subject (deck pages, carousel) | Subject and composition free; system locked |

Generate the `hero` first and pass QC before fanning out. A weak hero multiplies into N weak derivatives.

## Fan-Out Rules

- **Re-proportion, do not re-crop mentally.** Each aspect ratio gets its own Planner pass with that layout's DNA ratios. A 2:3 poster squeezed to 9:16 is not a story frame.
- **The anchor survives every crop.** If `color_anchor` is at 1.8% of the poster canvas, it must still read at thumbnail scale in the 1:1.
- **Copy scales per platform**, not proportionally. Story frames need larger type than print at the same visual weight.
- **Vary the subject in `variant` sets.** Ten deck pages with the same composition is a template. Rotate composition family per page while holding the system.

## Multi-Page Decks

For presentation sets, see [../layouts/presentation-deck.md](../layouts/presentation-deck.md). Assign each page a role before planning:

```
title → section → content ×N → data → closing
```

Do not run the full pipeline per page. Run it once for the deck system, then apply the page role.

## Series QC

After all outputs generate, run a **cross-image** pass in addition to per-image [evaluator.md](evaluator.md):

| Check | Pass criteria |
|-------|---------------|
| Palette drift | Every output draws from the same named palette; no new hues |
| Typographic identity | Same typeface family and hierarchy logic across outputs |
| Texture tier | Identical tier on every output — no mixed PRINT and FLAT |
| Anchor legibility | The color anchor reads at each output's smallest display size |
| Compositional variety | `variant` sets do not repeat the same composition family more than twice |
| Role fidelity | Derivatives are recognizably the same idea as the hero |

```yaml
series_qc:
  consistency: 0.91
  variety: 0.78
  outliers: [kv-story]
  note: "kv-story anchor drops below thumbnail legibility"
```

An outlier is fixed with [iteration.md](iteration.md) on that output alone. Never re-run the whole series for one bad frame.

## Output Contract

Emit one manifest per output plus one series header — see [../spec/visual-manifest.schema.md](../spec/visual-manifest.schema.md). Present to the user as a compact table (id, layout, ratio, status), not N full specs.

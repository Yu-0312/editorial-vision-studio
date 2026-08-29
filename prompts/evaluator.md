# Quality Evaluator

Run **after** image generation (or on prompt-only requests, evaluate prompt fidelity).

## Post-Generation Checklist

| Check | Pass criteria |
|-------|---------------|
| Photo fidelity | Source photo region unchanged when fidelity is required — no redraw, filter, extension |
| Visual traceability | Each abstract or decorative mark maps to a photo, product, brand, theme, or stated-goal fact |
| Style coherence | Single visual language, no mixed Swiss + Kinfolk |
| Typography restraint | No overcrowded text; respects style DNA caps |
| Recovery evidence | Panter anchor visible if flagged; riso visible only when layout is `zine` |
| Texture permission | CLEAN layouts show flat uniform grounds — zero grain, halftone, noise, or paper stain |
| Layout ratios | Within ±10% of Planner DNA |
| Avoid-list compliance | No glossy ad, mockup frame, cinematic HDR, neon |
| Production fit | Web/interface/social/product constraints are respected when relevant |

## Prompt-Only Evaluation

Score the compiled prompt. Used in two places: when the user skips generation, and as the scorer [optimizer.md](optimizer.md) reads before and after its ops — that layer applies rewrites but owns no rubric of its own, so this table is the single scale for prompt text.

| Dimension | Max | Criteria |
|-----------|-----|----------|
| Imageability | 25 | Every clause maps to visible pixels |
| Modularity | 20 | No analyzer logic leaked into compiler |
| Style fit | 20 | Matches Visual Language derivation |
| Recovery fit | 15 | Recoveries match Image Report flags |
| Hygiene | 20 | Zero banned adjectives |

This 100-point total uses the same grade bands as the Quality Score below. There is no quality vector without a generated image.

A low score here is a **wording** failure, not a decision failure: route it to [optimizer.md](optimizer.md), which rewrites expression within the existing spec. Only escalate to [iteration.md](iteration.md) when the Optimizer reports `rejected_to_compiler` — that is the signal the spec itself is missing a field.

## Quality Vector

A single grade cannot be acted on. Score **every dimension separately** — the lowest failing one is what [iteration.md](iteration.md) fixes.

| Dimension | Weight | Scores |
|-----------|--------|--------|
| `subject` | 0.15 | Subject reads immediately; silhouette and scale hold |
| `composition` | 0.15 | Planner ratios respected within ±10%; negative space preserved. **Spatial coherence**: one projection, one ground plane, one light direction, contact shadows present, and the source's arrangement, overlaps, and relative scale intact ([../assets/scene-construction.md](../assets/scene-construction.md)) |
| `focal_point` | 0.10 | One unambiguous entry point; anchor visible at thumbnail scale |
| `palette` | 0.10 | Palette count respected; contrast separation adequate; no drift hues |
| `typography` | 0.10 | Present when specified, absent when not; within style DNA caps; legible |
| `texture` | 0.10 | Tier matches [../assets/texture.md](../assets/texture.md) exactly; `ground` and `render_mode` are what the spec asked for, not what the model defaulted to. A `painterly` spec returned with uniform vector edges scores here, not as a near-miss |
| `style_coherence` | 0.10 | One dominant visual language; no mixed grammar |
| `photo_fidelity` | 0.10 | Source region unmodified when `fidelity: required` |
| `intent_fit` | 0.05 | Output sits in `intent.allowed_outputs`; serves the stated purpose |
| `platform_fit` | 0.05 | Copy-safe area, crop survival, print bleed, thumbnail legibility |

Score each 0.00–1.00. Dimensions that do not apply (`photo_fidelity` with no source photo, `typography` on a textless gallery print) are marked `null` and their weight is redistributed proportionally — never scored 0.

```yaml
quality:
  subject: 0.96
  composition: 0.92
  focal_point: 0.88
  palette: 0.90
  typography: 0.81
  texture: 1.00
  style_coherence: 0.89
  photo_fidelity: 1.00
  intent_fit: 0.95
  platform_fit: 0.85
  overall: 0.920
  grade: A
```

`overall` is the weighted mean, not the arithmetic mean.

## Grade Mapping

This is the **Quality Score** — an output measure of the generated image. Do not confuse it with the **Editorial Score** from [analyzer.md](analyzer.md), which is an input measure of the source photo and drives `editorial_mode` on a different set of bands.

`overall × 100` maps to:

| Total | Grade | Label |
|-------|-------|-------|
| 90–100 | A | Premium Editorial |
| 75–89 | B | Standard Editorial |
| 60–74 | C | Compensation / Acceptable |
| <60 | D | Re-run Planner or Recovery |

Any single dimension below **0.60** fails the run regardless of `overall` — an otherwise strong image with a redrawn source photo is not a B. Route it to [iteration.md](iteration.md).

## Failure → Action

Do not act on more than one failure per pass. Pick the lowest-scoring failing dimension, apply its fix, regenerate, re-score — the loop and its stop rules live in [iteration.md](iteration.md).

Each row names the dimension that should have caught it and the layer that owns the fix. The mutation itself is [iteration.md](iteration.md)'s job.

| Observed failure | Dimension | Layer |
|------------------|-----------|-------|
| Photo redrawn | `photo_fidelity` | Compiler — set `fidelity: required`, add `photo redraw` to avoids |
| Style conflict visible | `style_coherence` | Art Direction — one grammar, or switch to `runner_up` |
| Missing Panter anchor | `focal_point` | Recovery — add `color_anchor` ([recovery/focus.md](../recovery/focus.md)) |
| Texture on a CLEAN layout | `texture` | Compiler — correct `texture_tier` per [assets/texture.md](../assets/texture.md) |
| Ground came out paper-white when the spec said otherwise | `texture` | Compiler — restate `ground` as the first clause |
| Objects float; no contact shadow; two viewpoints in one object | `composition` | Planner — set projection, ground plane, and light in `direction.composition` |
| Source cluster came back as a row of isolated objects | `composition` | Planner — `relationship-first`, and restate the arrangement from `image_report.spatial` |
| `painterly` returned as clean vector shapes | `texture` | Compiler — add the mark-making clause from [../assets/scene-construction.md](../assets/scene-construction.md) |
| Layout overcrowded | `typography` | Planner — reduce type scale within style DNA caps |
| Missing web copy-safe area | `platform_fit` | Planner — re-apply [layouts/website-hero.md](../layouts/website-hero.md) |
| Fake UI or unreadable labels | `platform_fit` | Planner — re-apply [layouts/interface-asset.md](../layouts/interface-asset.md) |
| Series outlier (palette or type drift vs. set) | `style_coherence` | Compiler — recompile that output alone against the series system ([series.md](series.md)); never re-run the set |
| Lock violation on a memory run | `style_coherence` | Compiler — recompile against the locked DNA ([visual-memory.md](visual-memory.md)) |

## Evaluator Output

```yaml
evaluation:
  grade: B
  quality:
    subject: 0.94
    composition: 0.90
    focal_point: 0.85
    palette: 0.88
    typography: 0.55        # below the 0.60 floor — fails the run
    texture: 1.00
    style_coherence: 0.91
    photo_fidelity: null    # no source photo — weight redistributed
    intent_fit: 0.95
    platform_fit: 0.87
    overall: 0.873
  lowest_failing: typography
  responsible_layer: planner
  next_action: iterate        # ship | iterate | ask_user
```

`overall` here is 0.873 — above 0.85 — but `typography` is below the 0.60 floor, so the run still fails. The floor is binding regardless of the mean.

`lowest_failing` and `responsible_layer` are what [iteration.md](iteration.md) consumes. An evaluator output without them is incomplete — the loop has nothing to act on.

Append the vector to the open run manifest ([../spec/visual-manifest.schema.md](../spec/visual-manifest.schema.md)); the Compiler owns the file.

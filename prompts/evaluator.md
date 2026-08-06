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

When user skips generation, score the compiled prompt:

| Dimension | Max | Criteria |
|-----------|-----|----------|
| Imageability | 25 | Every clause maps to visible pixels |
| Modularity | 20 | No analyzer logic leaked into compiler |
| Style fit | 20 | Matches Visual Language derivation |
| Recovery fit | 15 | Recoveries match Image Report flags |
| Hygiene | 20 | Zero banned adjectives |

## Grade Mapping

| Total | Grade | Label |
|-------|-------|-------|
| 90–100 | A | Premium Editorial |
| 75–89 | B | Standard Editorial |
| 60–74 | C | Compensation / Acceptable |
| <60 | D | Re-run Planner or Recovery |

## Failure → Action

| Failure | Action |
|---------|--------|
| Photo redrawn | Reject output; recompile with stronger fidelity clause |
| Style conflict visible | Re-run Reviewer + Compiler |
| Missing Panter anchor | Append [recovery/focus.md](../recovery/focus.md) clause, regenerate |
| Texture on a CLEAN layout | Strip texture clauses per [assets/texture.md](../assets/texture.md), regenerate |
| Layout overcrowded | Switch toward Gallery or MUJI language, reduce type |
| Missing web copy-safe area | Recompile with [layouts/website-hero.md](../layouts/website-hero.md) |
| Fake UI or unreadable labels | Recompile with [layouts/interface-asset.md](../layouts/interface-asset.md) |

## Evaluator Output

```yaml
evaluation:
  grade: B+
  score: 91
  checks_passed: 6
  checks_failed: 1
  failures: ["typography slightly large for MUJI"]
  recommendation: "Regenerate with caption-scale type clause"
```

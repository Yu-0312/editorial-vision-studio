# Iteration Engine

Run **after** Evaluator returns a failing or partial grade. Never regenerate blindly.

A low score is not a reason to redesign the image. It is a **pointer to one layer** that produced a bad decision. Fix that layer, recompile, regenerate. Everything else stays frozen.

## Loop

```
Evaluator quality vector
    ↓
Pick the single lowest failing dimension
    ↓
Attribute it to the responsible layer
    ↓
Apply the minimal mutation to the spec (not the prompt)
    ↓
Recompile → re-adapt → re-review → regenerate
    ↓
Evaluator again
```

Mutate the **spec**, never the emitted prompt string. A hand-patched prompt cannot be reproduced from the manifest and breaks model switching.

This rule bans **post-generation hand patches**. It is not violated by [optimizer.md](optimizer.md), which runs pre-generation inside the compile path and is deterministic — same spec, same adapter, same `ruleset_version`, same string — with every op recorded in the manifest. After the image exists, nothing may touch the prompt: the fix is a spec field, and the recompile re-runs the Optimizer on the way back through.

## Failure → Responsible Layer

The dimension names are the ten in [evaluator.md](evaluator.md) — no others. Every mutation is a **spec field change**; the named layer is the layer that owns that field, not a layer you re-run in isolation.

Two rows name the **Compiler**. Those are schema-conformance corrections — the Compiler already owns spec validation ([compiler.md](compiler.md)) — not creative re-decisions. Everything above the Compiler in the escalation chain is a decision layer.

| Failing dimension | Responsible layer | Minimal spec mutation |
|-------------------|-------------------|-----------------------|
| `subject` | Recovery | Add `silhouette_boost` or raise subject scale ([../recovery/subject.md](../recovery/subject.md)) |
| `composition` | Planner | Adjust `direction.composition` ratios; do not change style |
| `focal_point` | Recovery | Add `color_anchor` ([../recovery/focus.md](../recovery/focus.md)) |
| `palette` | Recovery | Add `panter_mode` or tighten palette to 4 ([../recovery/contrast.md](../recovery/contrast.md), [../recovery/palette.md](../recovery/palette.md)) — contrast failures land here |
| `typography` | Planner | Change typography scale/weight within style DNA caps — not the style |
| `texture` | Compiler | Correct `design_tokens.texture_tier` and drop disallowed tokens per [../assets/texture.md](../assets/texture.md) |
| `photo_fidelity` | Compiler | Set `photo_policy.fidelity: required` and add `photo redraw` to `avoids`; the adapter emits the stronger clause on recompile |
| `style_coherence` | Art Direction | One grammar must dominate; if two are still fighting, switch to `runner_up` ([art-direction.md](art-direction.md)). **On a memory or series run the direction is locked** — the fix is a Compiler recompile against the locked DNA, never a `runner_up` swap |
| `intent_fit` | Art Direction | The direction is wrong, not the execution — switch to `runner_up`. **When a preset or memory is active there is no `runner_up`** — stop and tell the user the chosen look does not fit the brief, so they can pick another at the [Style Gate](style-gate.md) |
| `platform_fit` | Planner | Re-apply layout copy-safe / margin rules ([../layouts/](../layouts/)) |

**One mutation per iteration.** Two simultaneous fixes make the next score unattributable.

After any mutation, the run re-enters at the **Compiler** and flows forward normally: Compiler → Adapter → Reviewer → Optimizer → Generate → Evaluator. "Responsible layer" names whose decision changed, not an entry point.

## Escalation Rule

If the same dimension fails twice at the same layer, the layer above it is wrong. Escalate:

```
Compiler  →  Recovery  →  Planner  →  Art Direction  →  Intent
```

Example: `typography` fails, Planner shrinks the scale, it fails again → the style cannot carry the message. Escalate to Art Direction and switch to `runner_up`.

## Stop Rules

Stop and report to the user when any of these hit:

| Condition | Action |
|-----------|--------|
| `overall ≥ 0.85` **and** no dimension below 0.60 | Ship. |
| `overall ≥ 0.85` but a dimension is below 0.60 | Keep iterating — the floor is binding regardless of the mean. |
| `iteration.count = 3` | Stop. Report the best iteration and the unresolved dimension. |
| `overall` dropped vs. previous iteration | Revert to the previous spec, report, ask. |
| Escalation reached Intent | Stop. The brief is ambiguous — ask the user, do not guess. |
| Same mutation proposed twice | Stop. The mutation is a no-op; report it. |

Three iterations is a ceiling, not a target. Most runs should end at 0 or 1.

## Iteration Record

Every iteration appends to the open manifest ([../spec/visual-manifest.schema.md](../spec/visual-manifest.schema.md)), which the Compiler opened when it first compiled the spec:

```yaml
iteration:
  count: 2
  history:
    - n: 1
      failing_dimension: typography
      overall_before: 0.71
      layer: planner
      mutation: "typography scale display → caption"
      overall_after: 0.79
    - n: 2
      failing_dimension: focal_point
      overall_before: 0.79
      layer: recovery
      mutation: "+color_anchor (cobalt, 1.8% canvas)"
      overall_after: 0.91
  stopped_by: threshold_met
```

`stopped_by`: `threshold_met` | `max_iterations` | `score_regression` | `escalated_to_intent` | `no_op_mutation`

## What Never Triggers Iteration

- User taste disagreement ("I don't like blue") — that is a direction change, not a QC failure. Route to [art-direction.md](art-direction.md).
- Model rendering artifacts unrelated to the spec (hands, garbled text) — regenerate the same spec with a different seed; do not mutate.
- A failure the prompt-only score already caught (buried ground clause, vague quantity, duplicated instruction) — that is wording, and [optimizer.md](optimizer.md) owns it. Spending an iteration on it wastes one of three.
- A dimension the brief never asked for. Do not fix `typography` on a gallery print with no title.

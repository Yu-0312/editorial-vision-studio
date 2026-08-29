# VisualManifest — Run Record

One manifest per generated output. Opened by the Compiler when it first compiles the spec, appended to by the Evaluator and the Iteration Engine, finalized when the run settles (pass, or stopped by [../prompts/iteration.md](../prompts/iteration.md)).

The spec ([editorial-spec.schema.md](editorial-spec.schema.md)) is the **input contract** — what the engine decided before generating.
The manifest is the **output record** — what actually happened, including the direction not taken, the iterations, and the scores.

## Why

| Without manifest | With manifest |
|------------------|---------------|
| "Regenerate that one from last week" → re-run the whole pipeline, get a different image | Replay the manifest, get the same direction |
| "Try it in Flux" → re-analyze | Swap `model.provider`, reuse everything else |
| "Go back to the other direction" → the runner-up is gone | `art_direction.runner_up` is recorded |
| Cross-image consistency by memory | `memory_id` links the run to its system |

## Schema

```yaml
manifest_version: "1.0"

project:
  name: tokyo-tower-editorial     # slug, stable across regenerations
  run_id: tokyo-tower-editorial-r2
  series_id: null                 # set when part of a series
  preset: null                    # preset id when one was named or chosen at the gate
  style_gate:                     # outcome + reason, verbatim from prompts/style-gate.md
    outcome: commit
    reason: preset_chosen
    description: null             # the user's words verbatim when reason is freeform — required for replay
  memory_id: null                 # set when a Visual Memory applied; this run establishes one instead
  memory_version: null            # the preset/memory `version` this run compiled against

intent:
  goal: "editorial poster of Tokyo Tower"
  family: Gallery
  purpose: editorial
  audience: general
  emotion: nostalgic
  platform: instagram
  source_type: photo

image_report:                     # null when theme-only
  subject: architecture
  editorial_score: 74
  flags: [low_saturation]

art_direction:                    # same block name as the spec — never `direction`
  id: A
  name: "Swiss Editorial"
  thesis: "The tower is geometry before it is a landmark."
  fit_score: 0.88
  selection_mode: user            # auto | user
  runner_up: B                    # switchable without re-analysis
  candidates_offered: [A, B, C]

visual_system:
  ground: paper-light             # required — see ../assets/ground.md
  render_mode: photo-plus-graphic # required
  style: swiss
  visual_language: Architectural
  palette: [warm ivory, charcoal, muted red]
  typography: "grotesk title, sans metadata"
  texture_tier: FLAT              # FLAT | SURFACE | PRINT
  atmosphere: quiet contemporary

layout:
  type: poster
  aspect_ratio: "4:5"
  composition:
    photo_ratio: 0.65
    abstract_ratio: 0.25
    type_ratio: 0.05
    whitespace_ratio: 0.05
  production_context: social

recoveries: [panter_mode, color_anchor]

model:
  provider: openai
  adapter: gpt-image
  reference_image: edit
  prompt_hash: sha256:…           # of the emitted GenerationRequest.prompt, post-optimizer
  seed: null

optimizer:                        # see ../prompts/optimizer.md
  ruleset_version: "1.0"          # required for replay — the ops are versioned, not frozen
  passes: 1
  status: optimized               # optimized | unchanged | reverted | rejected_to_compiler
  prompt_score_before: 74
  prompt_score_after: 89
  ops_applied: [ground_first, concretize]
  variants: []                    # populated only in variant mode; the selected id is the shipped prompt

quality:                          # all ten dimensions, verbatim from the evaluator
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

iteration:
  count: 2
  stopped_by: threshold_met
  history: []                     # see ../prompts/iteration.md

provenance:
  spec_version: "1.1"
  skill_version: "2.0"
  parent_run_id: tokyo-tower-editorial-r1 | null
```

## Field Rules

- `project.name` is stable; `run_id` increments. Two runs of the same project share a name and differ by `run_id`.
- `art_direction` uses the same block name and field names as the spec. The manifest has no `direction` block — style and palette live under `visual_system`.
- `quality` carries all ten evaluator dimensions, `null` for inapplicable ones. A partial vector is not a manifest.
- `visual_system.ground` and `visual_system.render_mode` are mandatory. The evaluator's `texture` dimension scores the rendered image against them, so a manifest without them makes that score unverifiable.
- `project.memory_version` records which version of a preset or memory the run compiled against, so an edited preset does not silently invalidate old runs.
- `art_direction.runner_up` must be a real candidate id, not a guess. Null when Art Direction auto-committed with no alternatives.
- `quality.overall` is the weighted mean from [../prompts/evaluator.md](../prompts/evaluator.md), not an average of the listed dimensions.
- `model.prompt_hash` makes "did this actually change?" answerable across iterations. A mutation that leaves the hash unchanged is a no-op — see the `no_op_mutation` stop rule. The hash is taken **after** the Optimizer, because that is the string the model received.
- `optimizer.ruleset_version` is what makes the run replayable. The prompt string is a deterministic function of spec + adapter + ruleset; without the version, a later edit to [../prompts/optimizer.md](../prompts/optimizer.md) silently changes what "replay" reproduces. Same reason as `project.memory_version`.
- `optimizer.status: unchanged` is the healthy default. `reverted` and a repeated op across passes both point at `adapters/{model}.md`, not at this run.
- `provenance.parent_run_id` is set when the run reused a prior spec (model switch, direction switch, memory continuation).

## Replay

| User says | Read from manifest | Re-run |
|-----------|--------------------|--------|
| "同一張，改用 Flux" | everything except `model` | Adapter → Reviewer → Optimizer → Generate |
| "改用 B 那個方向" | intent, image_report | Planner → Compiler → Adapter → Reviewer → Optimizer → Generate |
| "一模一樣再生一次" | everything, `optimizer` included | Generate only — replay the recorded prompt, do not re-optimize |
| "換一個寫法試試" | spec, `optimizer.variants` | Optimizer in variant mode → Generate |
| "同一套視覺，換主題" | `visual_system` → Visual Memory | Analyzer → Art Direction (auto-commit) → Planner → downstream |

Never re-run the Analyzer when the manifest already carries a valid `image_report` for the same source image.

## Emission

The **Compiler owns the manifest**. It opens one when it first compiles a spec, the Optimizer appends its op block, the Evaluator appends the quality vector, the Iteration Engine appends each pass, and the Compiler finalizes it when the run settles. One writer per section, one owner for the file.

Emit it to the user only when asked, or when the run is part of a series. Otherwise keep it internal and surface the two-line summary: direction name + quality grade.

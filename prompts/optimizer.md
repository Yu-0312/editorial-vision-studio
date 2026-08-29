# Prompt Optimizer

Run **after** [Prompt Reviewer](reviewer.md), **before** image generation. Improves **how the prompt says** what the spec already decided.

The Reviewer answers *is this prompt legal against its spec*. It cannot improve a prompt that is legal but weakly worded — a correct clause buried in sentence four, a spec number softened into "a small accent", the same instruction stated twice. That is this layer's only job.

## The One Rule

**The Optimizer may not introduce a decision.**

Every clause it emits must trace to a field that already exists in the spec. A clause it wants to add with no spec field behind it is a **rejection routed back to the Compiler** with the missing field named — exactly as the Reviewer does. It never sets a ground, style, subject, layout, palette, typography scale, or abstraction level. It has no taste.

| Layer | Question | May change |
|-------|----------|------------|
| [Compiler](compiler.md) | What is the spec, and what does this model call it? | the spec (schema conformance only) |
| [Reviewer](reviewer.md) | Is this request legal against its spec? | the request, by correction |
| **Optimizer** | Is this the strongest wording of a legal request? | clause order, wording, specificity, redundancy, budget |
| [Evaluator](evaluator.md) | Did the image come out right? | nothing |
| [Iteration](iteration.md) | Which decision was wrong? | one spec field |

## Why This Does Not Break the Prompt-String Rule

[iteration.md](iteration.md) forbids mutating the emitted prompt string. That rule is about **hand patches after generation** — they cannot be replayed from the manifest and they break model switching.

The Optimizer is not a hand patch. It is deterministic, pre-generation, and recorded:

```
same spec + same adapter + same ruleset_version  →  same prompt string
```

It sits **inside** the compile path, so replay is unaffected: the manifest records `optimizer.ruleset_version` and every op applied, and re-running the manifest reproduces the string. Nothing downstream of generation may touch the prompt — that rule stands unchanged.

The Optimizer never runs on `review_status: rejected`. A rejected request goes back to the Compiler first; optimizing the wording of an illegal prompt is polishing a defect.

## Ops

Run in order — they interact. Each op either fires or does not; there is no partial op.

Every op reads the target adapter's **`optimizer_contract`** block ([adapters/_template.md](../adapters/_template.md)) for the model facts it needs: sentence budget, whether a `negative_prompt` field exists, and which clause the model weights first. Those facts belong to the adapter, never to this file — that is what keeps "add a new model" from touching a shared layer.

| # | Op | Fires when | Rewrite | Never |
|---|----|------------|---------|-------|
| 1 | `provenance_strip` | A clause traces to no spec field | Delete it | Delete a clause backed by a locked memory or preset field, or **any clause at or below the ladder's floor**. The spatial clauses have no spec field of their own on a theme-only run — `image_report.spatial` is null — yet [reviewer.md](reviewer.md) rejects a prompt without them. Requiring a clause in one layer and deleting it in the next is a loop, not a cleanup |
| 2 | `ground_first` | The ground clause sits later than the adapter's `emphasis_order` allows | Move it to the front, or to the sentence immediately after the emphasis clause | Move it ahead of a `title-first` adapter's title — [ideogram.md](../adapters/ideogram.md) weights the first sentence for text, and reordering it costs a legible masthead |
| 3 | `concretize` | A clause names a quality instead of a thing — "editorial feel", "sophisticated palette", "thoughtful composition" | Replace with the spec's own value | Invent a value the spec does not hold. Missing value → reject to Compiler |
| 4 | `bind_numbers` | The spec holds a quantity but the prompt says a vague word — "a small accent", "lots of space", "a few colours" | Substitute the spec's number | Add a number the spec does not have |
| 5 | `dedupe` | The same instruction appears twice | Keep the more specific statement, drop the other | Merge two statements that constrain *different* fields — that is compression, not duplication |
| 6 | `route_negatives` | An avoid sits in the positive prompt and the adapter carries a `negative_prompt` | Move it | Drop the hard-avoids paragraph on adapters with no negative field |
| 7 | `compress` | The request exceeds the adapter's stated budget (e.g. flux ≤3 sentences) | Drop clauses lowest-first on the provenance ladder | Drop anything at or below the floor line |

Ops 3 and 4 are where most of the gain is. An image model renders nouns and numbers; it cannot render an adjective about taste. [compiler.md](compiler.md) already bans the fluff list — this op catches the fluff that is not on any list because it was phrased as a compliment to the design rather than a description of it.

### Provenance Ladder

What `compress` drops, in order. Stop at the floor.

```
atmosphere words
secondary palette members (beyond four)
texture qualifiers
typography detail
composition ratios
──────────────────────────────── floor ────────────────────────────────
subject and primary image anchor
projection, ground plane, light direction, contact shadows
design_tokens.ground
photo fidelity clause · locked memory or preset fields
```

**Stated cues are exempt from the whole ladder.** On a `reason: freeform` run, [reviewer.md](reviewer.md) rejects any prompt that dropped a cue from `style_gate.description` — and the user's own words are frequently atmosphere words, which the ladder would drop first. Compressing a stated cue away manufactures the very rejection this layer exists to avoid. Treat every cue in `style_gate.description` as floor.

Below the floor, `compress` does not fire. If the request is still over budget with only floor clauses left, that is a spec that does not fit the model — stop and report it, or switch adapters. Never shorten a prompt by dropping the thing that makes it reproducible.

## Variant Mode

The optimizer can emit **2–3 prompts from one spec** so a wording choice can be compared instead of guessed.

Variants differ **only on expression axes**:

| Axis | Values |
|------|--------|
| `clause_density` | `terse` — one constraint per sentence · `bound` — grouped in [compiler.md](compiler.md) renderability order · `narrative` — continuous prose |
| `specificity` | `literal` — every spec number stated · `directive` — numbers only where load-bearing, the rest as instructions |
| `emphasis_order` | Fixed per adapter, not a free choice — read `optimizer_contract.emphasis_order` from `adapters/{model}.md`. Vary the other two axes instead |

**Every variant must share one `spec_hash`.** A variant that differs on any spec field is not a variant — it is a second direction, and it belongs to [art-direction.md](art-direction.md). This is the whole point: holding the direction constant is what makes the comparison mean something.

Run variant mode only when the user asks to compare wordings, when `target.model` is `generic`, or when a previous run of this spec failed on a dimension the Evaluator attributed to the prompt rather than the spec. Otherwise ship the single optimized request — three prompts for a routine run is a menu nobody asked for, the same defect the [Style Gate](style-gate.md) exists to prevent.

Score variants with the **Prompt-Only Evaluation** table in [evaluator.md](evaluator.md) — that rubric already exists and this layer does not add a second one. The winner ships; the losers are recorded, which is what makes 「試另一個寫法」 cost one adapter run instead of a full pipeline.

## Stop Rules

| Condition | Action |
|-----------|--------|
| No op fired | Ship as-is. **This is the normal outcome** for a mature adapter |
| Ops fired and the prompt-only score rose | Ship |
| Ops fired and the score did not rise | Revert to the pre-optimizer request and record why |
| The same op fires on a second pass | Stop. The **adapter** is emitting the defect every time — fix `adapters/{model}.md`, not this string |
| An op needs a value the spec does not hold | Stop. Reject to the Compiler, naming the missing field |
| `review_status: rejected` | Do not run |
| `passes = 2` | Ceiling |

Two passes is a ceiling, not a target. A run that needs two is telling you something about the adapter.

## Output

```yaml
optimizer:
  ruleset_version: "1.0"
  passes: 1
  prompt_score_before: 74
  prompt_score_after: 89
  ops_applied:
    - op: ground_first
      clause: "warm ivory paper field"
      moved: "sentence 3 → sentence 1"
    - op: concretize
      before: "a sophisticated restrained palette"
      after: "four colours only: warm ivory, charcoal, muted red, bone"
      spec_field: visual_system.palette
    - op: bind_numbers
      before: "a small saturated accent"
      after: "one cobalt block covering 2% of the canvas"
      spec_field: recovery.color_anchor
  ops_declined:
    - op: compress
      reason: within_budget
  variants: []
  status: optimized        # optimized | unchanged | reverted | rejected_to_compiler
  generation_request:
    model: flux
    prompt: "..."
    negative_prompt: "..."
    aspect_ratio: "3:4"
    reference_image: none
    extra_params: {}
```

Variant mode populates `variants`:

```yaml
  variants:
    - id: v1
      clause_density: bound
      specificity: literal
      prompt_score: 89
      selected: true
    - id: v2
      clause_density: terse
      specificity: directive
      prompt_score: 81
      selected: false
```

Append the block to the open manifest ([../spec/visual-manifest.schema.md](../spec/visual-manifest.schema.md)); the Compiler owns the file.

Keep the block internal. Surface it only when the user asks what changed, when `status: reverted`, or in variant mode — where the user is choosing, so they see the prompts, never the op log.

## What Never Reaches This Layer

- **A prompt the user wrote themselves** and asked to be generated verbatim. Their words are the brief ([style-brief.md](style-brief.md)); the Optimizer does not improve a person's sentence.
- **Taste** — 「感覺再暖一點」 is a direction change. Route to [art-direction.md](art-direction.md).
- **A weak generated image.** That is [evaluator.md](evaluator.md) → [iteration.md](iteration.md), and the fix is a spec field, not a rewrite.
- **Length for its own sake.** `compress` exists; `expand` does not. A longer prompt is not a stronger one — every clause added past the point of renderability dilutes the ones that mattered.

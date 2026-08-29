# Visual Design Engine — Architecture

Editorial Vision Studio is a **model-agnostic visual decision engine** plus **swappable model adapters**.

Only the adapter layer changes when switching image models. The decision pipeline is stable.

## Two-Layer Split

```
┌─────────────────────────────────────────────────────────┐
│  DECISION ENGINE (model-agnostic)                       │
│  Intent → Style Gate → Analyzer → Visual Language       │
│         → Art Direction → Planner → Recovery            │
└──────────────────────────┬──────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────┐
│  COMPILER  → EditorialSpec (YAML)                       │◄──┐
└──────────────────────────┬──────────────────────────────┘   │
                           ▼                                  │
┌─────────────────────────────────────────────────────────┐   │
│  MODEL ADAPTER (swappable)                              │   │
│  EditorialSpec → prompt + gen params                    │   │
└──────────────────────────┬──────────────────────────────┘   │
                           ▼                                  │
              GPT Image / Flux / Ideogram / …                 │
                           ▼                                  │
┌─────────────────────────────────────────────────────────┐   │
│  SHARED POST-LAYER                                      │   │
│  Reviewer → Optimizer → Generate                        │   │
│                       → Evaluator → Iteration ──────────┼───┘
└──────────────────────────┬──────────────────────────────┘  one spec
                           ▼ pass                            mutation,
┌─────────────────────────────────────────────────────────┐  max 3
│  PERSISTENCE                                            │  passes
│  VisualManifest (this run) · VisualMemory (the system)  │
└─────────────────────────────────────────────────────────┘
```

## Three Contracts

| Contract | Written by | Answers |
|----------|-----------|---------|
| [EditorialSpec](../spec/editorial-spec.schema.md) | Decision Engine, before generating | What should this image be? |
| [VisualManifest](../spec/visual-manifest.schema.md) | Compiler, opened at compile time, finalized when the run settles | What did this run actually do? |
| [VisualMemory](../spec/visual-memory.schema.md) | Visual Memory after a passing run, or authored as a [preset](../presets/registry.md) | What must the next image inherit? |

Spec is intent. Manifest is history. Memory is identity. Keeping them separate is what makes replay, model switching, and multi-image consistency cheap — each one answers a different question, and no layer has to guess.

## What Never Changes

These modules are **pure editorial logic** — no model syntax:

| Layer | Output |
|-------|--------|
| Intent Engine | output family, allowed layouts, intent dimensions |
| Style Gate | the user's chosen look, as a session lock |
| Visual Analyzer | Image Report, Editorial Score |
| Visual Language Engine | language → style derivation |
| Art Direction Engine | candidate directions, fit scores, committed direction |
| Editorial Planner | layout, ratios, abstraction level |
| Recovery Engine | recovery module IDs |
| Quality Evaluator | quality vector, responsible layer |
| Iteration Engine | spec mutations, stop reason |
| Visual Memory | locked DNA across runs |
| Style / Layout / Recovery files | DNA parameters |

## What Changes Per Model

Only the **Model Adapter** ([adapters/](../adapters/)):

- Prompt shape (paragraphs vs tags vs JSON)
- Negative prompt syntax
- Typography emphasis (Ideogram vs Flux)
- Reference image handling (edit vs img2img vs none)
- Default aspect ratio / resolution hints
- Parameter names (guidance, steps, quality tier)

## VisionSpec / EditorialSpec Contract

The decision engine **must** emit [spec/editorial-spec.schema.md](../spec/editorial-spec.schema.md) before any adapter runs.
For backward compatibility the file is still named EditorialSpec; for general visual work, treat it as VisionSpec.

Adapters **must not** re-analyze the image or override Planner decisions — only translate.

## Adding a New Model

1. Copy [adapters/_template.md](../adapters/_template.md) → `adapters/your-model.md`
2. Define: prompt shape, negative syntax, reference-image policy, default params
3. Register in [adapters/registry.md](../adapters/registry.md)
4. No changes to Analyzer, Planner, Recovery, or Style files

## Where the Optimizer Sits

[prompts/optimizer.md](../prompts/optimizer.md) is a **fixed layer with model-specific inputs** — the one module that is neither pure editorial logic nor part of the adapter.

Its op set is stable across models. What varies is what it reads *from* the adapter: sentence budget, whether a `negative_prompt` field exists, whether the model weights early tokens. Those facts live in `adapters/{model}.md`, so adding a model still touches no shared layer.

The boundary that keeps this honest: **the Optimizer decides nothing.** Every clause it emits traces to a spec field; a clause with no field behind it is a rejection to the Compiler, not an invention. It edits expression, never content — which is why it can sit downstream of the Reviewer without becoming a second decision engine.

It runs **before** generation and is deterministic, so it does not violate the prompt-string rule below: the manifest records `ruleset_version` and every op, and replay reproduces the string exactly.

## Where Loops Are Allowed

Exactly one edge closes a cycle: Evaluator → Iteration → **Compiler**. Iteration never re-enters higher than the Compiler. It mutates a spec field, then the run flows forward normally — Compiler → Adapter → Reviewer → Generate → Evaluator.

Escalation (Compiler → Recovery → Planner → Art Direction → Intent) chooses *which spec field* the next mutation touches. It does not add extra edges: whichever layer's decision changed, the run still re-enters at the Compiler. The loop is bounded at three passes.

Every other edge in the diagram is one-way. A layer that reaches backward — an Adapter that re-plans, a Compiler that re-analyzes, a Reviewer that edits the spec, an Optimizer that adds a clause the spec never held — is a bug, not an optimisation.

## Adding a Preset

1. Copy [../presets/_template.md](../presets/_template.md) → `presets/your-preset.md`
2. Fill every locked field, `ground` and `render_mode` included
3. Register it in [../presets/registry.md](../presets/registry.md)
4. No changes to Analyzer, Planner, Recovery, adapters, or the spec — a preset is a VisualMemory with `source: preset`, so the existing lock machinery already enforces it

## Adding a New Style or Layout

1. Add `styles/foo.md` or `layouts/foo.md`
2. Planner auto-picks via Visual Language
3. All adapters read the same EditorialSpec fields — no per-model style forks

## Philosophy

> Same creative direction. Different rendering dialect.

The engine interprets the photograph. The adapter speaks the model's language.

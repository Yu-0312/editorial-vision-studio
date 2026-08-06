# Visual Design Engine — Architecture

Editorial Vision Studio is a **model-agnostic visual decision engine** plus **swappable model adapters**.

Only the adapter layer changes when switching image models. The decision pipeline is stable.

## Two-Layer Split

```
┌─────────────────────────────────────────────────────────┐
│  DECISION ENGINE (model-agnostic)                       │
│  Intent → Analyzer → Visual Language → Planner          │
│         → Recovery → EditorialSpec                      │
└──────────────────────────┬──────────────────────────────┘
                           │ EditorialSpec (YAML)
                           ▼
┌─────────────────────────────────────────────────────────┐
│  MODEL ADAPTER (swappable)                              │
│  VisionSpec / EditorialSpec → prompt + gen params       │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
              GPT Image / Flux / Ideogram / …
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│  SHARED POST-LAYER                                      │
│  Reviewer → Generate → Evaluator                        │
└─────────────────────────────────────────────────────────┘
```

## What Never Changes

These modules are **pure editorial logic** — no model syntax:

| Layer | Output |
|-------|--------|
| Intent Engine | output family, allowed layouts |
| Visual Analyzer | Image Report, Editorial Score |
| Visual Language Engine | language → style derivation |
| Editorial Planner | layout, ratios, abstraction level |
| Recovery Engine | recovery module IDs |
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

## Adding a New Style or Layout

1. Add `styles/foo.md` or `layouts/foo.md`
2. Planner auto-picks via Visual Language
3. All adapters read the same EditorialSpec fields — no per-model style forks

## Philosophy

> Same creative direction. Different rendering dialect.

The engine interprets the photograph. The adapter speaks the model's language.

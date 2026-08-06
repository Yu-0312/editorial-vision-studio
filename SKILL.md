---
name: editorial-vision-studio
description: >-
  Universal visual direction engine for AI image, design, and layout work:
  model-agnostic decision pipeline (intent, analysis, visual language, planning,
  recovery/refinement) plus swappable adapters for GPT Image, Flux, Ideogram,
  and generic image backends. Use for visual concepting, image prompts,
  photo-to-design, posters, covers, zines, gallery prints, campaigns, brand
  key visuals, product/editorial imagery, social assets, website hero art,
  moodboards, Panter-style low-contrast recovery, or switching image models
  while preserving the same creative direction.
---

# Editorial Vision Studio

AI Creative Director for Visual Generation.

**Philosophy:** Do not decorate. Always interpret.

This skill evolves [photo-abstract-editorial](https://github.com/ZzzLc0405/photo-abstract-editorial) (faithful photo + derived abstraction) and [gc-minimal-zine-poster](https://github.com/LiamGvchi/gc-minimal-zine-poster) (modular prompt compiler). It is an **extensible Editorial Design Engine**: one decision pipeline, swappable model adapters.

**Architecture:** [reference/architecture.md](reference/architecture.md)

## When to Use

- User asks for AI image direction, image prompts, art direction, visual concepting, or prompt adaptation across models
- User uploads a photo and asks for photo-to-design, editorial poster, cover, zine, gallery print, campaign key visual, brand visual, product visual, or hero image
- User gives a theme only and wants a poster, social asset, zine, campaign, moodboard, or conceptual image
- User mentions low-contrast / gray photo recovery (Panter compensation)
- User specifies a style: Swiss, Kinfolk, MUJI, Brutalist, Wallpaper*, Purple, Apartamento, POPEYE
- User wants analysis → direction → prompt → image, not immediate generation
- User specifies model: `gpt-image`, `flux`, `ideogram` — or asks to reuse direction with a different model

## Architecture: Decision Engine + Model Adapters

```
DECISION ENGINE (fixed)          MODEL ADAPTER (swappable)
Intent → Analyzer                     VisionSpec / EditorialSpec
      → Visual Language      →      ↓
      → Planner              →   adapters/{model}.md
      → Recovery             →      ↓
      → VisionSpec           →   GenerationRequest → API
```

- **Decision Engine** emits [spec/editorial-spec.schema.md](spec/editorial-spec.schema.md) — pure visual logic, zero model syntax
- **Model Adapter** translates spec → prompt ([adapters/registry.md](adapters/registry.md))
- Switching GPT Image → Flux → Ideogram: **reuse VisionSpec / EditorialSpec**, re-run adapter only

## Core Pipeline

```
User Request
    ↓
Intent Engine          → [prompts/intent.md](prompts/intent.md)
    ↓
Visual Analyzer        → [prompts/analyzer.md](prompts/analyzer.md)  (skip if theme-only / prompt-only)
    ↓
Visual Language Engine → [prompts/visual-language.md](prompts/visual-language.md)
    ↓
Visual Planner         → [prompts/planner.md](prompts/planner.md)
    ↓
Recovery Engine        → [prompts/recovery.md](prompts/recovery.md) + [recovery/](recovery/)
    ↓
VisionSpec             → [spec/editorial-spec.schema.md](spec/editorial-spec.schema.md)
    ↓
Model Adapter          → [adapters/registry.md](adapters/registry.md)  ← swappable
    ↓
Prompt Reviewer        → [prompts/reviewer.md](prompts/reviewer.md)
    ↓
Image Generation
    ↓
Quality Evaluator      → [prompts/evaluator.md](prompts/evaluator.md)
```

Each layer does **one job**. Never analyze in Compiler. Never generate in Analyzer.

Quick routing: [reference/decision-tree.md](reference/decision-tree.md)

## Step 0: Intent Engine

Before analyzing pixels, resolve **user goal → output family**:

| User says | Intent | Allowed outputs |
|-----------|--------|-----------------|
| art book cover | Art Book Cover | gallery cover, magazine cover, minimal cover, book jacket |
| TEDx key visual | Event Campaign | campaign poster, key visual, social banner, stage screen |
| skincare brand launch | Branding | brand key visual, product editorial, social set, packaging mock |
| app hero image | Digital Product | website hero, app-store visual, social banner |
| zine page | Zine | zine spread, poster, editorial spread |
| gallery print | Gallery | gallery print, full bleed, diptych |
| moodboard | Concept Board | moodboard, palette study, material board |

Read [prompts/intent.md](prompts/intent.md). Reject mismatched formats (e.g. gallery print for TEDx campaign).

## Step 1: Visual Analyzer

Produce structured **Image Report** with star ratings and Editorial Score (0–100).

Dimensions: subject, clarity, contrast, saturation, composition, negative space, geometry, texture, lighting, emotion.

Read [prompts/analyzer.md](prompts/analyzer.md).

## Step 2: Visual Language Engine

Derive **Visual Language first**, then style/palette/layout — not the reverse.

Examples: Museum → Swiss + ivory + fine serif; Quiet Human → Kinfolk + cream/sage; Indie Memory → Zine + riso anchor.

Read [prompts/visual-language.md](prompts/visual-language.md). User `style:` override skips auto-derivation but Reviewer still validates DNA fit.

## Step 3: Editorial Planner

Decide layout, typography direction, abstraction level — **not** the final prompt.

Key rules (full matrix in [prompts/planner.md](prompts/planner.md)):

- Portrait + negative space >50% → Magazine Cover
- Architecture + strong geometry → Swiss Poster
- Landscape + quiet mood → Gallery Print
- Street + human story → Documentary Zine
- Food/object + minimal → Product Editorial

If user specifies `style: kinfolk`, load [styles/kinfolk.md](styles/kinfolk.md) DNA.

## Step 4: Recovery Engine

Apply **only** when Image Report flags weakness. Each recovery is one atomic fix — see [recovery/](recovery/).

| Problem | Recovery |
|---------|----------|
| Low contrast / gray (saturation <30%) | Panter Mode: warm/cool conflict hues, high-sat anchor, wider tonal separation |
| Weak subject | Increase silhouette / scale |
| Flat lighting | Directional light |
| Busy background | Simplify geometry |
| Too many colors | Limit palette to 4 |
| No focal point | Editorial color anchor |
| No rhythm | Abstract panel |

**Panter Mode** (from photo-panter lineage): discard dull grays; boost warm to 75% / cool to 70% saturation; add 8% high-chroma anchor block; widen tonal separation and mark scale. Panter is a **colour** compensation and never adds texture on its own. See [recovery/contrast.md](recovery/contrast.md).

**Texture Permission** — single source of truth: [assets/texture.md](assets/texture.md). Three tiers: **PRINT** (riso/halftone/scan defects) is `zine` only; **SURFACE** (substrate character such as cotton paper) is allowed on CLEAN layouts whose style DNA rates Texture ★★★+; **FLAT** (zero texture words) covers the `photo-abstract-diptych` panel ground, `interface-asset`, the `website-hero` copy-safe area, and the `product-editorial` background. Recoveries never raise a layout's tier.

Never redesign the entire image unless Editorial Score <50 (Concept Reconstruction).

## Step 5: Prompt Compiler + Model Adapter

**Phase 1:** Assemble VisionSpec / EditorialSpec — read [prompts/compiler.md](prompts/compiler.md)

**Phase 2:** Route to adapter by `target.model`:

| Model | When | Adapter |
|-------|------|---------|
| `gpt-image` (default for photo upload) | Diptych, photo fidelity | [adapters/gpt-image.md](adapters/gpt-image.md) |
| `flux` | Zine texture, atmosphere | [adapters/flux.md](adapters/flux.md) |
| `ideogram` | Cover/campaign typography | [adapters/ideogram.md](adapters/ideogram.md) |
| `generic` | Unknown backend | [adapters/generic.md](adapters/generic.md) |

User: `model: flux` or "用 Flux 生成" → set adapter, **do not** re-analyze.

**Same direction, different model:** reuse VisionSpec / EditorialSpec, swap adapter only.

## Step 6: Prompt Reviewer

Before generation, run conflict detection. Read [prompts/reviewer.md](prompts/reviewer.md).

Examples:
- Swiss grid + Kinfolk organic → reject or resolve
- MUJI + heavy typography → reject
- Brutalist + soft pastoral palette → warn

Auto-correct incompatible pairings.

## Step 7: Quality Evaluator

After generation, verify photo fidelity, abstract traceability, style coherence, recovery evidence. Grade A–D.

Read [prompts/evaluator.md](prompts/evaluator.md). On failure, recompile with targeted fix — do not blindly regenerate.

## Editorial Score & Modes

| Score | Mode |
|-------|------|
| 90+ | Premium Editorial — refined extraction, minimal recovery |
| 70–89 | Standard Editorial |
| 50–69 | Compensation Mode — apply Recovery stack |
| <50 | Concept Reconstruction — abstract reinterpretation |

## Output Contract

Match the requested depth. Default to a concise direction summary plus `GenerationRequest`.

- Include an Image Report only when a source image is analyzed.
- Include full VisionSpec / EditorialSpec when the user asks for a reusable direction, comparison, or model switch.
- Include a generated image only when an image-generation tool is available and the user asks for generation; otherwise return the model-ready prompt.
- Include Quality Grade and evaluator notes after generating, or when the user requests review.

### Model switch without re-analysis

User: "同一份方向，改用 Ideogram" → reuse VisionSpec / EditorialSpec, run [adapters/ideogram.md](adapters/ideogram.md) only.

### Bilingual output

- Image prompt: English (model-optimized)
- Analysis/direction summary: match user's language (中文/English)

## Guardrails

**Never:**
- Redraw, filter, or stylize the original photo region (photo-abstract-editorial principle)
- Blindly copy fixed 60/30/10 layout — adapt proportions to subject
- Mix style languages without Reviewer pass
- Overload typography or decorative elements

**Always:**
- Preserve visual identity of source photo when one is provided
- Make every abstract mark traceable to a photo fact, theme fact, brand cue, or stated goal
- Keep prompts imageable and concrete
- Apply Recovery only when Image Report warrants it

## Style & Layout Reference

| Style | File |
|-------|------|
| Swiss | [styles/swiss.md](styles/swiss.md) |
| Kinfolk | [styles/kinfolk.md](styles/kinfolk.md) |
| MUJI | [styles/muji.md](styles/muji.md) |
| Brutalist | [styles/brutalist.md](styles/brutalist.md) |
| Wallpaper* | [styles/wallpaper.md](styles/wallpaper.md) |
| Apartamento | [styles/apartamento.md](styles/apartamento.md) |
| Purple Magazine | [styles/purple.md](styles/purple.md) |
| POPEYE | [styles/popeye.md](styles/popeye.md) |
| Monocle | [styles/monocle.md](styles/monocle.md) |
| COS | [styles/cos.md](styles/cos.md) |

| Layout | File |
|--------|------|
| Editorial Poster | [layouts/poster.md](layouts/poster.md) |
| Magazine Cover | [layouts/magazine-cover.md](layouts/magazine-cover.md) |
| Gallery Print | [layouts/gallery-print.md](layouts/gallery-print.md) |
| Zine | [layouts/zine.md](layouts/zine.md) |
| Editorial Spread | [layouts/editorial-spread.md](layouts/editorial-spread.md) |
| Campaign Poster | [layouts/campaign-poster.md](layouts/campaign-poster.md) |
| Brand Key Visual | [layouts/brand-key-visual.md](layouts/brand-key-visual.md) |
| Product Editorial | [layouts/product-editorial.md](layouts/product-editorial.md) |
| Website Hero | [layouts/website-hero.md](layouts/website-hero.md) |
| Social Asset | [layouts/social-asset.md](layouts/social-asset.md) |
| Moodboard | [layouts/moodboard.md](layouts/moodboard.md) |
| Interface Asset | [layouts/interface-asset.md](layouts/interface-asset.md) |

## Extending the Engine

| Extend | Action | Touch Decision Engine? |
|--------|--------|--------------------------|
| New style (Aesop, NYT Mag) | Add `styles/foo.md` | No |
| New layout | Add `layouts/foo.md` | No |
| New recovery | Add `recovery/foo.md` | No |
| New image model | Add `adapters/foo.md` + register | **No** |
| New intent family | Edit `prompts/intent.md` | Yes (minimal) |

See [adapters/_template.md](adapters/_template.md) for new models.

## Extending Styles

Add new magazines/brands by creating `styles/your-style.md` with Style DNA table + compiler clauses. No need to rewrite SKILL.md.

## Agent Config

Model parameters: [agents/openai.yaml](agents/openai.yaml)

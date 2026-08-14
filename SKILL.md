---
name: editorial-vision-studio
description: >-
  Universal visual direction engine for AI image, design, and layout work:
  model-agnostic decision pipeline (intent, analysis, visual language, planning,
  recovery/refinement) plus swappable adapters for GPT Image, Flux, Ideogram,
  and generic image backends. Use for visual concepting, image prompts,
  photo-to-design, posters, covers, zines, gallery prints, campaigns, brand
  key visuals, product/editorial imagery, social assets, website hero art,
  moodboards, presentation decks, Panter-style low-contrast recovery,
  multi-image series with a consistent visual system, or switching image
  models while preserving the same creative direction.
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
- User wants a **set**, not an image: campaign at all sizes, a carousel, a multi-page deck
- User wants a **second image that matches the first** — series consistency, brand system, "同一套視覺"

## Architecture: Decision Engine + Model Adapters

```
DECISION ENGINE (fixed)          MODEL ADAPTER (swappable)
Intent → Analyzer                     VisionSpec / EditorialSpec
      → Visual Language      →      ↓
      → Art Direction        →   adapters/{model}.md
      → Planner              →      ↓
      → Recovery             →   GenerationRequest → API
      → VisionSpec
                                 SHARED POST-LAYER
                                 Reviewer → Generate → Evaluator → Iteration
                                                                       │
                                 one spec mutation, re-enter Compiler ◄┘
                                 (max 3 passes)
```

- **Decision Engine** emits [spec/editorial-spec.schema.md](spec/editorial-spec.schema.md) — pure visual logic, zero model syntax
- **Model Adapter** translates spec → prompt ([adapters/registry.md](adapters/registry.md))
- Switching GPT Image → Flux → Ideogram: **reuse VisionSpec / EditorialSpec**, re-run adapter only
- Every run emits a **VisualManifest** ([spec/visual-manifest.schema.md](spec/visual-manifest.schema.md)) so any run is replayable

## Core Pipeline

```
User Request
    ↓
Intent Engine          → [prompts/intent.md](prompts/intent.md)
    ↓                    (series? → [prompts/series.md](prompts/series.md) · memory? → [prompts/visual-memory.md](prompts/visual-memory.md))
Visual Analyzer        → [prompts/analyzer.md](prompts/analyzer.md)  (skip if theme-only / prompt-only)
    ↓
Visual Language Engine → [prompts/visual-language.md](prompts/visual-language.md)
    ↓
Art Direction Engine   → [prompts/art-direction.md](prompts/art-direction.md)  ← 2–3 candidates, commit to one
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
Quality Evaluator      → [prompts/evaluator.md](prompts/evaluator.md)  → quality vector
    ↓
Iteration Engine       → [prompts/iteration.md](prompts/iteration.md)  ← fix one layer, loop (max 3)
    ↓
VisualManifest         → [spec/visual-manifest.schema.md](spec/visual-manifest.schema.md)
```

Each layer does **one job**. Never analyze in Compiler. Never generate in Analyzer. Never re-decide in Iteration — mutate the spec and re-run the layer that owns the failure.

Quick routing: [reference/decision-tree.md](reference/decision-tree.md)

## Step 0: Intent Engine

Before analyzing pixels, resolve **user goal → output family**:

| User says | Intent | Allowed outputs |
|-----------|--------|-----------------|
| art book cover | Art Book | `magazine-cover`, `gallery-print`, `poster` |
| TEDx key visual | Event Campaign | `campaign-poster`, `brand-key-visual`, `social-asset` |
| skincare brand launch | Branding | `brand-key-visual`, `product-editorial`, `social-asset` |
| app hero image | Digital Product | `website-hero`, `interface-asset`, `social-asset` |
| zine page | Zine | `zine`, `poster`, `editorial-spread` |
| gallery print | Gallery | `gallery-print`, `photo-abstract-diptych`, `poster` |
| moodboard | Visual Concept | `moodboard`, `poster`, `editorial-spread` |
| 10-page deck | any family, `purpose: presentation` | `presentation-deck` |

Also resolve the six intent dimensions — **subject, purpose, audience, emotion, platform, aspect ratio**. Infer them; ask only when a missing one is load-bearing. They are what every later layer is graded against.

Detect scope here, not later: `series_id` for a set ([prompts/series.md](prompts/series.md)), `memory_id` for a continuation ([prompts/visual-memory.md](prompts/visual-memory.md)).

Read [prompts/intent.md](prompts/intent.md). Reject mismatched formats (e.g. gallery print for TEDx campaign).

## Step 1: Visual Analyzer

Produce structured **Image Report** with star ratings and Editorial Score (0–100).

Dimensions: subject, clarity, contrast, saturation, composition, negative space, geometry, texture, lighting, emotion.

Read [prompts/analyzer.md](prompts/analyzer.md).

## Step 2: Visual Language Engine

Derive **Visual Language first**, then style/palette/layout — not the reverse.

Examples: Museum → Swiss + ivory + fine serif; Quiet Human → Kinfolk + cream/sage; Indie Memory → Zine + riso anchor.

Read [prompts/visual-language.md](prompts/visual-language.md). User `style:` override skips auto-derivation but Reviewer still validates DNA fit.

## Step 3: Art Direction Engine

Do not generate the first plausible reading of the brief. Draft **2–3 competing directions**, each with a one-sentence thesis and a named trade-off, score them for fit, commit to one, and keep the runner-up.

| Situation | Behaviour |
|-----------|-----------|
| Explicit `style:`, narrow intent family, or active Visual Memory | Auto-commit; name the runner-up in one line. The layer still runs — every spec needs an `art_direction` block |
| Top two candidates within 0.15 `fit_score`, theme-only brief, or Editorial Score <50 | Offer 2–3 and let the user pick |

Candidates must differ on at least two of: visual language, layout family, abstraction level, palette temperature, typography weight. Palette swaps are not directions.

The committed direction is a **hard constraint** on the Planner. Switching to the runner-up later re-runs Planner onward only — never the Analyzer.

Read [prompts/art-direction.md](prompts/art-direction.md).

## Step 4: Editorial Planner

Decide layout, typography direction, abstraction level **inside the committed direction** — **not** the final prompt.

Key rules (full matrix in [prompts/planner.md](prompts/planner.md)):

- Portrait + negative space >50% → Magazine Cover
- Architecture + strong geometry → Swiss Poster
- Landscape + quiet mood → Gallery Print
- Street + human story → Documentary Zine
- Food/object + minimal → Product Editorial

If user specifies `style: kinfolk`, load [styles/kinfolk.md](styles/kinfolk.md) DNA.

## Step 5: Recovery Engine

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

## Step 6: Prompt Compiler + Model Adapter

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

## Step 7: Prompt Reviewer

Before generation, run conflict detection. Read [prompts/reviewer.md](prompts/reviewer.md).

Examples:
- Swiss grid + Kinfolk organic → reject or resolve
- MUJI + heavy typography → reject
- Brutalist + soft pastoral palette → warn

Auto-correct incompatible pairings.

## Step 8: Quality Evaluator

After generation, score a **quality vector** — not one number. Ten weighted dimensions: subject, composition, focal_point, palette, typography, texture, style_coherence, photo_fidelity, intent_fit, platform_fit. Each 0.00–1.00; inapplicable dimensions are `null`, never 0.

`overall` is the weighted mean and maps to grade A–D. **Any single dimension below 0.60 fails the run regardless of `overall`.**

The evaluator must name `lowest_failing` and `responsible_layer` — without them the iteration loop has nothing to act on.

Read [prompts/evaluator.md](prompts/evaluator.md).

## Step 9: Iteration Engine

A low score points at **one layer**, not at the whole image. Fix that layer, recompile, regenerate.

```
lowest failing dimension → responsible layer → one minimal spec mutation → recompile → re-score
```

- **One mutation per iteration.** Two at once makes the next score unattributable.
- **Mutate the spec, never the prompt string.** A hand-patched prompt cannot be replayed from the manifest.
- **Escalate** when the same dimension fails twice: Compiler → Recovery → Planner → Art Direction → Intent.
- **Re-enter at the Compiler** after every mutation. "Responsible layer" names whose decision changed, not an entry point.
- **Stop** at `overall ≥ 0.85` **with no dimension below 0.60**, at 3 iterations, on score regression, or when escalation reaches Intent — then report, don't guess.

Taste disagreement ("I don't like the blue") is a direction change, not a QC failure — route it to [prompts/art-direction.md](prompts/art-direction.md).

Read [prompts/iteration.md](prompts/iteration.md).

## Visual Memory & Series

The engine keeps a visual system across images, not just within one.

| Ask | Read |
|-----|------|
| "now do the next one," "同一套視覺," brand assets supplied | [prompts/visual-memory.md](prompts/visual-memory.md) |
| "一套 10 頁簡報," "campaign 全尺寸," carousel, all platform sizes | [prompts/series.md](prompts/series.md) |

**Visual Memory** locks the identity fields — style, visual language, palette, typography, texture tier — and leaves layout, composition, abstraction level, and aspect ratio free per image. Locking composition produces a template, not a system. On a memory run, Art Direction still runs but auto-commits to the locked DNA rather than offering candidates. A lock that no longer fits gets **forked**, never silently mutated. Contract: [spec/visual-memory.schema.md](spec/visual-memory.schema.md).

**Series** runs Intent → Analyzer → Visual Language → Art Direction **once**, then fans out per output. Generate the `hero` first and pass QC before derivatives — a weak hero multiplies into N weak frames. Cross-image QC scores palette drift, typographic identity, texture tier, anchor legibility, and compositional variety; a single outlier is fixed alone, never by re-running the set.

## Run Manifest

Every run emits [spec/visual-manifest.schema.md](spec/visual-manifest.schema.md): direction taken and runner-up, visual system, quality vector, iteration history, prompt hash, provenance.

This is what makes the cheap paths cheap:

| User says | Re-run |
|-----------|--------|
| "同一張，改用 Flux" | Adapter → Reviewer → Generate |
| "改用 B 那個方向" | Planner → downstream |
| "一模一樣再生一次" | Generate only |
| "同一套視覺，換主題" | Analyzer → Art Direction (auto-commit) → Planner → downstream |

Never re-run the Analyzer when a valid `image_report` for the same source image already exists. Keep the manifest internal unless the user asks for it or the run is part of a series.

## Editorial Score & Modes

This is the **Editorial Score** — an input measure of the source photo, produced by the Analyzer. It is not the Quality Score from Step 8, which measures the generated image on different bands.

| Editorial Score | Mode |
|-----------------|------|
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
- Include the full quality vector only when the user asks why, or when a dimension failed.
- Include the VisualManifest only on request, or when the run belongs to a series.
- Present Art Direction candidates as a 3-line table — name, thesis, trade-off — never as raw YAML.

### Model switch without re-analysis

User: "同一份方向，改用 Ideogram" → reuse VisionSpec / EditorialSpec, run [adapters/ideogram.md](adapters/ideogram.md) only.

### Direction switch without re-analysis

User: "改用 B 那個方向" → reuse Intent + Image Report, set the runner-up as selected, re-run Planner onward.

### Series and continuation

User: "同一套視覺，做東京街景" → load the Visual Memory, run Analyzer on the new photo, let Art Direction auto-commit to the locked DNA, and Planner sets free fields only.

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
- Give each direction a thesis before giving it a palette
- Fix one layer per iteration, and stop at three
- Fork a Visual Memory rather than mutating a lock to rescue one image

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
| Presentation Deck | [layouts/presentation-deck.md](layouts/presentation-deck.md) |

## Extending the Engine

| Extend | Action | Touch Decision Engine? |
|--------|--------|--------------------------|
| New style (Aesop, NYT Mag) | Add `styles/foo.md` | No |
| New layout | Add `layouts/foo.md` | No |
| New recovery | Add `recovery/foo.md` | No |
| New image model | Add `adapters/foo.md` + register | **No** |
| New intent family | Edit `prompts/intent.md` | Yes (minimal) |
| New QC dimension | Edit `prompts/evaluator.md` + map a layer in `prompts/iteration.md` | Yes (minimal) |
| New lockable DNA field | Edit `spec/visual-memory.schema.md` + enforce in `prompts/reviewer.md` | Yes (minimal) |

See [adapters/_template.md](adapters/_template.md) for new models.

## Extending Styles

Add new magazines/brands by creating `styles/your-style.md` with Style DNA table + compiler clauses. No need to rewrite SKILL.md.

## Engine Reference

| Layer | File |
|-------|------|
| Intent Engine | [prompts/intent.md](prompts/intent.md) |
| Visual Analyzer | [prompts/analyzer.md](prompts/analyzer.md) |
| Visual Language Engine | [prompts/visual-language.md](prompts/visual-language.md) |
| Art Direction Engine | [prompts/art-direction.md](prompts/art-direction.md) |
| Visual Planner | [prompts/planner.md](prompts/planner.md) |
| Recovery Engine | [prompts/recovery.md](prompts/recovery.md) |
| Prompt Compiler | [prompts/compiler.md](prompts/compiler.md) |
| Prompt Reviewer | [prompts/reviewer.md](prompts/reviewer.md) |
| Quality Evaluator | [prompts/evaluator.md](prompts/evaluator.md) |
| Iteration Engine | [prompts/iteration.md](prompts/iteration.md) |
| Visual Memory | [prompts/visual-memory.md](prompts/visual-memory.md) |
| Series Planner | [prompts/series.md](prompts/series.md) |

| Contract | File |
|----------|------|
| VisionSpec / EditorialSpec | [spec/editorial-spec.schema.md](spec/editorial-spec.schema.md) |
| VisualManifest | [spec/visual-manifest.schema.md](spec/visual-manifest.schema.md) |
| VisualMemory | [spec/visual-memory.schema.md](spec/visual-memory.schema.md) |

## Agent Config

Model parameters: [agents/openai.yaml](agents/openai.yaml)

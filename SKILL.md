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

An **extensible Editorial Design Engine**: one decision pipeline, swappable model adapters. Every layer — intent, analysis, art direction, planning, recovery, compilation, review, iteration — is a separate module with a single job, and the contract between them is a model-agnostic spec.

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
- User names a preset, or wants a specific fixed look reproduced exactly

## Architecture: Decision Engine + Model Adapters

```
DECISION ENGINE (fixed)          MODEL ADAPTER (swappable)
Intent → Style Gate                   VisionSpec / EditorialSpec
      → Analyzer             →      ↓
      → Visual Language      →   adapters/{model}.md
      → Art Direction        →      ↓
      → Planner              →   GenerationRequest → API
      → Recovery             →
      → Compiler → VisionSpec
                                 SHARED POST-LAYER
                                 Reviewer → Optimizer → Generate
                                          → Evaluator → Iteration
                                                             │
                                 one spec mutation,          │
                                 re-enter Compiler ◄─────────┘
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
Style Gate             → [prompts/style-gate.md](prompts/style-gate.md)  ← ask the user once, up front
    ↓
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
Prompt Reviewer        → [prompts/reviewer.md](prompts/reviewer.md)  ← is it legal?
    ↓
Prompt Optimizer       → [prompts/optimizer.md](prompts/optimizer.md)  ← is it well said?
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
| skincare brand launch | Branding | `brand-key-visual`, `product-editorial`, `social-asset`, `campaign-poster`, `website-hero` |
| app hero image | Digital Product | `website-hero`, `interface-asset`, `social-asset` |
| zine page | Zine | `zine`, `poster`, `editorial-spread` |
| gallery print | Gallery | `gallery-print`, `photo-abstract-diptych`, `poster` |
| moodboard | Visual Concept | `moodboard`, `poster`, `editorial-spread` |
| 10-page deck | any family, `purpose: presentation` | `presentation-deck` |

Also resolve the six intent dimensions — **subject, purpose, audience, emotion, platform, aspect ratio**. Infer them; ask only when a missing one is load-bearing. They are what every later layer is graded against.

Detect scope here, not later: `series_id` for a set ([prompts/series.md](prompts/series.md)), `memory_id` for a continuation ([prompts/visual-memory.md](prompts/visual-memory.md)).

Read [prompts/intent.md](prompts/intent.md). Reject mismatched formats (e.g. gallery print for TEDx campaign).

## Step 0.5: Style Gate

**The first question about how the image should look.** Before analysing a single pixel, ask which look they want — a numbered menu of the compatible presets plus 「讓 AI 提案」. (Intent may have asked about purpose or platform first; that is a different question.)

Half a step because it computes nothing. It asks one question and turns the answer into a lock.

```
要什麼風格？

1. 米色明信片 —— 米白紙底，照片重畫成簡化色塊，大量留白。
2. 時代海報 —— 飽和油墨滿版，硬邊平面色塊，地名做成版面。
3. 紙雕明信片 —— 寫實照片，明信片上長出立體紙雕世界。1:1。
4. 自己描述 —— 你說想要的樣子，我拆解照片後照你的描述重畫。
5. 讓 AI 提案 —— 看過照片和用途後給你三個方向再選。
```

**Nobody chooses a look twice unless they asked to see proposals.** The gate emits a single field, `style_gate.outcome`, and Art Direction branches on that and nothing else:

| Gate answer | `outcome` | Art Direction |
|-------------|-----------|---------------|
| A preset | `commit` | Becomes the session's Visual Memory; auto-commits, asks nothing |
| 自己描述, or just typing a description | `commit` | Captured verbatim; the Analyzer deconstructs the photo, then [prompts/style-brief.md](prompts/style-brief.md) rebuilds it to the description. A description is never answered with a menu |
| 「讓 AI 提案」 | `offer` | Offers 2–3 candidates. This is the one path with two prompts, and the user asked for it |

It skips — `outcome: commit`, no menu — when a preset or style is already named, a memory or series is active, a direction was committed earlier this session, or the run is unattended. It skips to `outcome: offer` when fewer than two presets fit the intent, because a one-item menu is not a choice. Unattended runs pick the highest-fit preset with intent-only weights and say which in one line — never silently.

Presets declare `intended_layouts`; the gate offers only those that intersect `intent.allowed_outputs`.

Show looks, not internal vocabulary. Nobody outside this repo knows what Kinfolk means.

The gate settles **ground and medium only**. Layout, composition, abstraction, and copy stay with the Planner.

Read [prompts/style-gate.md](prompts/style-gate.md).

## Step 1: Visual Analyzer

Produce structured **Image Report** with star ratings and Editorial Score (0–100).

Dimensions: subject, clarity, contrast, saturation, composition, negative space, geometry, texture, lighting, emotion.

Read [prompts/analyzer.md](prompts/analyzer.md).

## Step 2: Visual Language Engine

Derive **Visual Language first**, then style/palette/layout — not the reverse.

Examples: Museum → Swiss or MUJI + fine serif; Quiet Human → Kinfolk + cream/sage; Indie Memory → Zine + riso anchor; Poster Graphic → Travel Poster + flat inks.

Read [prompts/visual-language.md](prompts/visual-language.md). User `style:` override skips auto-derivation but Reviewer still validates DNA fit.

## Step 3: Art Direction Engine

Do not generate the first plausible reading of the brief. Draft competing directions, each with a one-sentence thesis and a named trade-off, score them for fit, commit to one, and keep the runner-up.

**Branch on `style_gate.outcome` and nothing else.** The gate already resolved every case — preset chosen, free text, 「你決定」, explicit `style:`, memory, series, unattended, too few presets — into one of two values:

| `outcome` | Behaviour |
|-----------|-----------|
| `commit` | Build one direction, commit silently. **Never offer candidates** — that would be the user's second time choosing a look. Name the runner-up in one line when one exists |
| `offer` | Present 2–3 candidates and ask. Two for a narrow intent family, three for a theme-only brief or Editorial Score <50 |

The layer always runs, even on `commit` — every spec needs an `art_direction` block with a thesis.

**Every candidate must set a different `ground`.** Eight of the eleven style DNAs resolve to a light paper field, so candidates chosen on style alone come back as three shades of ivory. Beyond that, candidates must differ on at least two of: render mode, visual language, layout family, abstraction level, typography weight. Palette swaps are not directions.

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

**Panter Mode**: discard dull grays; boost warm to 75% / cool to 70% saturation; add 8% high-chroma anchor block; widen tonal separation and mark scale. Panter is a **colour** compensation and never adds texture on its own. See [recovery/contrast.md](recovery/contrast.md).

**Texture Permission** — single source of truth: [assets/texture.md](assets/texture.md). Three tiers: **PRINT** (riso/halftone/scan defects) is `zine` only; **SURFACE** (substrate character such as cotton paper) is allowed on CLEAN layouts whose style DNA rates Texture ★★★+; **FLAT** (zero texture words) covers the `photo-abstract-diptych` panel ground, `interface-asset`, the `website-hero` copy-safe area, the `product-editorial` background, and every page of a `presentation-deck` that contains a `data` page. Recoveries never raise a layout's tier.

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

## Step 7.5: Prompt Optimizer

The Reviewer answers *is this prompt legal*. It cannot fix a prompt that is legal but weakly worded — the ground clause buried in sentence four, a spec number softened to "a small accent", the same instruction stated twice.

Half a step because it decides nothing. **Every clause it emits must trace to a field the spec already holds**; a clause with no spec field behind it is a rejection back to the Compiler, not an invention. It never sets a ground, style, subject, layout, palette, or type scale.

Seven ordered ops: `provenance_strip` · `ground_first` · `concretize` · `bind_numbers` · `dedupe` · `route_negatives` · `compress`. Most gain comes from `concretize` and `bind_numbers` — a model renders nouns and numbers, never an adjective about taste.

This does **not** break the "mutate the spec, never the prompt string" rule in [prompts/iteration.md](prompts/iteration.md). That rule forbids **hand patches after generation**. The Optimizer is deterministic, pre-generation, and recorded: same spec + same adapter + same `ruleset_version` produces the same string, so the manifest still replays.

**No op firing is the normal outcome.** The same op firing twice means the adapter emits that defect every time — fix `adapters/{model}.md`, not the string.

Variant mode emits 2–3 prompts from **one** spec, differing only on `clause_density`, `specificity`, and `emphasis_order`. A variant that differs on any spec field is a second direction, not a variant. Offer it only when the user asks to compare wordings, when the model is `generic`, or when a prompt-attributed dimension failed last run.

Read [prompts/optimizer.md](prompts/optimizer.md).

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

## Ground & Render Mode

Two axes, **required, with no default**:

| Axis | Values | Question it answers |
|------|--------|---------------------|
| `design_tokens.ground` | `paper-light` · `neutral-gray` · `dark` · `saturated` · `full-bleed-photo` · `duotone` | What is the canvas field itself? |
| `direction.render_mode` | `photographic` · `photo-plus-graphic` · `graphic` · `painterly` · `mixed` | What medium is the image made of? |

Enum values are contract tokens, never prompt words — every adapter resolves them to prose through [assets/ground.md](assets/ground.md).

An unset value is a **rejection, not a fallback**. This matters more than it looks: before these fields existed, an undecided ground fell through to warm ivory paper on every run, because that was the most-repeated value in the repo. A field with no default cannot be skipped.

`render_mode` is not `abstraction_level`. Medium and distance-from-source are independent — a `photographic` image can still be `full-abstract`.

## Presets

A preset is a shipped [VisualMemory](spec/visual-memory.schema.md) with `source: preset` — a locked partial spec authored in the repo instead of established from a run. It reuses the entire memory mechanism; nothing new enforces it.

| Preset | Ground | Render mode |
|--------|--------|-------------|
| [ivory-postcard](presets/ivory-postcard.md) | `paper-light` | `painterly` |
| [vintage-travel-poster](presets/vintage-travel-poster.md) | `saturated` | `graphic` |
| [papercraft-diorama-postcard](presets/papercraft-diorama-postcard.md) | `full-bleed-photo` | `photographic` |

The [Style Gate](prompts/style-gate.md) offers these as a numbered menu at the start of a run; `preset: ivory-postcard` is the shortcut that skips the menu, for automation and series work. Three different grounds is deliberate — it is what gives the candidate-diversity rule something to draw on. Registry and authoring guide: [presets/registry.md](presets/registry.md).

Unlike a series memory, a preset **may lock `composition`, `aspect_ratio`, and `layout`**: a preset is avowedly a template, which is what it is for.

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
- Include the optimizer op log only when the user asks what changed, or when it reverted. In variant mode show the prompts, never the op log — the user is choosing wordings, not reviewing edits.
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
- Redraw, filter, or stylize the original photo region when `photo_policy.fidelity: required`
- Blindly copy fixed 60/30/10 layout — adapt proportions to subject
- Mix style languages without Reviewer pass
- Overload typography or decorative elements
- Let the Optimizer add a clause no spec field backs, or lengthen a prompt to strengthen it

**Always:**
- Preserve the source's arrangement, overlaps, and relative scale — abstraction removes detail, not relationships
- Name one projection, one ground plane, one light direction; give every grounded object a contact shadow
- Preserve visual identity of source photo when one is provided
- Make every abstract mark traceable to a photo fact, theme fact, brand cue, or stated goal
- Keep prompts imageable and concrete
- Apply Recovery only when Image Report warrants it
- Give each direction a thesis before giving it a palette
- Fix one layer per iteration, and stop at three
- Fork a Visual Memory rather than mutating a lock to rescue one image

## Style & Layout Reference

| Asset module | File |
|--------------|------|
| Ground & Render Mode | [assets/ground.md](assets/ground.md) |
| Scene construction (space) | [assets/scene-construction.md](assets/scene-construction.md) |
| Texture permission | [assets/texture.md](assets/texture.md) |
| Palette | [assets/palette.md](assets/palette.md) |
| Typography | [assets/typography.md](assets/typography.md) |

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
| Period Travel Poster | [styles/travel-poster.md](styles/travel-poster.md) |

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
| Photo-Abstract Diptych | [layouts/photo-abstract-diptych.md](layouts/photo-abstract-diptych.md) |
| Presentation Deck | [layouts/presentation-deck.md](layouts/presentation-deck.md) |

## Extending the Engine

| Extend | Action | Touch Decision Engine? |
|--------|--------|--------------------------|
| New style (Aesop, NYT Mag) | Add `styles/foo.md` | No |
| New preset | Add `presets/foo.md` + register | **No** |
| New layout | Add `layouts/foo.md` | No |
| New recovery | Add `recovery/foo.md` | No |
| New image model | Add `adapters/foo.md` + register | **No** |
| New intent family | Edit `prompts/intent.md` | Yes (minimal) |
| New QC dimension | Edit `prompts/evaluator.md` + map a layer in `prompts/iteration.md` | Yes (minimal) |
| New prompt-wording op | Add a row to `prompts/optimizer.md` + bump `ruleset_version` | **No** |
| New lockable DNA field | Edit `spec/visual-memory.schema.md` + enforce in `prompts/reviewer.md` | Yes (minimal) |

See [adapters/_template.md](adapters/_template.md) for new models.

## Extending Styles

Add new magazines/brands by creating `styles/your-style.md` with Style DNA table + compiler clauses. No need to rewrite SKILL.md.

## Engine Reference

| Layer | File |
|-------|------|
| Intent Engine | [prompts/intent.md](prompts/intent.md) |
| Style Gate | [prompts/style-gate.md](prompts/style-gate.md) |
| Style Brief | [prompts/style-brief.md](prompts/style-brief.md) |
| Visual Analyzer | [prompts/analyzer.md](prompts/analyzer.md) |
| Visual Language Engine | [prompts/visual-language.md](prompts/visual-language.md) |
| Art Direction Engine | [prompts/art-direction.md](prompts/art-direction.md) |
| Visual Planner | [prompts/planner.md](prompts/planner.md) |
| Recovery Engine | [prompts/recovery.md](prompts/recovery.md) |
| Prompt Compiler | [prompts/compiler.md](prompts/compiler.md) |
| Prompt Reviewer | [prompts/reviewer.md](prompts/reviewer.md) |
| Prompt Optimizer | [prompts/optimizer.md](prompts/optimizer.md) |
| Quality Evaluator | [prompts/evaluator.md](prompts/evaluator.md) |
| Iteration Engine | [prompts/iteration.md](prompts/iteration.md) |
| Visual Memory | [prompts/visual-memory.md](prompts/visual-memory.md) |
| Series Planner | [prompts/series.md](prompts/series.md) |
| Preset Registry | [presets/registry.md](presets/registry.md) |

| Contract | File |
|----------|------|
| VisionSpec / EditorialSpec | [spec/editorial-spec.schema.md](spec/editorial-spec.schema.md) |
| VisualManifest | [spec/visual-manifest.schema.md](spec/visual-manifest.schema.md) |
| VisualMemory | [spec/visual-memory.schema.md](spec/visual-memory.schema.md) |

## Agent Config

Model parameters: [agents/openai.yaml](agents/openai.yaml)

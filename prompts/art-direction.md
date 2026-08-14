# Art Direction Engine

Run **after** Visual Language, **before** Planner.

Visual Language derives *one* likely reading of the brief. Art Direction proves it was a choice, not a reflex: draft **2–3 competing directions**, score them, commit to one, and record the runner-up so the user can switch without re-running the pipeline.

## Why This Layer Exists

A single auto-derived direction hides the decision. When the user says "not quite," there is nothing to pivot to and the whole pipeline re-runs. Candidate directions make the pivot a one-line swap.

## When to Offer Choices

| Condition | Behaviour |
|-----------|-----------|
| User set explicit `style:` | **Auto-commit.** Build one direction from the override. Name the runner-up in one line. |
| Intent family is narrow (Interface Asset, Product / Object) | **Auto-commit.** Name the runner-up in one line. |
| Series continuation (`memory_id` set) | **Auto-commit** to the locked DNA. The layer still runs and still emits `art_direction` — it just offers no candidates. See [visual-memory.md](visual-memory.md) |
| Top two candidates within 0.15 `fit_score` | **Offer 2–3.** Ask the user to pick. |
| No image and no style (theme-only brief) | **Offer 2–3.** The brief underdetermines the look. |
| Editorial Score <50 (reconstruction) | **Offer 2–3.** Reconstruction is an interpretive act; show the interpretations. |

Never offer more than three. Three directions that genuinely differ beat five that are palette swaps.

## Direction Requirements

Each candidate must differ on **at least two** of: visual language, layout family, abstraction level, palette temperature, typography weight. Two directions that share everything but the accent hue are one direction.

Each candidate carries a **thesis** — one sentence naming what it argues the image is *about*. A candidate with no thesis is decoration.

## Candidate Schema

The layer's working output is a `directions[]` list plus a `selected` id. What lands in the spec is the **single `art_direction` block** ([../spec/editorial-spec.schema.md](../spec/editorial-spec.schema.md)), populated from the selected candidate.

```yaml
directions:
  - id: A
    name: "Swiss Editorial"
    thesis: "The tower is geometry before it is a landmark."
    visual_language: Architectural
    style: swiss
    layout: poster
    abstraction_level: identity-cue
    palette: [warm ivory, charcoal, muted red]
    typography: "grotesk, strong hierarchy"
    trade_off: "Loses atmosphere; gains structural clarity."
    fit_score: 0.88

  - id: B
    name: "Japanese Documentary"
    thesis: "The tower is a memory of a specific afternoon."
    visual_language: Indie Memory
    style: popeye
    layout: zine
    abstraction_level: relationship-first
    palette: [paper white, faded slate, cobalt anchor]
    typography: "typewriter microtext"
    trade_off: "Loses commercial polish; gains intimacy."
    fit_score: 0.74

selected: A
selection_mode: auto | user
runner_up: B
```

## Fit Score

Score each candidate 0.0–1.0. Not a quality rating — a **fit to brief** rating.

| Weight | Dimension | Question |
|--------|-----------|----------|
| 0.30 | Intent fit | Does the layout sit in `intent.allowed_outputs`? A blocked layout scores 0 overall. |
| 0.25 | Emotion fit | Does the direction match `intent.emotion` and the Image Report `emotion`? |
| 0.20 | Evidence fit | Is the thesis traceable to photo facts, brand cues, or stated goals? |
| 0.15 | Platform fit | Does it survive `intent.platform` — thumbnail scale, copy-safe area, print bleed? |
| 0.10 | Style DNA headroom | Can the style carry the typography and texture the direction wants? |

Auto-commit to the highest score. If the top two are within 0.15 `fit_score` and the table above says "offer," ask. This 0.15 gap is the only quantitative trigger — there is no separate Visual Language score.

## Asking the User

Present directions as a compact table — name, thesis, trade-off. Do not paste the full YAML at the user. Do not generate images for all three unless the user asks; that is a generation-budget decision, not a direction decision.

```
A — Swiss Editorial: geometry before landmark. Structural, cool, typographic.
B — Japanese Documentary: a memory of one afternoon. Quiet, grainy, intimate.
C — Contemporary Museum: the tower as abstract mass. Sparse, tonal, near-textless.

選一個方向，或說「你決定」。
```

If the user says "你決定" / "you pick," commit to the highest `fit_score` and state the thesis in one line before continuing.

## Handoff to Planner

The selected direction becomes a **hard constraint** on the Planner. The Planner sets ratios, typography scale, and recovery plan *within* the direction — it does not re-pick the style.

Write the selected candidate into the spec's `art_direction` block — `id`, `name`, `thesis`, `fit_score`, `selection_mode`, `runner_up` ([../spec/editorial-spec.schema.md](../spec/editorial-spec.schema.md)) — and into the manifest's `art_direction` block ([../spec/visual-manifest.schema.md](../spec/visual-manifest.schema.md)).

The spec's separate `direction:` block belongs to Visual Language and the Planner. Do not write art-direction fields into it.

## Direction Switch Without Re-Analysis

User: "改用 B 那個方向" →

1. Reuse Intent + Image Report unchanged
2. Set `selected: B` in the working list and re-emit the spec's `art_direction` block from candidate B
3. Re-run Planner → Compiler → Adapter → Reviewer
4. Do **not** re-run Analyzer or Visual Language

Same rule as a model switch: the expensive upstream layers are cached.

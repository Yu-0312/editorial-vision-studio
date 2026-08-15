# Art Direction Engine

Run **after** Visual Language, **before** Planner.

**Read `style_gate.outcome` first.** `commit` means the look is already settled — auto-commit and ask nothing, because asking again makes the user choose twice. Only `offer` reaches this layer's question. See [style-gate.md](style-gate.md).

Visual Language derives *one* likely reading of the brief. Art Direction proves it was a choice, not a reflex: draft **2–3 competing directions**, score them, commit to one, and record the runner-up so the user can switch without re-running the pipeline.

## Why This Layer Exists

A single auto-derived direction hides the decision. When the user says "not quite," there is nothing to pivot to and the whole pipeline re-runs. Candidate directions make the pivot a one-line swap.

## When to Offer Choices

| Condition | Behaviour |
|-----------|-----------|
| `outcome: commit`, `reason: freeform` | **Auto-commit** to the direction resolved from `style_gate.description` — see [style-brief.md](style-brief.md). No runner-up: a description commits to one reading. |
| `outcome: commit`, any other reason | **Auto-commit. No candidates, no question.** Name the runner-up in one line when one exists. |
| `outcome: offer` + narrow intent family (Interface Asset, Product / Object) | Offer 2, not 3 — the space is small. |
| `outcome: offer` + theme-only brief, or Editorial Score <50 | Offer 3. The brief underdetermines the look, and reconstruction is interpretive. |
| `outcome: offer`, anything else | Offer 2–3. |

Never offer more than three. Three directions that genuinely differ beat five that are palette swaps.

Every case the gate can produce — preset chosen, free-text look, 「你決定」, explicit `style:`, memory active, series, unattended, too few presets — already resolved to a `commit` or an `offer` before this layer ran. Do not re-derive it here.

**On `reason: freeform` this layer does real work**, not a lookup. It reads `style_gate.description` together with the Image Report and lands both required axes: the photo supplies what is in the scene, the description supplies how it is treated. Cue→axis tables, the negation rule, and the one-clarifying-question cap live in [style-brief.md](style-brief.md). Restate the resolution in one line before generating.

**A committed direction sticks for the session.** After committing on an `offer` run, later images skip the gate with `reason: direction_committed` and arrive here as `commit`. That is what stops 「讓 AI 提案」 re-asking on every image.

## Direction Requirements

**Every candidate must set a different `ground`.** This is the one hard diversity rule, and it exists for a concrete reason: eight of the eleven style DNAs resolve to a light paper field, and only Purple and Period Travel Poster leave neutral territory at all, so candidates chosen on style alone come back as three shades of ivory. Differing grounds force genuinely different images.

Beyond that, each candidate must differ on **at least two** of: render mode, visual language, layout family, abstraction level, typography weight. Two directions that share everything but the accent hue are one direction.

The shipped presets are three different grounds by design — `paper-light`, `saturated`, `full-bleed-photo` — and make useful candidate seeds when a brief is wide open.

Each candidate carries a **thesis** — one sentence naming what it argues the image is *about*. A candidate with no thesis is decoration.

## Candidate Schema

The layer's working output is a `directions[]` list plus a `selected` id. What lands in the spec is the **single `art_direction` block** ([../spec/editorial-spec.schema.md](../spec/editorial-spec.schema.md)), populated from the selected candidate.

```yaml
directions:
  - id: A
    name: "Swiss Editorial"
    thesis: "The tower is geometry before it is a landmark."
    ground: paper-light
    render_mode: graphic
    visual_language: Architectural
    style: swiss
    layout: poster
    abstraction_level: identity-cue
    palette: [warm ivory, charcoal, muted red]
    typography: "grotesk, strong hierarchy"
    trade_off: "Loses atmosphere; gains structural clarity."
    fit_score: 0.88

  - id: B
    name: "Night Fashion"
    thesis: "The tower is a light source, not a structure."
    ground: dark
    render_mode: photographic
    visual_language: Fashion Edge
    style: purple
    layout: magazine-cover
    abstraction_level: identity-cue
    palette: [near-black, sodium amber, cold steel]
    typography: "bold serif headline"
    trade_off: "Loses daylight legibility; gains drama."
    fit_score: 0.79

  - id: C
    name: "Period Poster"
    thesis: "The tower is a destination someone once advertised."
    ground: saturated
    render_mode: graphic
    visual_language: Poster Graphic
    style: travel-poster
    layout: poster
    abstraction_level: full-abstract
    palette: [deep teal, burnt orange, cream]
    typography: "condensed sans wordmark"
    trade_off: "Loses contemporary edge; gains optimism."
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

Ground and render mode are **not** scored. They are diversity constraints on the candidate set, not merits of any one candidate.

Within `offer`, present the top candidates. Within `commit`, take the highest silently. The 0.15 gap describes how close a set is; it never starts a question — `style_gate.outcome` alone decides whether one happens.

## Asking the User

Present directions as a compact table — name, thesis, trade-off. Do not paste the full YAML at the user. Do not generate images for all three unless the user asks; that is a generation-budget decision, not a direction decision.

```
A — Swiss Editorial：先是幾何，才是地標。象牙白紙底，結構、冷、字體主導。
B — Night Fashion：塔是光源，不是結構。深底照片，戲劇性強。
C — Period Poster：塔是某個年代被拿來宣傳的目的地。飽和色場，平面色塊。

選一個方向，或說「你決定」。也可以直接指定 preset。
```

Note the three grounds — ivory, dark, saturated. That is the rule working.

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

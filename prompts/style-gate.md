# Style Gate

Run **immediately after Intent**, before the Analyzer.

Intent may ask one clarifying question before this — about purpose or platform, never about looks. The gate is the first question about what the image should *look like*, and the only one, unless the user asks to see proposals.

Half a step, not a full one: this layer computes nothing. It asks one question, once, and turns the answer into a lock.

## The Problem It Solves

A visual engine that starts working before the user has said what the thing should *look like* will produce whatever its examples produce. In this repo that was warm ivory paper, on every run, forever. Making `design_tokens.ground` required with no default ([../assets/ground.md](../assets/ground.md)) stops the silent fallback; this gate is where the value actually gets decided, out loud, by the person who cares.

## The Invariant

**Nobody is asked to choose a look twice unless they asked to see proposals.**

The gate and the [Art Direction](art-direction.md) question are the same question at different depths. Picking a preset ends it. Picking 「讓 AI 提案」 is an explicit request for a second, better-informed question — that is the option's entire purpose, and it is the only path where a second menu appears.

Describing a look can produce **one narrow follow-up** — naming a single unresolved axis, never the menu again, capped at one per run ([style-brief.md](style-brief.md)). That is a clarification of the user's own words, not a second choice of look.

## The Only Field Art Direction Reads

The gate emits an `outcome`. Art Direction branches on that and nothing else:

```yaml
style_gate:
  outcome: commit | offer      # the only value Art Direction reads
  reason: <see table>          # for the record and the manifest
  preset: ivory-postcard       # null unless a preset was chosen
  description: null            # the user's own words, verbatim, when reason is freeform
```

| `reason` | `outcome` | Why |
|----------|-----------|-----|
| `preset_chosen` | `commit` | User picked from the menu |
| `freeform` | `commit` | User described a look — [style-brief.md](style-brief.md) resolves it. Never answer a description with a menu |
| `deferred` | `commit` | User said 「你決定」/「隨便」 — they opted out of choosing |
| `named_in_request` | `commit` | A preset or style was already in the brief |
| `memory_active` | `commit` | A memory, preset, or series system is already locked |
| `series` | `commit` | The set's system is decided once, at the hero |
| `unattended` | `commit` | Nobody is there — see below |
| `direction_committed` | `commit` | A direction was already committed earlier this session |
| `proposals_requested` | `offer` | User picked 「讓 AI 提案」 |
| `too_few_presets` | `offer` | Under two presets fit the intent; a one-item menu is not a choice, so hand the question to Art Direction |

`outcome: commit` with no preset means Art Direction commits to its highest-fit candidate silently. `outcome: offer` means it presents 2–3.

There is no null state once Intent has run. Every path sets both fields.

## When the Gate Shows a Menu

Only when none of the `commit` reasons above already apply. Concretely: the request names no preset and no style, does not already describe a look in the user's own words, no memory or series is active, no direction has been committed this session, at least two presets fit the intent, and someone is there to answer.

A description that arrives in the opening brief is treated exactly like picking option 4 — `reason: freeform`, captured verbatim, no menu. Answering a description with a menu is the one thing this layer must never do.

## Compatibility Filter

Each preset declares `intended_layouts`. A preset is offered when that list intersects `intent.allowed_outputs`, and its locked `aspect_ratio`, if any, suits `intent.platform`.

Deriving compatibility from `blocked_layouts` instead does not work: `ivory-postcard` blocks only `zine`, so a website-hero brief would pass the filter and be offered a preset whose locked composition — small subject, upper-centre, 55% quiet field — cannot produce a copy-safe hero. `intended_layouts` states the fit positively, so it cannot leak.

Filter silently. Never explain why a preset was withheld.

## Presenting the Choice

Show **looks, not internal vocabulary.** Nobody outside this repo knows what Kinfolk or COS means. Name the ground and the medium in plain words, one line each, in the user's language.

```
要什麼風格？

1. 米色明信片 —— 米白紙底，照片重畫成三到五個簡化色塊，大量留白。安靜、克制。
2. 時代海報 —— 飽和油墨滿版，硬邊平面色塊，地名做成版面的一部分。明亮、圖像感。
3. 紙雕明信片 —— 寫實照片，復古明信片上長出立體紙雕世界，背景散景。1:1。
4. 自己描述 —— 你說想要的樣子，我拆解照片後照你的描述重畫。
5. 讓 AI 提案 —— 我看過你的照片和用途後，給你三個方向再選。

回數字，或直接把想要的樣子講出來。
```

Rules for the menu:

- **Numbered, not lettered.** Users reply with digits.
- **Two escape hatches, always present, always last two**: 自己描述 then 讓 AI 提案. The gate is a shortcut, not a cage.
- **Never more than five options** — at most three compatible presets, plus those two. Trim presets by fit score, never the escape hatches.
- **No YAML, no field names, no star ratings.** Those belong in [../presets/registry.md](../presets/registry.md).
- Picking 自己描述, or simply typing a description instead of a digit, both set `reason: freeform`, `outcome: commit`, and store the words verbatim in `style_gate.description`. Resolution happens later — see [style-brief.md](style-brief.md).

## The 自己描述 Path

The gate **captures and stops**. It does not parse the description, and it does not resolve `ground` or `render_mode` — the Analyzer has not run yet, and the whole promise of this option is that the photo gets deconstructed first and rebuilt to the description afterwards.

```
Gate captures 「想要暗一點、有點像雜誌」
    ↓
Analyzer deconstructs the photo → Image Report
    ↓
Art Direction resolves description + Image Report → one direction
```

The photo supplies **what is in the scene**; the description supplies **how it is treated**. Full cue→axis mapping, negation handling, and the one-follow-up rule: [style-brief.md](style-brief.md).

## Turning the Answer Into a Lock

A chosen preset **becomes the active Visual Memory for the session** ([visual-memory.md](visual-memory.md)):

```yaml
preset: ivory-postcard
memory_id: ivory-postcard
memory_version: "1.0"
style_gate: {outcome: commit, reason: preset_chosen, preset: ivory-postcard}
```

That is what makes the gate ask once rather than once per image: image two sees `memory_id` set and skips with `reason: memory_active`. Same mechanism as any series continuation — no new state.

On the `proposals_requested` path there is no preset to lock, so the **committed direction** does the same job. Once Art Direction commits, later images in the session skip the gate with `reason: direction_committed` and Art Direction auto-commits to the same direction. This is the existing direction-reuse rule ([art-direction.md](art-direction.md)), not a new one — and it is why a session that chose 「讓 AI 提案」 is still asked only once.

The user can change their mind at any point:

| User says | Do |
|-----------|-----|
| 「換成時代海報」 | Swap the active memory, re-run Planner onward. Do not re-analyze. |
| 「這次不要用 preset」 | Clear `preset` for this run only; set `outcome: offer`. |
| 「以後都用時代海報」 | Swap the active memory and keep it for the session. |
| 「重新給我幾個方向」 | Set `outcome: offer`; clear `direction_committed`. |

## What the Gate Does Not Decide

Layout, composition, abstraction level, recovery, aspect ratio, and copy. Those stay with the Planner, per each preset's `free` list. The gate settles **ground and medium** — the two axes with no safe default — and nothing else.

A gate that decided everything would be a template picker, not an art direction engine.

## Unattended Runs

When nobody is there to answer, do not block. Set `reason: unattended`, `outcome: commit`, and pick the best-fitting compatible preset.

Score with **intent-only weights**. The Art Direction fit table cannot be used here — it weights Emotion fit and Evidence fit against the Image Report, and the Analyzer has not run yet.

| Weight | Dimension | Source |
|--------|-----------|--------|
| 0.40 | Layout fit | `intended_layouts` ∩ `intent.allowed_outputs`, as a proportion of allowed outputs |
| 0.30 | Platform fit | Locked `aspect_ratio` and `ground` against `intent.platform` |
| 0.20 | Emotion fit | Preset `atmosphere` against `intent.emotion` |
| 0.10 | Audience fit | Preset register against `intent.audience` |

Open the response with one line naming the choice:

```
沒有指定風格，用「時代海報」preset（活動主視覺 + 印刷輸出最合適）。要換再說。
```

Never silently default to the first preset in the registry. That is how ivory won the first time.

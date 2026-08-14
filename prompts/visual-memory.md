# Visual Memory

Turns a one-off image into a **visual system**. The second image in a project must not restart the pipeline from zero.

Without memory, "now do Tokyo Street" produces a different palette, a different serif, and a different negative-space budget than "Tokyo Tower" did ten minutes ago. With memory, all four images read as one publication.

## What Memory Is

A **locked subset** of a previous run's spec, carried forward as a hard constraint. Not a cache of the whole spec — a deliberate choice about what stays and what is free to move.

```
Run 1: Tokyo Tower      → emit VisualMemory (lock DNA)
Run 2: Tokyo Street     → load VisualMemory → locked fields are constraints
Run 3: Tokyo Museum     → load VisualMemory → same
Run 4: Tokyo Architecture → same
```

Contract: [../spec/visual-memory.schema.md](../spec/visual-memory.schema.md)

## Locked vs. Free

| Field | Default | Why |
|-------|---------|-----|
| `ground` | **locked** | The canvas field. Two images cannot share a system across a paper ground and a dark one |
| `render_mode` | **locked** | Medium is identity. A painterly image and a photograph are not one publication |
| `palette` | **locked** | The single strongest cross-image consistency signal |
| `typography` | **locked** | A changed typeface reads as a different publication |
| `style` | **locked** | Style is the system's identity |
| `visual_language` | **locked** | Downstream of style; unlocking it unlocks everything |
| `texture` tier | **locked** | Mixed PRINT and FLAT grounds never read as a set |
| `atmosphere` | locked | Soft-lock: may shift one step, not invert |
| `abstraction_level` | free | Per-subject; a portrait and a skyline abstract differently |
| `composition` ratios | free | Adapts to each subject's aspect and negative space |
| `layout` | free | Cover, spread, and social crop belong to the same system |
| `aspect_ratio` | free | Set by each output's platform |
| `title` / `subtitle` | free | Per-image copy |
| `recoveries` | free | Driven by each source image's own flags |

Locking composition is a common mistake: it produces four images with the subject in the same corner, which reads as a template, not a system.

**Presets are the exception.** A [preset](../presets/registry.md) (`source: preset`) may lock `composition`, `aspect_ratio`, and `layout`, because a preset is avowedly a template — reproducing one exact look is the whole job. Memories established from a run may not.

## Establishing Memory

Emit memory after the first run **passes QC** — `overall ≥ 0.85` **and** no dimension below 0.60, the same gate [iteration.md](iteration.md) ships on. Do not lock a direction that failed.

```yaml
visual_memory:
  memory_id: tokyo-series
  established_from: tokyo-tower-editorial-r1
  locked:
    ground: paper-light
    render_mode: photo-plus-graphic
    style: swiss
    visual_language: Architectural
    palette: [warm ivory, charcoal, muted red]
    typography: "grotesk title, sans metadata"
    texture_tier: FLAT          # Swiss is Texture ★★ — no SURFACE tokens, see ../assets/texture.md
    atmosphere: quiet contemporary
  blocked_layouts: [zine]        # required whenever layout is free — PRINT would break a FLAT system
  free: [layout, composition, abstraction_level, aspect_ratio, title, recoveries]
  runs: [tokyo-tower-editorial-r1]
```

## Applying Memory

On a continuation request:

1. **Art Direction auto-commits.** The layer still runs and still emits the `art_direction` block — it simply offers no candidates, because the direction is already decided ([art-direction.md](art-direction.md)).
2. **Run Analyzer normally.** Each new photo gets its own Image Report; memory does not suppress analysis.
3. **Planner obeys locks.** It sets free fields only. A locked field the Planner wants to change is a conflict, not a preference.
4. **Reviewer enforces locks.** Add the lock check to the conflict pass ([reviewer.md](reviewer.md)).
5. **Append the run** to `visual_memory.runs`.

## Lock Conflicts

When a new subject genuinely fights the locked DNA:

| Conflict | Resolution |
|----------|------------|
| Locked palette vs. source photo's dominant hue | Keep the lock. Use the photo hue as the anchor only if it is already in the palette; otherwise desaturate it. |
| Locked texture tier vs. a layout needing a different tier (e.g. FLAT lock vs. `zine`) | Reject the layout, not the lock. Zine needs PRINT; a second tier breaks the set. |
| Locked style MUJI vs. a subject needing display type | Reject the type scale. Style DNA caps still apply ([reviewer.md](reviewer.md)). |
| Locked style genuinely wrong for the whole series | **Fork the memory** — do not silently mutate it. New `memory_id`, new `established_from`. |

Never edit a lock mid-series to make one image work. That is how a system becomes four unrelated images.

## Brand Mode

When the user supplies real brand assets, memory is established **from the brand**, not from a generated run:

```yaml
visual_memory:
  memory_id: acme-2026
  established_from: brand_input
  source: user_provided
  locked:
    ground: paper-light
    render_mode: photo-plus-graphic
    palette: [ivory, ink black, signal red]
    typography: "brand grotesk, tight tracking"
    style: cos
    texture_tier: FLAT
  blocked_layouts: [zine]
  free: [layout, composition, abstraction_level, aspect_ratio, title, recoveries]
  hard_constraints:
    - "Never invent a logo mark"
    - "Never alter supplied brand hues"
```

Brand locks are stricter than series locks: no soft-lock, no fork without the user saying so.

## Reporting Memory to the User

State it once, briefly, at the top of a continuation run:

```
沿用 tokyo-series 視覺系統：Swiss / 象牙白＋炭黑＋暗紅 / grotesk 標題。
版面與構圖依這張照片重新決定。
```

Do not restate the full lock table on every image.

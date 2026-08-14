# Prompt Reviewer

Run **after** Model Adapter, **before** image generation. Validates the `GenerationRequest` **against the spec it was compiled from** — it reads the spec to detect drift, but never edits it. Corrections are applied to the request; anything that needs a spec change is a rejection routed back to the Compiler.

## Model-Specific Checks

| Model | Extra check |
|-------|-------------|
| gpt-image + fidelity required | Paragraph 2 contains "preserve" / "exactly" / "no retouching" |
| flux | Prompt ≤3 sentences OR negative_prompt populated |
| ideogram + title present | Title in first sentence, quoted, "legible" clause present |
| any + panter_mode | Saturated anchor described — not "pale accent" |
| any + supplied exact copy | Exact copy is short, quoted, and given a clear placement clause |

## Conflict Matrix

| Combination | Verdict | Fix |
|-------------|---------|-----|
| Swiss + Kinfolk | Warning | Pick one: grid precision OR organic warmth |
| MUJI + heavy headline | Reject | Reduce type to caption scale |
| Brutalist + pastoral soft palette | Warning | Shift palette to concrete/neutral |
| Gallery + dense typography | Reject | Type ≤3% canvas |
| Zine + corporate clean UI | Reject | Add print defects, aged paper |
| Purple Magazine + pastoral Kinfolk | Warning | Split: fashion subject + organic margin only |
| Campaign poster + gallery print layout | Reject | Use [campaign-poster.md](../layouts/campaign-poster.md) |
| Photo-abstract diptych without fidelity clause | Reject | Insert "preserves uploaded photograph exactly" |
| Zine + "pale accent" wording | Reject | Require saturated ink anchor per variation-engine |
| Non-`zine` layout + riso / grain / halftone / xerox / scan-noise wording | Reject | Strip PRINT clause; compensate with contrast, spacing, mark scale ([assets/texture.md](../assets/texture.md)) |
| panter_mode + texture clause on non-`zine` layout | Reject | Panter is colour-only outside `zine` |
| FLAT target (`photo-abstract-diptych` panel, `interface-asset`, `website-hero` copy area, `product-editorial` bg, any `presentation-deck` containing a `data` page) + any texture word | Reject | Remove all texture language; ground stays flat and uniform |
| SURFACE token on a style rated Texture ★★ or lower, or with no material dimension | Warning | Drop to flat matte — style DNA does not support material character |
| COS / MUJI + multiple chroma anchors | Reject | One accent maximum |
| Monocle + brutalist raw concrete | Warning | Choose cosmopolitan OR raw industrial |
| Website hero + no copy-safe space | Reject | Insert copy-safe negative space clause |
| Interface asset + fake UI text | Reject | Remove fake controls/text; use symbolic visual |
| Product editorial + distorted product identity | Reject | Add silhouette/proportion preservation clause |
| `ground` or `render_mode` unset | Reject | Both are required with no default. Send it back to Art Direction rather than letting it fall through to a paper ground |
| `style_gate.outcome: commit` + Art Direction offered candidates | Reject | The user already chose, or opted out. A second menu is a defect ([style-gate.md](style-gate.md)) |
| `style_gate.outcome: offer` + Art Direction committed silently | Reject | The user asked to see options and never got them |
| Prompt describes a ground that contradicts `design_tokens.ground` | Reject | The ground is the one thing a stray "ivory" or "white background" clause silently overwrites |
| `render_mode: photographic` + flat paint / illustration wording | Reject | Pick one medium |
| `render_mode: painterly` or `graphic` + "photorealistic", "8K", depth-of-field wording | Reject | Same, inverted |
| `memory_id` or `preset` set + prompt contradicts a locked field | Reject | Recompile against the lock; never relax it to fit one image ([visual-memory.md](visual-memory.md)) |
| Locked `texture_tier` + a layout requiring a different tier (e.g. FLAT lock + `zine`) | Reject | A second tier breaks the set. Change the layout, not the lock |
| `series_id` set + palette or typeface differs from the hero | Reject | Derivatives inherit the system verbatim ([series.md](series.md)) |
| Presentation deck + riso / scan / halftone wording | Reject | Decks are FLAT or SURFACE only ([../layouts/presentation-deck.md](../layouts/presentation-deck.md)) |

## Style DNA Compatibility

Each style file defines dimension stars (Typography, Geometry, Negative Space, Texture, Color). Reviewer checks Planner choices against style DNA caps:

- MUJI: Typography ★★ max, Geometry ★★ max
- Swiss: Geometry ★★★★★, Negative Space ★★★★★
- Brutalist: Typography ★★★★, Texture ★★★

## Review Checklist

- [ ] No banned adjective fluff in prompt
- [ ] Photo fidelity clause present when photo layout selected
- [ ] Recovery clauses match Image Report flags (no orphan fixes)
- [ ] Single dominant style language
- [ ] High-chroma anchor specified when Panter or zine layout active
- [ ] PRINT-defect wording only when layout is `zine`; CLEAN layouts carry SURFACE tokens at most; FLAT targets carry none
- [ ] Web/interface outputs preserve copy-safe or UI-safe space
- [ ] Product/brand outputs avoid fake logos, fake labels, and distorted identity
- [ ] Supplied in-image copy is short enough to render and has an explicit placement
- [ ] `ground` and `render_mode` are set, and the prompt says the same thing they do
- [ ] Every locked field from an active Visual Memory or preset survives into the prompt
- [ ] Series derivatives carry the hero's palette, typeface, and texture tier unchanged
- [ ] Hard avoids paragraph present

## Output

```yaml
review_status: pass | corrected | rejected
conflicts_found: []
corrections_applied: []
generation_request:
  model: flux
  prompt: "..."
  negative_prompt: "..."
  aspect_ratio: "3:4"
  reference_image: none
  extra_params: {}
```

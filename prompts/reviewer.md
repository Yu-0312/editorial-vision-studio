# Prompt Reviewer

Run **after** Model Adapter, **before** image generation. Validates `GenerationRequest`, not raw EditorialSpec.

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
| FLAT target (`photo-abstract-diptych` panel, `interface-asset`, `website-hero` copy area, `product-editorial` bg) + any texture word | Reject | Remove all texture language; ground stays flat and uniform |
| SURFACE token on a style rated Texture ★★ or lower, or with no material dimension | Warning | Drop to flat matte — style DNA does not support material character |
| COS / MUJI + multiple chroma anchors | Reject | One accent maximum |
| Monocle + brutalist raw concrete | Warning | Choose cosmopolitan OR raw industrial |
| Website hero + no copy-safe space | Reject | Insert copy-safe negative space clause |
| Interface asset + fake UI text | Reject | Remove fake controls/text; use symbolic visual |
| Product editorial + distorted product identity | Reject | Add silhouette/proportion preservation clause |

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

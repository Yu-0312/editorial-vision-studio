# Quick Decision Tree

```
START: User request + optional image / brand / product / theme
│
├─ Asking for a SET (N pages, all sizes, carousel)?
│   └─ YES → prompts/series.md → run Intent…Art Direction ONCE
│            → hero first, QC, then fan out derivatives
│
├─ Continuing an existing look ("同一套視覺", brand assets)?
│   └─ YES → prompts/visual-memory.md → load locked DNA
│            → Art Direction auto-commits (no candidates offered)
│            → Analyzer still runs per photo
│
├─ No image, theme/text only?
│   └─ YES → Intent: Visual Concept / Zine / Campaign / Brand / Web
│            → optional Variation Engine → Compiler (skip pixel Analyzer)
│
└─ Has image
    │
    ├─ Step 0: Intent Engine → output family + allowed layouts
    │
    ├─ Step 1: Analyzer → Image Report + Editorial Score
    │
    ├─ Score < 50?
    │   └─ YES → Visual Language: Indie Memory / Architectural
    │            → editorial_mode: reconstruction
    │
    ├─ Score 50–69 OR panter_mode flag?
    │   └─ YES → editorial_mode: compensation
    │            → load recovery/ modules (start with contrast.md)
    │
    ├─ Score 70–89?
    │   └─ YES → editorial_mode: standard
    │
    └─ Score 90+?
        └─ YES → editorial_mode: premium (minimal recovery)
    │
    ├─ Step 2: Visual Language Engine → derive style/layout/palette
    │   (override if user said style: X)
    │
    ├─ Step 3: Art Direction → 2–3 candidates → commit one, keep runner-up
    │   auto-commit if: style override · narrow family · memory active
    │   offer choices if: close scores · theme-only · score <50
    │
    ├─ Step 4: Planner → layout + composition ratios (inside the direction)
    │
    ├─ Step 5: Recovery → apply flagged modules only
    │
    ├─ Step 6: Compiler Phase 1 → VisionSpec / EditorialSpec
    │   (validate: spec/editorial-spec.schema.md)
    │
    ├─ Step 6b: Compiler Phase 2 → Model Adapter
    │   resolve target.model → adapters/{model}.md → GenerationRequest
    │
    ├─ Step 7: Reviewer → validate GenerationRequest + memory locks
    │
    ├─ Generate image (unless prompt-only)
    │
    ├─ Step 8: Evaluator → quality vector → lowest_failing + responsible_layer
    │
    ├─ overall ≥ 0.85 and no dimension <0.60?
    │   ├─ YES → finalize VisualManifest → report to user
    │   └─ NO  → Step 9: Iteration → one spec mutation → re-enter at Step 6
    │            escalation picks WHICH field; entry is always Step 6
    │            stop at 3 passes, on regression, or when escalated to Intent
    │
    └─ First passing run of a set? → establish VisualMemory
```

## Reuse Shortcuts

| User says | Entry point |
|-----------|-------------|
| "改用 Flux" | Adapter (Step 6b) |
| "改用 B 那個方向" | Planner (Step 4) |
| "一模一樣再生一次" | Generate |
| "同一套視覺，換主題" | Analyzer (Step 1), with memory locks applied |

Never re-enter above the shortcut. The manifest already holds everything upstream.

## Subject Shortcuts

| Subject | First layout to consider |
|---------|--------------------------|
| person + big sky | magazine-cover |
| building | poster (swiss) |
| landscape | gallery-print |
| street | zine or poster |
| food/product | editorial-spread |
| event name in request | campaign-poster |
| brand launch | brand-key-visual |
| website / SaaS / app | website-hero or interface-asset |
| social post / story | social-asset |
| theme-only concept | moodboard or zine |
| multi-page slide set | presentation-deck |

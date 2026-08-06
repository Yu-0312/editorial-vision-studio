# Quick Decision Tree

```
START: User request + optional image / brand / product / theme
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
    ├─ Step 3: Planner → layout + composition ratios
    │
    ├─ Step 4: Recovery → apply flagged modules only
    │
    ├─ Step 5: Compiler Phase 1 → VisionSpec / EditorialSpec
    │   (validate: spec/editorial-spec.schema.md)
    │
    ├─ Step 5b: Compiler Phase 2 → Model Adapter
    │   resolve target.model → adapters/{model}.md → GenerationRequest
    │
    ├─ Step 6: Reviewer → validate GenerationRequest
    │
    ├─ Generate image (unless prompt-only)
    │
    └─ Step 7: Evaluator → grade A–D → report to user
```

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

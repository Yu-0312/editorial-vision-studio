# Style Brief

Turns a user's free-text description of a look into a spec. The fourth Style Gate option: **你說想要的樣子，我拆解照片後照你的描述重畫。**

A description is not a spec. 「想要暗一點、有點像雜誌」 names a feeling, not a `ground`, a `render_mode`, or a palette. This layer does that translation — and refuses to guess where the description is silent.

## Where It Runs

The [Style Gate](style-gate.md) **captures** the description verbatim and stops there. Resolution happens later, at [Art Direction](art-direction.md), with the Image Report already in hand.

```
Style Gate      capture description verbatim → style_gate.description
    ↓
Visual Analyzer deconstruct the photo → Image Report
    ↓
Art Direction   description + Image Report → one committed direction
```

That order is the whole point. The photo is deconstructed **first**, then rebuilt to the description — not filtered toward it. Resolving the description at the gate would throw away the facts the Analyzer is about to produce.

With no photo (theme-only brief), the description carries the run alone and the Analyzer is skipped as usual.

## Photo Supplies Facts, Description Supplies Treatment

The division that makes this work:

| From the photo (Image Report) | From the description |
|-------------------------------|----------------------|
| What is in the scene, and how the parts relate | Ground and medium |
| Subject, geometry, negative space | Palette temperature and chroma |
| Existing contrast, saturation, clarity | Typography weight and presence |
| Composition strengths and weaknesses | Atmosphere and register |

**The description wins on treatment; the photo wins on content.** A night photograph described as 「明亮、輕快」 becomes a bright image of that same scene — the engine is reconstructing, not correcting the exposure. A photo of a bridge never becomes a portrait because the description said 「人像感」; that is a content claim the photo cannot support, and it is a rejection.

## Cue → Axis

Both required axes must land. These tables are the mapping, not an exhaustive lexicon — read the intent behind the words.

### Ground

| Cue words | → `ground` |
|-----------|-----------|
| 暗、深色、夜、低調、沉、moody, dark, night | `dark` |
| 白、乾淨、留白、紙感、素、clean, paper, airy, minimal | `paper-light` |
| 鮮豔、飽和、撞色、大色塊、bold, saturated, punchy, colour-blocked | `saturated` |
| 照片感、滿版、真實場景、photographic, full-bleed, real | `full-bleed-photo` |
| 雙色、單色調、兩個顏色、duotone, two-tone | `duotone` |
| 灰、水泥、中性、冷靜、concrete, grey, neutral | `neutral-gray` |

### Render mode

| Cue words | → `render_mode` |
|-----------|----------------|
| 寫實、照片、實拍、photographic, realistic, shot | `photographic` |
| 插畫、手繪、水彩、繪本、painterly, illustrated, gouache, hand-drawn | `painterly` |
| 平面、扁平、向量、幾何、色塊、flat, vector, graphic, geometric | `graphic` |
| 上照片下插畫、照片加圖形、拼貼、collage, photo plus graphic | `photo-plus-graphic` |
| Two of the above named for different regions | `mixed` — and name the zones |

### Style and register

| Cue words | → `style` |
|-----------|----------|
| 雜誌感、編輯感、editorial | Depends on the rest: 冷靜 → swiss, 溫暖 → kinfolk, 時裝 → purple |
| 日系、無印、極簡、Japanese, MUJI | `muji` |
| 北歐、性冷淡、材質感、Scandinavian | `cos` |
| 復古、懷舊、老海報、vintage, retro | `travel-poster` |
| 街頭、青春、城市、street, youthful | `popeye` |
| 粗獷、工業、水泥、brutalist, raw | `brutalist` |
| 溫暖、生活感、居家、homely, lived-in | `apartamento` |
| 高級、時裝、戲劇、fashion, dramatic | `purple` |

Full DNA: [../styles/](../styles/). Prompt prose for the two axes: [../assets/ground.md](../assets/ground.md).

## Negative Descriptions

「不要太亮」 is a constraint, not a value. Never let a negation set an axis by itself.

1. Add the negation to `avoids` verbatim, in concrete visible terms
2. Then choose a **positive** value for the axis it constrains, and say which

「不要太亮」 → `avoids: [blown highlights, high-key wash]` **and** `ground: neutral-gray` (stated), never `ground: not-paper-light`.

## When the Description Is Silent

If either required axis is still unresolved after parsing, ask **one narrow question naming only the missing axis**:

```
了解 —— 雜誌感、克制。底色想要哪種？

1. 米白紙底  2. 深色底  3. 照片滿版
```

This is a clarification, not a second choice of look. Rules:

- **One follow-up per run, maximum.** If it is still ambiguous after that, take the highest-fit reading and say which in one line.
- **Name only the unresolved axis.** Never re-show the preset menu — the user already declined it.
- **Never ask about both axes at once.** If both are silent, the description was too thin to be a brief; treat it as `proposals_requested` and hand the run to Art Direction candidates instead.

## Output

The Style Gate emits the description untouched:

```yaml
style_gate:
  outcome: commit
  reason: freeform
  preset: null
  description: "想要暗一點、有點像雜誌，不要太花"
```

Art Direction resolves it into a normal `art_direction` block whose `thesis` restates the description as a claim about the image:

```yaml
art_direction:
  id: A
  name: "Quiet Night Editorial"
  thesis: "The scene is a held breath after dark."
  fit_score: 0.86
  selection_mode: user
  runner_up: null          # freeform commits to one reading; there is no runner-up
```

`selection_mode: user` — the user chose this, in their own words.

## Restating Before You Build

Echo the resolution back in one line before generating, so a misread is caught before the pixels, not after:

```
照你的描述：深色底、寫實照片、Swiss 編輯排版、克制的字。照片內容照原本的，只是重新處理。
```

One line. Do not paste the spec.

## Saving It

If the result passes QC (`overall ≥ 0.85`, no dimension below 0.60) and the user wants more like it, offer once:

```
要把這組存成 preset 以後直接用嗎？
```

On yes, write it as a `source: preset` VisualMemory ([../presets/_template.md](../presets/_template.md)) with `intended_layouts` set from what it just produced. This is how the preset library grows from real use instead of authoring.

## Guardrails

**Never:**

- Set an axis from a negation alone
- Let the description override what the photograph contains
- Answer a description with the preset menu
- Ask more than one clarifying question

**Always:**

- Deconstruct the photo before applying the description
- Land both required axes explicitly, or ask about exactly one
- Restate the resolution in one line before generating
- Record the description verbatim in the manifest, so the run is replayable

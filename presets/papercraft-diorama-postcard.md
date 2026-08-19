# Papercraft Diorama Postcard — Preset

A photoreal 1:1 social image: a vintage postcard lying in the foreground with a layered paper-cut world rising out of it, a collectible traveller figure standing inside that world, and the real destination dissolving into bokeh behind.

Ground is the **photograph itself** — the third of the three shipped grounds, and the only preset here with `render_mode: photographic`.

Requires a `[destination]` value. Everything else is derived from it.

## Locked DNA

```yaml
memory_version: "1.0"
memory_id: papercraft-diorama-postcard
version: "1.0"
established_from: preset
source: preset

locked:
  ground: full-bleed-photo
  render_mode: photographic
  style: wallpaper                      # design-forward, cosmopolitan, product-friendly
  visual_language: Product Stillness
  layout: social-asset                  # presets may lock layout; this one is 1:1 social by definition
  intended_layouts: [social-asset]
  abstraction_level: identity-cue
  aspect_ratio: "1:1"                   # locked, unlike the other presets
  palette: derived-from-destination     # climate and culture of [destination], not a fixed set
  typography: "handwritten postcard script, destination-specific; postmark and stamp as objects"
  texture_tier: FLAT                    # the canvas ground is a photograph; depicted paper is content, not canvas texture
  atmosphere: warm natural light, cinematic, quietly controlled
  composition:
    camera: slightly elevated three-quarter view
    depth_order: [blurred real destination, sharp postcard, paper-cut landscape, traveller figure]
    focus: postcard tack-sharp, background creamy bokeh

free: [title, subtitle, recoveries]

hard_constraints:
  - "The postcard stays tack-sharp; only the real-world background is blurred"
  - "One to three iconic landmarks maximum — never a crowd of attractions"
  - "The paper-cut elements emerge from the postcard surface, physically continuous with it"
  - "Handwritten copy is unique to the destination, never generic"
```

## Photo Policy

```yaml
photo_policy:
  fidelity: none
  reference_image: none
  source_region: full-bleed
```

No photograph is supplied — the model generates one. `ground: full-bleed-photo` is satisfied here by `render_mode: photographic`, which is the second of the two routes the spec allows ([../spec/editorial-spec.schema.md](../spec/editorial-spec.schema.md)).

## Destination Adaptation Rules

| Element | Derived from |
|---------|--------------|
| Traveller's clothing | Local climate and culture |
| Architecture in the paper-cut | Local building vernacular |
| Landmark selection | 1–3 iconic features, no more |
| Colour grade | The destination's own light |
| Handwritten quote | Unique to that place |

Emphasise the secret, dreamlike, storybook register. Avoid generic tourist scenes and crowds.

## Copy Slots

```
Title       [destination], [country]
Body        有些地方不求關注 —— 它們只是留在你心中。
Closing     去往地圖靜止的地方。
```

Keep exact in-image text short; image models are unreliable with long strings. If the model garbles the body line, drop it to the postcard's handwriting layer as illegible script rather than lengthening the prompt.

## Avoids

Crowds and generic tourist landmarks · flat lighting · a blurred postcard · paper-cut elements floating free of the postcard surface · more than three landmarks · invented logos · watermark · mockup device frame

## Compiler anchor

```
Square photoreal image, slightly elevated three-quarter view. A vintage
postcard of [destination] lies in the foreground, tack-sharp, with rounded
corners, visible paper stock, a stamp, a postmark, and handwritten travel
notes. A layered paper-cut world rises out of the postcard surface and is
physically continuous with it: cut-paper terrain, local vernacular
architecture, one to three iconic landmarks, vegetation, coastline. A
stylised collectible traveller figure stands inside that world in local
climate-appropriate dress, expressive pose, realistic proportions, small
narrative props. Behind everything, the real [destination] falls away into
creamy bokeh. Warm natural light, shallow depth of field, realistic shadows,
colour grade drawn from the destination's own light. Handwritten line reads
"[quote]". Avoid crowds, generic tourist landmarks, flat lighting, a blurred
postcard, floating paper elements, watermark, mockup frame.
```

## Provenance

This preset was written from the author's own known-good prompt, kept below unedited so the preset's origin is auditable. It is **reference material, not an alternative path** — the compiler anchor above is what runs. The anchor drops `8K`, `masterpiece`, `premium`, `luxury`, and `杰作`, which [../prompts/compiler.md](../prompts/compiler.md) bans because they do not change pixels, and the [Reviewer](../prompts/reviewer.md) rejects any request that reintroduces them.

```text
[目的地]： 一张高度详细的写实主义风格、适合Instagram发布的[目的地]旅行明信片立体模型，从略高的角度俯视。一张真实的复古明信片位于前景，带有圆角、纸张质感、邮票、手写旅行笔记和邮戳。从明信片中浮现出一个复杂的3D纸雕世界，展示了[目的地]最具标志性的元素，由层叠的剪纸地形、建筑、地标、植被和海岸线构建而成。

一个风格化的收藏级旅行者角色自然地站在纸雕场景中，其设计旨在匹配目的地的氛围和文化。该角色具有高端电影感造型、表现力十足的姿势、写实比例、时尚的旅行装束、微妙的叙事配件，并无缝地融入这个缩微世界。

背景是[目的地]柔焦模糊的现实场景，创造了深度以及现实与想象之间的联系。明信片保持清晰锐利，而背景则具有奶油般的虚化效果。

明信片上的手写文字带有个人色彩、诗意且具有目的地特色，表达了发现、惊奇和秘境旅行。优雅的邮戳、目的地印章和细致的水墨插画补充了构图，而不会显得凌乱。

视觉层级：模糊的真实目的地 → 真实的明信片 → 浮现的纸雕景观 → 旅行者角色。

奢侈旅行社论审美、温暖的自然光、浅景深、优质纸张质感、电影级调色、写实阴影、手工细节、超精细的纸雕结构、收藏级人偶品质、旅游杂志封面品质、极具社交媒体分享性的美感、写实照片、8K、杰作。

目的地特定适配规则：
角色服装灵感来自当地气候和文化。
建筑反映当地建筑风格。
地标选择限于1-3个标志性特征。
色调灵感来自目的地。
手写引言对目的地而言是独一无二的。
纸雕元素从明信片表面自然浮现。
避免通用的旅游景点和拥挤的场景。
强调秘境、梦幻、童话般的旅行感。

标题 [目的地]，[国家]
段落 有些地方不求关注 —— 它们只是留在你心中。
结束语 去往地图静止的地方。
纵横比 1:1
```

## Notes

`texture_tier: FLAT` is not a contradiction of all the paper in this image. The tier governs the **canvas ground**, and this preset's canvas ground is a photograph — `ground: full-bleed-photo` is always FLAT ([../assets/texture.md](../assets/texture.md)). The postcard stock and cut-paper layers are *objects inside the photograph*: subject matter, not canvas texture. No PRINT-tier defect language is permitted, and Wallpaper\* would not qualify for SURFACE in any case (no material dimension in its DNA).

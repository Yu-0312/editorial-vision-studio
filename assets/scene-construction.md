# Scene Construction

Rules for **space**. The rest of `assets/` governs surface — ground, palette, texture, typography. Nothing governed how objects sit in relation to each other, and that gap is what produces a sticker sheet instead of a scene.

## The Failure This Prevents

Simplify a photograph without these rules and you get objects that are individually correct and collectively wrong: a chair drawn in side elevation next to a table drawn top-down, both floating on a clean ground with no shadow, arranged in a row that has nothing to do with how they stood in the room.

Every mark is defensible. The image is not. It reads as clip art.

The photograph already solved the spatial problem. Abstraction should keep that solution and discard only the detail.

## The Four Rules

### 1. One projection

Choose **one** viewpoint and hold it across every object in the frame: flat elevation, three-quarter, or top-down. Never mix.

The common break is a single object rendered from two angles at once — a round table drawn as a top-down ellipse sitting on a front-view pedestal. Each half looks right; the object is impossible.

- Flat elevation is the safest for `graphic` and `painterly` — it reads as a deliberate convention rather than a mistake
- Deep perspective recession and converging vanishing points are usually wrong for a reduced image, but that is **not** a licence to drop projection altogether
- «Avoid perspective» in a prompt means *avoid deep recession*, never *avoid a consistent viewpoint*. Say the projection you want; do not only say what to avoid

### 2. One ground plane

Everything stands on the same surface, at the same eye level. Objects do not float.

- Give the plane a visible edge, a horizon, or an implied line where the objects' bases align
- If the composition is genuinely floating — a moodboard, a specimen study — say so explicitly. Floating by decision is fine; floating by omission is the defect

### 3. One light, and contact

A single light direction, stated. Every object that touches the ground gets a **contact shadow** in that direction.

This is the highest-leverage rule in the file. A cast shadow does four jobs at once: it fixes the light, fixes the ground, fixes the time of day, and ties separate objects into one scene. Its absence is why simplified images look weightless.

- One shared direction. Two objects with opposite shadows is a rejection
- The shadow can be a single flat shape — it does not need modelling
- On `ground: paper-light` the shadow is the same family as the ground, one or two steps darker, never black

### 4. Relative position and scale survive

Whatever the photograph said about *where things are* and *how big they are relative to each other*, the reduction keeps.

- Chairs that surrounded a table still surround it. They do not become a row
- Overlap that existed is preserved — occlusion is spatial information, not clutter
- A person beside a tree stays that fraction of the tree's height
- Absolute scale, framing, and crop are free. **Relations are not**

## Form Count Is About Kinds, Not Instances

A cap like «three to five simplified forms» is a rule about **vocabulary**, not about how many objects may appear.

- Correct reading: three to five *kinds* of form — tree, figure, bench, shadow — repeated as often as the scene needs
- Wrong reading: at most five objects total, so drop everything else

The wrong reading is what turns a courtyard of chairs into three isolated pieces of furniture. Write the constraint as `form_types: 3-5`, and never let it delete a relationship rule from above.

## Mark Quality: `painterly` vs `graphic`

Both are flat-colour modes, which is why they collapse into each other. The difference is at the edges and inside the shapes:

| | `painterly` | `graphic` |
|---|-------------|-----------|
| Edge | Slightly irregular, brush- or dab-made, minutely uneven | Mathematically clean, uniform weight |
| Inside a shape | Slight tonal variation, visible mark direction, occasional gaps | One flat value, edge to edge |
| Repeated elements | Each instance differs a little | Instances are identical |
| Reads as | Made by a hand | Made by a tool |

A `painterly` spec that comes back with uniform vector edges is a `render_mode` failure, not a stylistic near-miss. Name the mark-making explicitly in the prompt — «dabbed», «brush-made», «slightly uneven edges» — or the model will default to vector cleanliness because that is what «flat colour» usually means to it.

## Compiler Clauses

Emit these after the ground clause and before the palette clause ([../prompts/compiler.md](../prompts/compiler.md)):

```
Single consistent {projection} viewpoint across every object — never two
angles within one object. All objects rest on one shared ground plane at a
common eye level. One light from {direction}; every object casts a flat
contact shadow in that direction. Preserve the source's relative positions,
relative scale, and overlaps; only detail is removed.
```

For `render_mode: painterly`, add:

```
Marks are dabbed and brush-made with slightly uneven edges and faint tonal
variation inside each shape; repeated elements differ slightly from one
another. No vector-clean outlines.
```

## Avoids

Objects floating with no contact shadow · two viewpoints inside one object · converging vanishing points on a reduced image · a row of isolated objects where the source had a cluster · identical repeated instances on a `painterly` spec · shadows in conflicting directions

## When These Rules Do Not Apply

`moodboard` and `interface-asset` are legitimately non-spatial — fragments on a field, symbols with no world. State that as a decision in the spec rather than letting it happen by default.

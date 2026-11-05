# Block catalog

Every step has `blocks` (the text and media the learner reads, in order) and an optional `visual` (the
step's main illustration). The portal shows the visual pinned beside the text in scrollytelling, split
screen in the stepper and slides, and inline in document mode.

- `blocks` accepts: `markdown`, `callout`, `image`, `code`, `math`, `table`, `quote`, `chart`, `diagram`, `animation`,
  `simulator`, `scene3d`, `hotspots`, `flashcards`.
- `visual` accepts: `chart`, `diagram`, `animation`, `image`, `simulator`, `scene3d`, `hotspots`.
- Unknown keys are rejected everywhere. Optional `caption` fields are shown under the element.

All examples below pass `coursmith validate`.

## Step skeleton

```json step
{
  "id": "why-high-voltage",
  "title": "Why high voltage?",
  "visual": { "type": "diagram", "syntax": "mermaid", "code": "flowchart LR\n  A[Plant] --> B[Lines] --> C[Homes]" },
  "blocks": [
    { "type": "markdown", "text": "Wires waste energy as heat, so the grid raises the [[voltage]]." }
  ]
}
```

## Text blocks

### markdown

Paragraphs, lists, **bold**, *italic*, `inline code` and links. Rules:

- No raw HTML (`<br>`, `<b>`, `<script>`…). No Markdown images: use an `image` block.
- Links must start with `https://`.
- Mark glossary terms as `[[term]]` or `[[term|text shown]]`. The term must be defined in the glossary of
  this section or an earlier one (its `term` or one of its `aliases`, case-insensitive).
- Keep one idea per block; prefer several short blocks to one long one.

```json block
{
  "type": "markdown",
  "text": "A [[transformer]] changes the [[voltage|voltage level]] of alternating current.\n\n- Step-up: raises it for transmission\n- Step-down: lowers it for homes\n\nMore on [Wikipedia](https://en.wikipedia.org/wiki/Transformer)."
}
```

### callout

`variant` is one of `info`, `tip`, `warning`, `example`. `title` is optional.

```json block
{ "type": "callout", "variant": "warning", "title": "Safety first", "text": "Never open an electrical panel yourself." }
```

### quote

```json block
{ "type": "quote", "text": "Nothing in life is to be feared, it is only to be understood.", "cite": "Marie Curie" }
```

### code

`language` is a lowercase highlighter name such as `python`, `ts`, `js`, `bash`, `sql`, `json`, `c++`, `c#`.

```json block
{ "type": "code", "language": "python", "code": "power_kw = 2\nhours = 1.5\nprint(power_kw * hours)  # 3.0 kWh", "caption": "Energy = power × time" }
```

### math

LaTeX rendered with KaTeX. Escape backslashes in JSON (`\\frac`).

```json block
{ "type": "math", "tex": "E = P \\times t", "caption": "Energy equals power times time." }
```

### table

`headers` has 1–10 columns; every row must have exactly as many cells as `headers`. Up to 50 rows.

```json block
{
  "type": "table",
  "headers": ["Device", "Power", "Energy in 2 h"],
  "rows": [["Oven", "2 kW", "4 kWh"], ["LED bulb", "10 W", "0.02 kWh"]],
  "caption": "Typical household devices"
}
```

## Media blocks and visuals

### image

Refers to an entry of `assets` by id. Raster images (`png`, `jpg`, `webp`) and `svg` are accepted.

```json visual
{ "type": "image", "asset": "home-panel", "caption": "A typical home electrical panel" }
```

The matching asset entry:

```json asset
{ "id": "home-panel", "kind": "svg", "file": "assets/home-panel.svg", "alt": "Electrical panel with meter, main switch and four breakers", "generatedBy": "hand-written SVG" }
```

### chart (Vega-Lite)

A [Vega-Lite v6](https://vega.github.io/vega-lite/) spec. Rules:

- Data must be inline in `data.values`. Any `url` (data or images) and any `href` are rejected.
- Keep it small (well under 200 KB): aggregate before charting.
- Give axes readable titles in the course language; label illustrative numbers as such in the `caption`.
- Leave size and colors to the portal's theme (light and dark) unless a color carries meaning.

```json visual
{
  "type": "chart",
  "title": "Share of household consumption",
  "caption": "Illustrative figures.",
  "spec": {
    "data": { "values": [
      { "use": "Heating", "share": 45 },
      { "use": "Appliances", "share": 30 },
      { "use": "Lighting", "share": 10 },
      { "use": "Other", "share": 15 }
    ] },
    "mark": { "type": "arc", "innerRadius": 50 },
    "encoding": {
      "theta": { "field": "share", "type": "quantitative" },
      "color": { "field": "use", "type": "nominal", "title": "Use" }
    }
  }
}
```

Line and bar charts follow the usual Vega-Lite shape (`mark: "line"` / `"bar"`, `x` and `y` encodings).

### diagram (Mermaid)

`syntax` is always `"mermaid"`. The first line must be one of these diagram types: `flowchart`, `graph`,
`sequenceDiagram`, `classDiagram`, `stateDiagram`, `stateDiagram-v2`, `erDiagram`, `gantt`, `pie`,
`journey`, `mindmap`, `timeline`, `quadrantChart`, `xychart-beta`.
Not allowed: `click` handlers, links (`href`) and `%%{init: ...}%%` directives.

```json visual
{
  "type": "diagram",
  "syntax": "mermaid",
  "code": "sequenceDiagram\n  participant P as Plant\n  participant G as Grid operator\n  P->>G: Produces 500 MW\n  G-->>P: Reduce output",
  "caption": "The operator keeps supply and demand balanced."
}
```

### animation

An animation is a preset rendered by the portal from parameters. See `animation-presets.md` for all eight.

```json visual
{
  "type": "animation",
  "caption": "From the plant to your home",
  "animation": {
    "preset": "process",
    "steps": [
      { "title": "Generation", "description": "A turbine spins a generator." },
      { "title": "Transmission", "description": "High-voltage lines carry the power." },
      { "title": "Distribution", "description": "Local lines deliver 230 V." }
    ]
  }
}
```

## Interactive blocks

Learners remember what they manipulate. Use these where the idea is about **change** (a value that depends on
others), **space** (a shape, a structure, a position) or **recall** (terms to learn by heart). Everything is
declarative: you describe the element, the portal draws it; no code runs from a course.

### simulator

Sliders (`controls`) drive live values (`outputs`) and an optional live line `chart`. Formulas use the control
`id`s (and, in chart series, the `x.id` variable) with `+ - * / % ^`, comparisons, `&&`, `||`, `!`,
`cond ? a : b`, the constants `pi` and `e`, and these functions: `sin cos tan asin acos atan atan2 sqrt cbrt abs
exp ln log10 log2 pow floor ceil round sign hypot min max clamp deg rad` (angles in radians; `rad(30)` converts).
Every control needs `min < value < max`; give `step` for whole numbers. At most 6 controls, 6 outputs, 4 series.

```json visual
{
  "type": "simulator",
  "title": "How much does an appliance cost to run?",
  "controls": [
    { "id": "watts", "label": "Power", "min": 5, "max": 3000, "step": 5, "value": 1000, "unit": "W" },
    { "id": "hours", "label": "Use per day", "min": 0.5, "max": 24, "step": 0.5, "value": 2, "unit": "h" },
    { "id": "price", "label": "Price", "min": 0.05, "max": 0.6, "step": 0.01, "value": 0.25, "unit": "€/kWh" }
  ],
  "outputs": [
    { "label": "Energy per day", "formula": "watts * hours / 1000", "unit": "kWh", "decimals": 2 },
    { "label": "Cost per month", "formula": "watts * hours / 1000 * price * 30", "unit": "€", "decimals": 2 }
  ],
  "chart": {
    "x": { "id": "day", "label": "Days", "from": 0, "to": 365, "points": 60 },
    "yLabel": "Total cost (€)",
    "series": [{ "label": "Cost", "formula": "watts * hours / 1000 * price * day" }]
  },
  "caption": "Move the sliders: cost grows with both power and time."
}
```

Ideas: physics laws (Ohm, free fall, pendulum), compound interest, population growth, dose–response, chemical
equilibria, unit conversions, probability of independent events. State the formula in the text too, so the
simulator illustrates it rather than replacing the explanation.

### scene3d

A 3D scene built from primitives the learner can rotate (drag) and zoom (pinch or scroll). Shapes: `box`,
`sphere`, `cylinder`, `cone`, `torus`, `plane`. `size` is width, height, depth (a sphere's diameter is the
width; a torus uses width as outer diameter and depth as tube thickness). `rotation` is in degrees. Optional
`label` text floats above an object; `motion` animates it: `spin`, `orbit` (around the origin), `bob`, `pulse`.
`arrows` draw vectors. Up to 60 objects and 30 arrows. Use `#rrggbb` colors.

```json visual
{
  "type": "scene3d",
  "title": "A water molecule",
  "objects": [
    { "shape": "sphere", "size": [1.4, 1.4, 1.4], "color": "#ef4444", "label": "Oxygen" },
    { "shape": "sphere", "position": [1.1, -0.8, 0], "size": [0.8, 0.8, 0.8], "color": "#f1f5f9", "label": "Hydrogen" },
    { "shape": "sphere", "position": [-1.1, -0.8, 0], "size": [0.8, 0.8, 0.8], "color": "#f1f5f9", "label": "Hydrogen" },
    { "shape": "cylinder", "position": [0.55, -0.4, 0], "rotation": [0, 0, 54], "size": [0.15, 1.3, 0.15], "color": "#94a3b8" },
    { "shape": "cylinder", "position": [-0.55, -0.4, 0], "rotation": [0, 0, -54], "size": [0.15, 1.3, 0.15], "color": "#94a3b8" }
  ],
  "camera": { "position": [0, 1, 6] },
  "autoRotate": true,
  "caption": "The two hydrogen atoms sit at about 104.5° from each other."
}
```

Ideas: molecules, crystal lattices, planets and orbits, geometric solids and their sections, vectors and forces,
simple machines, the layers of the Earth. Keep scenes small and label what the text mentions. On phones the
scene is about 340 px wide: avoid tiny objects that only make sense when zoomed.

### hotspots

Numbered points on an image (any image asset); tapping a point shows its title and text. `x` and `y` are
percentages from the image's left and top edges. Up to 12 spots.

```json visual
{
  "type": "hotspots",
  "asset": "home-panel",
  "spots": [
    { "x": 50, "y": 15, "title": "Meter", "text": "Counts the kilowatt-hours you use." },
    { "x": 50, "y": 58, "title": "Residual current device", "text": "Cuts the power if current leaks, for example through a person." }
  ],
  "caption": "Explore the panel."
}
```

Ideas: parts of a machine, an anatomy diagram, a map, a painting's details, a user interface.

### flashcards

Cards with a front and a back; the learner taps to flip, and can shuffle. 2–24 cards. A good way to review the
terms of a section right before the quiz (they are not scored).

```json block
{
  "type": "flashcards",
  "title": "Units to remember",
  "cards": [
    { "front": "Volt (V)", "back": "Unit of voltage: the electrical push." },
    { "front": "Ampere (A)", "back": "Unit of current: how much charge flows per second." },
    { "front": "Kilowatt-hour (kWh)", "back": "Energy used by 1,000 watts for one hour." }
  ]
}
```

## Questions

### check (end of a step, not scored)

Types `single`, `multiple`, `true_false`. Option ids are short lowercase strings such as `a`, `b`, `c`.

```json check
{
  "id": "check-transformer",
  "type": "single",
  "prompt": "What does a transformer change?",
  "options": [
    { "id": "a", "text": "The voltage" },
    { "id": "b", "text": "The frequency" },
    { "id": "c", "text": "The colour of the wires" }
  ],
  "correct": "a",
  "explanation": "Transformers raise or lower voltage; frequency stays the same."
}
```

### final quiz questions

Same as checks plus `sectionRef` (the id of the section the question tests) and the `ordering` type.

```json question
{
  "id": "q-multiple-renewables",
  "sectionRef": "generation",
  "type": "multiple",
  "prompt": "Which of these sources are renewable?",
  "options": [
    { "id": "a", "text": "Wind" },
    { "id": "b", "text": "Coal" },
    { "id": "c", "text": "Sunlight" }
  ],
  "correct": ["a", "c"],
  "explanation": "Wind and sunlight replenish naturally; coal is a fossil fuel."
}
```

```json question
{
  "id": "q-true-false-frequency",
  "sectionRef": "generation",
  "type": "true_false",
  "prompt": "The European grid runs at 50 Hz.",
  "correct": true,
  "explanation": "European grids use 50 Hz; North America uses 60 Hz."
}
```

```json question
{
  "id": "q-ordering-journey",
  "sectionRef": "generation",
  "type": "ordering",
  "prompt": "Put the journey of electricity in order.",
  "items": [
    { "id": "plant", "text": "Power plant" },
    { "id": "lines", "text": "Transmission lines" },
    { "id": "home", "text": "Home" }
  ],
  "explanation": "Electricity is generated, transmitted at high voltage, then distributed to homes."
}
```

## Glossary entries

```json glossary
{ "term": "kilowatt-hour", "aliases": ["kilowatt-hours", "kWh"], "definition": "The energy used by a 1 kW device running for one hour; the unit on electricity bills." }
```

## Course metadata

`category` is one of: `technology`, `programming`, `data-ai`, `science`, `mathematics`, `engineering`,
`business`, `finance`, `marketing`, `design`, `arts-music`, `languages`, `history`, `humanities`,
`health-wellness`, `environment`, `law-policy`, `personal-development`, `education`, `other`.

```json meta
{
  "title": "Come arriva l'elettricità a casa tua",
  "subtitle": "Dalla centrale alla presa, passo dopo passo",
  "description": "Segui il viaggio dell'energia elettrica: produzione, trasmissione ad alta tensione, rete di distribuzione e quadro elettrico di casa.",
  "language": "it",
  "difficulty": "beginner",
  "category": "engineering",
  "tags": ["energia", "rete elettrica"],
  "estimatedMinutes": 30,
  "learningObjectives": ["Spiegare come funziona un generatore", "Descrivere il ruolo dei trasformatori"],
  "prerequisites": [],
  "coverImage": "cover",
  "accentColor": "#2563eb"
}
```

## Limits

| Item | Limit |
|---|---|
| `course.json` | 2 MB |
| One asset file | 3 MB; all assets 25 MB |
| Sections | 1–12, each 1–15 steps, 80 steps in total |
| Blocks per step | 1–20 |
| Glossary per section | 1–40 entries |
| Final quiz | 10–40 questions |
| Markdown block | 20,000 characters |

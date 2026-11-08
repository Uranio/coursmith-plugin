# Block catalog

Every step has `blocks` (the text and media the learner reads, in order) and an optional `visual` (the
step's main illustration). The portal shows the visual pinned beside the text in scrollytelling, split
screen in the stepper and slides, and inline in document mode.

- `blocks` accepts: `markdown`, `callout`, `image`, `code`, `math`, `table`, `quote`, `chart`, `diagram`, `animation`,
  `simulator`, `scene3d`, `hotspots`, `flashcards`, `molecule`, `dynamics`, `worked_example`, `exercise`,
  `details`, `tabs`, `scenario`, `playground`.
- `visual` accepts: `chart`, `diagram`, `animation`, `image`, `simulator`, `scene3d`, `hotspots`, `molecule`,
  `dynamics`.
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
- Formulas inside text go between double dollars: `$$E = mc^2$$` (inline, LaTeX). A single `$` stays a dollar
  sign. Chemistry uses mhchem: `$$\\ce{2H2 + O2 -> 2H2O}$$`. Escape backslashes in JSON.
- Question prompts, options, explanations and blank texts are plain text, but `$$…$$` formulas work there too.

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

Chemical equations: `\\ce{...}` (mhchem) handles formulas, charges, arrows and states.

```json block
{ "type": "math", "tex": "\\ce{CH4 + 2O2 -> CO2 + 2H2O}", "caption": "Combustion of methane." }
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
`sphere`, `cylinder`, `cone`, `torus`, `plane`, `capsule`, `ring`, `tetrahedron`, `octahedron`, `icosahedron`.
`size` is width, height, depth (a sphere's diameter is the width; a torus uses width as outer diameter and depth
as tube thickness; a ring uses width as outer diameter and depth as its band). `rotation` is in degrees. Optional
`label` text floats above an object; `motion` animates it: `spin`, `orbit` (around the origin), `bob`, `pulse`.
`arrows` draw vectors; `lines` draw polylines (orbits, trajectories, field lines; `closed` joins the ends). Up to
60 objects, 30 arrows, 20 lines. Use `#rrggbb` colors.

Make scenes explorable:

- `info` on an object: the learner taps the part and reads what it is and does.
- `explode` on objects (an offset): an "Exploded view" slider moves the parts apart, for machines and layers.
- `tour`: 2–10 guided stops; the camera glides to each `position`/`target` while its `title` and `text` show.

For molecules prefer the `molecule` block below, which places atoms and bonds for you.

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

```json visual
{
  "type": "scene3d",
  "title": "Inside the Earth",
  "objects": [
    { "shape": "sphere", "size": [2, 2, 2], "color": "#f59e0b", "label": "Core", "info": "Iron and nickel, about 5,000 °C." },
    { "shape": "sphere", "size": [3.4, 3.4, 3.4], "color": "#ef4444", "opacity": 0.35, "label": "Mantle", "info": "Hot rock that flows very slowly.", "explode": [0, 1.5, 0] },
    { "shape": "sphere", "size": [4, 4, 4], "color": "#3b82f6", "opacity": 0.2, "wireframe": true, "label": "Crust", "info": "The thin solid shell we live on.", "explode": [0, 3, 0] }
  ],
  "lines": [{ "points": [[-3, 0, 0], [3, 0, 0]], "color": "#94a3b8", "label": "Diameter ≈ 12,700 km" }],
  "tour": [
    { "title": "The whole planet", "position": [0, 2, 9] },
    { "title": "The core", "text": "Solid inner core, liquid outer core.", "position": [0, 0.5, 4] }
  ],
  "caption": "Tap a layer, or open the exploded view."
}
```

Ideas: crystal lattices, planets and orbits, geometric solids and their sections, vectors and forces, simple
machines (exploded), the layers of the Earth, an organ or a cell (tap the parts). Keep scenes small and label what
the text mentions. On phones the scene is about 340 px wide: avoid tiny objects that only make sense when zoomed.

### molecule

A 3D molecule from atoms and bonds; learners switch between ball-and-stick, space-filling and sticks and can show
atom labels. `position` is in ångström (use real geometry: C–H 1.09 Å, C–C 1.54 Å, O–H 0.96 Å; tetrahedral
109.5°). `bonds` refer to atoms by index (0-based); `order` is 1, 2 or 3. Colors follow the CPK convention.
Up to 300 atoms.

```json visual
{
  "type": "molecule",
  "title": "Water (H₂O)",
  "atoms": [
    { "element": "O", "position": [0, 0, 0] },
    { "element": "H", "position": [0.757, 0.586, 0] },
    { "element": "H", "position": [-0.757, 0.586, 0] }
  ],
  "bonds": [{ "a": 0, "b": 1 }, { "a": 0, "b": 2 }],
  "caption": "The bent shape (104.5°) makes water polar."
}
```

```json block
{
  "type": "molecule",
  "title": "Carbon dioxide",
  "atoms": [
    { "element": "C", "position": [0, 0, 0] },
    { "element": "O", "position": [1.16, 0, 0] },
    { "element": "O", "position": [-1.16, 0, 0] }
  ],
  "bonds": [{ "a": 0, "b": 1, "order": 2 }, { "a": 0, "b": 2, "order": 2 }],
  "style": "ball-and-stick"
}
```

### dynamics

A system that evolves over time: each variable has an `initial` value and a `rate` (how fast it changes, i.e. its
derivative). The portal integrates it (Runge–Kutta), plays it as an animation, and recomputes when the learner
moves a `controls` slider. `rate` formulas use the controls, every variable and `t`; `initial` uses the controls.
Same formula language as the simulator. 1–4 variables; `chart.phase` (two variables) adds a phase portrait.

```json visual
{
  "type": "dynamics",
  "title": "An epidemic (SIR model)",
  "controls": [
    { "id": "beta", "label": "Contacts × infectivity", "min": 0.1, "max": 1, "step": 0.01, "value": 0.3 },
    { "id": "gamma", "label": "Recovery rate", "min": 0.02, "max": 0.5, "step": 0.01, "value": 0.1, "unit": "/day" }
  ],
  "variables": [
    { "id": "s", "label": "Susceptible", "initial": "0.99", "rate": "-beta * s * i" },
    { "id": "i", "label": "Infected", "initial": "0.01", "rate": "beta * s * i - gamma * i" },
    { "id": "r", "label": "Recovered", "initial": "0", "rate": "gamma * i" }
  ],
  "time": { "to": 160, "unit": "days", "label": "Time" },
  "chart": { "yLabel": "Share of the population", "yMin": 0, "yMax": 1 },
  "caption": "Lower the contacts and watch the peak flatten."
}
```

Ideas: falling with air resistance (`v' = g - k*v^2`), a pendulum or spring (position and velocity, with
`phase`), radioactive decay, charging a capacitor, predator and prey, compound interest, drug concentration,
cooling (Newton's law). Write the equations in the text too.

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

## Exercises and worked examples

Practice is where learning sticks. In maths, physics, chemistry, biology and any skill with a method, show one
**worked example**, then give an **exercise** of the same kind.

### worked_example

A solved problem the learner reveals step by step (each step can have a formula in `tex`), ending with a `result`.

```json block
{
  "type": "worked_example",
  "title": "How long does the kettle take?",
  "problem": "A 2,000 W kettle heats 1 kg of water from 20 °C to 100 °C. Water needs $$4{,}186\\ \\text{J}$$ per kg per °C. How long does it take?",
  "steps": [
    { "text": "Energy needed: mass × specific heat × temperature change.", "tex": "Q = 1 \\times 4186 \\times 80 = 334{,}880\\ \\text{J}" },
    { "text": "Power is energy per second, so time is energy divided by power.", "tex": "t = \\frac{334{,}880}{2000} \\approx 167\\ \\text{s}" }
  ],
  "result": "About **2 minutes 47 seconds** (a little more in practice, because some heat is lost)."
}
```

### exercise

A problem the learner solves: `parts` are 1–6 questions of any type (see Questions below), shown as (a), (b)…
with immediate feedback. `hints` are revealed one at a time on request; `solution` steps are offered after a
first try. `level` is `easy`, `medium` or `hard`. Give every exercise hints or a solution.

```json block
{
  "type": "exercise",
  "title": "Free fall",
  "level": "medium",
  "problem": "A stone is dropped from a bridge and hits the water after 3 s. Ignore air resistance and use $$g = 9.81\\ \\text{m/s}^2$$.",
  "parts": [
    { "id": "speed", "type": "numeric", "prompt": "How fast is it going when it hits the water?", "answer": 29.43, "tolerancePercent": 1, "unit": "m/s", "explanation": "v = g·t = 9.81 × 3 = 29.43 m/s." },
    { "id": "height", "type": "numeric", "prompt": "How high is the bridge?", "answer": 44.1, "tolerancePercent": 1, "unit": "m", "explanation": "h = ½·g·t² = 0.5 × 9.81 × 9 ≈ 44.1 m." },
    { "id": "law", "type": "expression", "prompt": "Write the height h as a function of the time t (use g).", "answer": "g * t^2 / 2", "variables": ["g", "t"], "domain": [1, 10], "explanation": "Distance under constant acceleration from rest: h = ½·g·t²." }
  ],
  "hints": ["Speed grows by g every second.", "The distance is the area under the speed–time line: a triangle."],
  "solution": [
    { "text": "Speed after t seconds:", "tex": "v = g t = 9.81 \\times 3 = 29.43\\ \\text{m/s}" },
    { "text": "Distance fallen:", "tex": "h = \\tfrac{1}{2} g t^2 = 0.5 \\times 9.81 \\times 3^2 \\approx 44.1\\ \\text{m}" }
  ]
}
```

```json block
{
  "type": "exercise",
  "title": "Balance the combustion of propane",
  "level": "easy",
  "problem": "Propane burns in oxygen and gives carbon dioxide and water.",
  "parts": [
    {
      "id": "balance",
      "type": "chem_balance",
      "prompt": "Balance the equation.",
      "species": [
        { "formula": "C3H8", "side": "reactant", "coefficient": 1 },
        { "formula": "O2", "side": "reactant", "coefficient": 5 },
        { "formula": "CO2", "side": "product", "coefficient": 3 },
        { "formula": "H2O", "side": "product", "coefficient": 4 }
      ],
      "explanation": "3 carbons give 3 CO₂, 8 hydrogens give 4 H₂O; that is 10 oxygen atoms, so 5 O₂."
    }
  ],
  "hints": ["Balance carbon first, then hydrogen, and oxygen last."]
}
```

Subject ideas:

- **Maths**: `expression` (simplify, expand, differentiate: any equivalent form is accepted), `numeric` (solve,
  compute), `ordering` (steps of a proof), `fill_blanks` (complete a derivation).
- **Physics**: `numeric` with `unit` and `tolerancePercent` (1–2% absorbs rounding), `expression` for laws
  (`variables` such as `["m", "a"]`), a `simulator` or `dynamics` to explore before computing.
- **Chemistry**: `chem_balance`, `numeric` for moles, masses and pH, `fill_blanks` with `caseSensitive` for
  formulas and symbols, `categorize` (acids/bases, metals/non-metals), `molecule` to see the shape.
- **Biology**: `image_point` (tap the mitochondrion), `matching` (organelle → function), `categorize`
  (prokaryote/eukaryote), `fill_blanks` with `caseSensitive` for DNA (complementary strand), `ordering` (phases of
  mitosis), `hotspots` to explore first.

## Structure blocks

### details

A collapsible section, closed by default: proofs, history, extra depth, a long example. Contains simple blocks
(`markdown`, `callout`, `image`, `code`, `math`, `table`, `quote`).

```json block
{
  "type": "details",
  "title": "Why is it ½·g·t²? (proof)",
  "blocks": [
    { "type": "markdown", "text": "Speed grows linearly, $$v = g t$$, so the distance is the area of the triangle under the speed line." },
    { "type": "math", "tex": "h = \\tfrac{1}{2} \\cdot t \\cdot g t = \\tfrac{1}{2} g t^2" }
  ]
}
```

### tabs

2–6 tabs, each with simple blocks: the same idea in several languages or notations, cases side by side.

```json block
{
  "type": "tabs",
  "tabs": [
    { "label": "Python", "blocks": [{ "type": "code", "language": "python", "code": "def energy(kw, hours):\n    return kw * hours" }] },
    { "label": "JavaScript", "blocks": [{ "type": "code", "language": "js", "code": "const energy = (kw, hours) => kw * hours;" }] }
  ]
}
```

### scenario

A branching story: the learner makes decisions and sees their consequences. `nodes` have `text` and up to 4
`choices` (each with `next` and optional `feedback`); a node without choices is an ending with `ending`:
`success`, `failure` or `neutral`. Every node must be reachable from `start`, and an ending must be reachable.
Ideal for safety, first aid, customer service, management, ethics, diagnosis, negotiation.

```json block
{
  "type": "scenario",
  "title": "A smell of burning",
  "start": "smell",
  "nodes": [
    {
      "id": "smell",
      "text": "You smell burning plastic near the kitchen socket. The kettle is plugged in.",
      "choices": [
        { "text": "Unplug the kettle by pulling the cable", "next": "cable", "feedback": "Pulling the cable can damage it further." },
        { "text": "Switch off the circuit at the panel", "next": "panel", "feedback": "Cutting the power first is the safe move." }
      ]
    },
    { "id": "cable", "text": "The plug is hot and the cable insulation is melted. You got a small shock.", "ending": "failure" },
    { "id": "panel", "text": "The power is off. You can now inspect the socket safely and call an electrician.", "ending": "success" }
  ]
}
```

### playground

Code the learner edits and runs in the browser: `javascript` or `python` (Python downloads about 10 MB on first
run). `tests` are expressions evaluated after the code (true means passed), e.g. `add(2, 3) == 5`. Optional
`instructions` and `solution`. The code runs offline in a sandbox: no network, no files; output is what it prints
(`print` / `console.log`). Time limit: 5 s (JavaScript), 15 s (Python).

```json block
{
  "type": "playground",
  "title": "Compute the energy",
  "language": "python",
  "instructions": "Complete `energy_kwh` so it returns the energy in kWh used by a device of `watts` running for `hours`.",
  "code": "def energy_kwh(watts, hours):\n    # your code here\n    return 0\n\nprint(energy_kwh(2000, 1.5))",
  "tests": [
    { "name": "A 2,000 W oven for 1.5 h uses 3 kWh", "check": "abs(energy_kwh(2000, 1.5) - 3) < 1e-9" },
    { "name": "A 10 W bulb for 10 h uses 0.1 kWh", "check": "abs(energy_kwh(10, 10) - 0.1) < 1e-9" }
  ],
  "solution": "def energy_kwh(watts, hours):\n    return watts * hours / 1000"
}
```

## Questions

Eleven types, usable everywhere a question goes: a step's `check` (practice, not scored), the `parts` of an
`exercise`, and the final quiz (which adds `sectionRef`). Every question has `id`, `prompt` and `explanation`.
Option and item ids are short lowercase strings such as `a`, `b`, `c`.

| Type | The learner… | Answer key |
|---|---|---|
| `single` | picks one option | `correct`: option id |
| `multiple` | picks all correct options | `correct`: option ids |
| `true_false` | says true or false | `correct`: boolean |
| `ordering` | puts items in order | `items` listed in the correct order (shuffled for learners) |
| `matching` | matches each left item with a right item | `pairs` (the right column is shuffled) |
| `categorize` | puts each item in one of 2–4 groups | each item's `category` |
| `fill_blanks` | completes `{{blank-id}}` gaps in a text, typing or choosing | each blank's accepted `answers` |
| `numeric` | types a number (3,5 · 1/4 · 6.02e23 are fine) | `answer`, with `tolerance` or `tolerancePercent` |
| `expression` | types a formula (`2x + 1`, `(x+1)^2`) | `answer`: any equivalent expression is accepted |
| `image_point` | taps a spot on an image | `targets`: x, y and `radius` in percent |
| `chem_balance` | writes the coefficients of an equation | `coefficient` of each species (any balanced set is accepted) |

### check (end of a step, not scored)

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

```json check
{
  "id": "check-units",
  "type": "matching",
  "prompt": "Match each quantity with its unit.",
  "pairs": [
    { "id": "voltage", "left": "Voltage", "right": "Volt (V)" },
    { "id": "current", "left": "Current", "right": "Ampere (A)" },
    { "id": "energy", "left": "Energy", "right": "Kilowatt-hour (kWh)" }
  ],
  "explanation": "Volts measure the push, amperes the flow, kilowatt-hours the energy used."
}
```

```json check
{
  "id": "check-sources",
  "type": "categorize",
  "prompt": "Renewable or not?",
  "categories": [{ "id": "renewable", "label": "Renewable" }, { "id": "fossil", "label": "Fossil" }],
  "items": [
    { "id": "wind", "text": "Wind", "category": "renewable" },
    { "id": "coal", "text": "Coal", "category": "fossil" },
    { "id": "sun", "text": "Sunlight", "category": "renewable" },
    { "id": "gas", "text": "Natural gas", "category": "fossil" }
  ],
  "explanation": "Wind and sunlight replenish naturally; coal and gas took millions of years to form."
}
```

`fill_blanks`: put `{{id}}` in `text` for each blank. Without `options` the learner types; case, accents and extra
spaces are ignored unless `caseSensitive` (set it for chemical formulas, symbols and DNA). With `options` the
learner picks from a drop-down.

```json check
{
  "id": "check-dna",
  "type": "fill_blanks",
  "prompt": "Complete the sentences.",
  "text": "In DNA, adenine pairs with {{a}} and cytosine pairs with {{c}}. The complementary strand of ATGC is {{strand}}.",
  "blanks": [
    { "id": "a", "answers": ["thymine", "T"], "options": ["thymine", "uracil", "guanine"] },
    { "id": "c", "answers": ["guanine", "G"] },
    { "id": "strand", "answers": ["TACG"], "caseSensitive": true }
  ],
  "explanation": "A–T and C–G: each base pairs with its complement, so ATGC becomes TACG."
}
```

`numeric`: learners may type `3,5`, `1/4`, `6.02×10^23` or `2pi`. Without a tolerance only the exact value is
accepted: give `tolerancePercent` (relative) or `tolerance` (absolute) for anything computed.

```json check
{
  "id": "check-ohm",
  "type": "numeric",
  "prompt": "A 230 V heater draws 10 A. What is its power?",
  "answer": 2300,
  "tolerancePercent": 1,
  "unit": "W",
  "explanation": "P = V × I = 230 × 10 = 2,300 W."
}
```

`expression`: the learner's formula is compared with `answer` at random points of `domain` (default [-5, 5]),
so `x^2 - 1` and `(x-1)(x+1)` are both right. List the allowed `variables`. Learners may write `2x`, `x²`,
`3(x+1)`. For logarithms and roots use a positive domain such as `[0.5, 5]`.

```json check
{
  "id": "check-derivative",
  "type": "expression",
  "prompt": "What is the derivative of $$x^3 + 2x$$?",
  "answer": "3*x^2 + 2",
  "variables": ["x"],
  "explanation": "Differentiate term by term: (x³)′ = 3x² and (2x)′ = 2."
}
```

`image_point`: `x`, `y` and `radius` are percentages of the image (from the left and top edges). Up to 3 targets
(any of them is right).

```json check
{
  "id": "check-meter",
  "type": "image_point",
  "prompt": "Tap the meter.",
  "asset": "home-panel",
  "targets": [{ "x": 50, "y": 15, "radius": 10 }],
  "explanation": "The meter sits at the top of the panel and counts the kilowatt-hours."
}
```

`chem_balance`: formulas like `H2O`, `Ca(OH)2`, `CuSO4·5H2O`, ions `SO4^2-`, `Fe^3+`, electrons `e^-`. The
validator checks that your coefficients balance atoms and charge; learners get credit for any balanced set.

```json check
{
  "id": "check-water",
  "type": "chem_balance",
  "prompt": "Balance the formation of water.",
  "species": [
    { "formula": "H2", "side": "reactant", "coefficient": 2 },
    { "formula": "O2", "side": "reactant", "coefficient": 1 },
    { "formula": "H2O", "side": "product", "coefficient": 2 }
  ],
  "explanation": "Two O atoms on the left need two H₂O on the right, which need two H₂."
}
```

### final quiz questions

Any of the types above, plus `sectionRef` (the id of the section the question tests).

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

```json question
{
  "id": "q-numeric-bill",
  "sectionRef": "home",
  "type": "numeric",
  "prompt": "A 1,000 W heater runs 3 hours a day for 30 days at 0.25 €/kWh. How much does it cost?",
  "answer": 22.5,
  "tolerance": 0.05,
  "unit": "€",
  "explanation": "1 kW × 3 h × 30 = 90 kWh; 90 × 0.25 = 22.50 €."
}
```

Mix types in the final quiz: recall (single, true_false, matching) and application (numeric, expression,
fill_blanks, chem_balance, image_point).

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

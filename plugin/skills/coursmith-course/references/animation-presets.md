# Animation presets

Animations are declared, not coded: you choose a preset and give it parameters, and the portal renders it
with consistent motion and styling (and shows a static version to learners who prefer reduced motion).
Use an animation block as a step's `visual` or among its `blocks`:

```json
{ "type": "animation", "caption": "Optional caption", "animation": { "preset": "…", "…": "…" } }
```

Pick the preset that matches the idea, not the one that looks busiest:

| Preset | Shows | Use it for |
|---|---|---|
| `sequence` | Items appearing one after another | Lists of parts, ingredients, features, principles |
| `flow` | Nodes connected by arrows with moving particles | Causes and effects, data or energy flowing through a system |
| `timeline` | Dated events along a line | History, project phases, evolution of an idea |
| `process` | Numbered stages revealed in order | Procedures and pipelines with a clear order |
| `compare` | Before/after slider | Contrasting two states, a wrong vs right approach |
| `counter` | Numbers counting up to a value | Orders of magnitude, key statistics |
| `highlight-svg` | Your SVG with parts highlighted one by one | Anatomy of an object, parts of a diagram or interface |
| `svg-motion` | Your SVG with elements moving, turning, fading or changing color over time | Mechanisms, cycles, trajectories, anything that moves |

All examples below pass `coursmith validate`.

## sequence

- `layout`: `row` (default), `column` or `grid`.
- `items`: 2–12, each `label` (≤ 80 chars), optional `description` (≤ 240) and `icon` (a single emoji).

```json animation
{
  "preset": "sequence",
  "layout": "grid",
  "items": [
    { "label": "Sun", "icon": "☀️", "description": "Solar panels convert light." },
    { "label": "Wind", "icon": "🌬️", "description": "Turbines convert moving air." },
    { "label": "Water", "icon": "💧", "description": "Dams convert falling water." }
  ]
}
```

## flow

- `direction`: `LR` (left to right, default) or `TB` (top to bottom).
- `nodes`: 2–12, each with an `id` (lowercase-dash) and `label` (≤ 60).
- `edges`: 1–24, each `from` and `to` (node ids) and an optional short `label` (≤ 40).

```json animation
{
  "preset": "flow",
  "direction": "LR",
  "nodes": [
    { "id": "sun", "label": "Sunlight" },
    { "id": "panel", "label": "Solar panel" },
    { "id": "inverter", "label": "Inverter" },
    { "id": "home", "label": "Home" }
  ],
  "edges": [
    { "from": "sun", "to": "panel" },
    { "from": "panel", "to": "inverter", "label": "DC" },
    { "from": "inverter", "to": "home", "label": "AC" }
  ]
}
```

## timeline

- `events`: 2–16, each `date` (free text ≤ 40, e.g. `1882`, `Q3 2024`, `Day 1`), `title` (≤ 80), optional `description`.

```json animation
{
  "preset": "timeline",
  "events": [
    { "date": "1831", "title": "Faraday discovers induction" },
    { "date": "1882", "title": "First public power station", "description": "Pearl Street, New York." },
    { "date": "1990s", "title": "Grid interconnection across Europe" }
  ]
}
```

## process

- `steps`: 2–10, each `title` (≤ 80) and optional `description` (≤ 240). Stages are numbered automatically.

```json animation
{
  "preset": "process",
  "steps": [
    { "title": "Measure", "description": "Read the meter at the start of the month." },
    { "title": "Subtract", "description": "Subtract it from the reading at the end." },
    { "title": "Multiply", "description": "Multiply the kWh by the price per kWh." }
  ]
}
```

## compare

- `before` and `after`: each a `label` (≤ 60) plus `text` (≤ 400) and/or `asset` (an image or svg asset id).
  At least one of `text` or `asset` is required on each side.

```json animation
{
  "preset": "compare",
  "before": { "label": "Incandescent bulb", "text": "60 W for 800 lumens; most energy becomes heat." },
  "after": { "label": "LED bulb", "text": "9 W for the same 800 lumens." }
}
```

## counter

- `items`: 1–6, each `label`, `to` (target number), optional `from` (default 0), `unit` (≤ 12 chars) and
  `decimals` (0–4, default 0).

```json animation
{
  "preset": "counter",
  "items": [
    { "label": "Transmission line", "to": 380000, "unit": "V" },
    { "label": "Home socket", "to": 230, "unit": "V" },
    { "label": "Efficiency", "to": 93.5, "unit": "%", "decimals": 1 }
  ]
}
```

## highlight-svg

- `asset`: an asset of kind `svg`.
- `highlights`: 1–12, each `elementId` (the `id` attribute of an element in the SVG; letters, digits, `_`
  and `-`, starting with a letter), `label` and optional `description`. Elements are highlighted in order.
- Group each highlighted part in a `<g id="...">` so the whole part lights up. The portal removes scripts,
  event handlers, `<foreignObject>`, `<image>`, `<use>` and external links, but keeps ids; if an id you
  reference is missing after that cleanup, the submission is sent back with `animation.unknown_element`.

```json animation
{
  "preset": "highlight-svg",
  "asset": "home-panel",
  "highlights": [
    { "elementId": "meter", "label": "Meter", "description": "Measures the energy you use." },
    { "elementId": "breakers", "label": "Circuit breakers", "description": "Protect each circuit." }
  ]
}
```

## svg-motion

A keyframed animation of your own SVG: the most flexible preset. Draw the scene as an SVG asset, give an `id` to
each element (usually a `<g>`) that moves, then describe its keyframes. The learner can play, pause and restart.

- `asset`: an asset of kind `svg`. Same cleanup rules as `highlight-svg`; every `elementId` must survive it.
- `tracks`: 1–20, each an `elementId` and 2–12 `keyframes`. A keyframe has `at` (seconds, 0–30, increasing
  within a track) and any of: `x`, `y` (offset in SVG units), `rotate` (degrees, around the element's center),
  `scale`, `opacity` (0–1), `fill` (`#rrggbb`). Values a keyframe leaves out carry over from the previous one.
- `loop` (default `true`) and `labels`: captions shown from their `at` time on, to narrate what happens.
- The animation lasts until the latest `at` across tracks. Keep it short (5–15 s) and let labels explain it.

```json animation
{
  "preset": "svg-motion",
  "asset": "home-panel",
  "loop": true,
  "tracks": [
    { "elementId": "meter", "keyframes": [{ "at": 0, "opacity": 0.3 }, { "at": 1.5, "opacity": 1, "scale": 1.08 }, { "at": 3, "scale": 1 }] },
    { "elementId": "breakers", "keyframes": [{ "at": 3, "opacity": 0.3 }, { "at": 4.5, "opacity": 1 }, { "at": 6, "opacity": 1 }] }
  ],
  "labels": [
    { "at": 0, "text": "Electricity enters through the meter…" },
    { "at": 3, "text": "…then splits into circuits, each protected by a breaker." }
  ]
}
```

# Writing guide

The review agent checks accuracy, clarity, language, difficulty, glossary, quiz and visuals. Courses that
follow this guide usually pass on the first submission.

## Structure

- **Sections** are chapters: 3–6 for a 30-minute course, each with a clear purpose stated in its `summary`.
- **Steps** are single ideas: 80–250 words of text plus one visual. If a step needs two visuals or more
  than one heading, split it.
- Order steps so each one builds on the previous ones. Never use a concept before the step that introduces it.
- Open a section with why it matters; close it with a short recap (a `callout` of variant `info` works well).
- Add a `check` question to the steps that introduce the key ideas: it keeps learners active in stepper mode.

## Difficulty levels

| Level | Assume | Write |
|---|---|---|
| `beginner` | No prior knowledge of the field | Everyday language, analogies, one new term at a time, concrete examples before abstractions, simple numbers |
| `intermediate` | The basics of the field | Correct technical terms, cause–effect reasoning, worked examples with real numbers, common pitfalls |
| `advanced` | Solid working knowledge | Precise terminology, trade-offs and edge cases, formulas and derivations where useful, references to standards or primary sources |

Declare prerequisites in `meta.prerequisites` for intermediate and advanced courses.

## Accuracy

- State only what you are confident is correct and current. When experts disagree, say so.
- Use real, sourced figures when you cite statistics, and say where they come from (for example
  "Eurostat, 2023") in the text or caption. If numbers are invented to illustrate a point, say
  "illustrative figures" in the caption.
- Check every calculation, unit and date. Keep units consistent (SI unless the field uses another convention).
- Avoid absolute claims ("always", "never") unless they are true.
- For health, legal and financial topics, add a `callout` disclaimer pointing learners to qualified
  professionals, and do not give individual advice.

## Language

- Write every learner-facing string in the course language (`meta.language`): titles, text, captions, chart
  axis titles, alt text, glossary, quiz, explanations. Keep proper names and code as they are.
- Use one term per concept throughout the course; if you introduce a synonym, add it to `aliases`.
- Short sentences, active voice, direct address ("you").

## Glossary

- Every section has a glossary. Include every concept the section **introduces** and every technical,
  scientific, legal or domain-specific word it **uses**, including acronyms.
- A definition is one or two sentences, self-contained (it does not require reading the step), correct, and
  at the course's difficulty level. Do not define a term with itself.
- Mark the term in the text where it is first used in that section, as `[[term]]` or `[[term|shown text]]`.
- Terms defined in an earlier section can be marked later without redefining them. Do not define the same
  term twice in one section.
- Add plurals, inflections and abbreviations to `aliases` (e.g. `"aliases": ["kWh", "kilowatt-hours"]`).
- The review agent is strict, and these are the gaps it flags most often:
  - everyday words that are domain terms in context ("grid", "power", "energy", "panel", "electron");
  - terms that appear only in visuals: chart axes, diagram nodes, animation and highlight labels;
  - technical words used inside another term's definition (if "voltage" is defined with "circuit",
    define "circuit" too);
  - units and their symbols (kW, kWh, Hz, Ω).
- Before submitting, read each section and list every word a newcomer to the topic might not know. Each one
  must be in this section's glossary or an earlier one.

## Visuals

- Every visual must teach something the text refers to; mention it in the text ("the chart shows…").
- Charts: one message per chart, labelled axes with units, sorted bars, at most about 8 categories.
- Diagrams: at most about 10 nodes; short labels.
- Pick the animation preset that matches the idea (see `animation-presets.md`).
- Alt text describes what the image shows and why it matters, in the course language
  ("Bar chart: gas provides 40% of electricity, the largest share"), not "image" or the file name.
- Avoid text inside images; when unavoidable, keep it large and in the course language.

## Phones first

Most learners follow courses on a phone (about 360 px wide). The player adapts the layout, but the content must
read well on a small screen too:

- **Steps**: one idea, about 80–180 words. Short paragraphs (2–4 sentences); use lists for sequences.
- **Charts**: never set `width`; prefer horizontal bars when category names are long; at most about 8
  categories; category labels under about 16 characters; legends with few entries.
- **Diagrams**: up to about 8 nodes with labels of 1–3 words. Left-to-right flowcharts are drawn top-down on
  phones automatically, so keep them simple. Sequence diagrams: at most 4 participants. Avoid `gantt` and
  `journey` (they scroll sideways); use a `timeline` animation instead.
- **Tables**: at most 4 columns and short cells; split a wide table in two, or turn it into a list.
- **Images and SVG**: give SVGs a `viewBox` and no fixed `width`/`height`; text inside an image must stay
  readable when the image is 340 px wide (about 14 px or larger at that size). Prefer portrait or square
  images for tall content.
- **Formulas and code**: keep lines short (about 40 characters); break long formulas into steps.

`coursmith validate` reports layouts that do not fit a phone as `mobile.*` warnings: fix them unless the
content really needs the space.

## Final quiz

- At least 10 questions; spread them so every section is tested, roughly in proportion to its length.
- Test understanding, not memorized wording: ask learners to apply, compare or explain.
- Exactly one correct answer for `single`; for `multiple`, say "Select all that apply" in the prompt.
- Distractors are plausible and similar in length to the correct answer; avoid "all of the above".
- The `explanation` says why the answer is right (and, when useful, why a tempting option is wrong), and is
  consistent with the course text.
- Mix types: mostly `single` and `multiple`, a few `true_false` and at most one or two `ordering`.
- Passing requires 70%, so the quiz must be answerable from the course content alone.

## Before you submit

- [ ] `coursmith validate <folder> --json` returns `ok: true`.
- [ ] Title, description and objectives match what the course actually teaches.
- [ ] All text is in the declared language, at the declared difficulty.
- [ ] Figures are correct or labelled as illustrative; sensitive topics have a disclaimer.
- [ ] Every quiz answer is correct and explained.

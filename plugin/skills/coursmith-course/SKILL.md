---
name: coursmith-course
description: Create an interactive course (steps, charts, diagrams, animations, images, a glossary per section and a final quiz) and publish it to the Coursmith catalog with the coursmith CLI. Use when the user asks to create, write, generate, update, publish or submit a course or lesson for Coursmith, or to check a Coursmith review.
---

# Create and publish a Coursmith course

You write a course as a folder containing `course.json` and an `assets/` directory, check it with the
`coursmith` CLI, and submit it. A review agent on the portal then approves it, suggests improvements (the
user decides whether to apply them), requires changes to parts that break the content policy, or rejects it.
Your job ends when the course is published, or when you have explained the review outcome to the user.

Run the CLI as `npx --yes coursmith@latest <command> --json`. The public npm package includes the CLI and this skill;
no GitHub access or global installation is needed. If `npx` is missing, ask the user to install Node.js 20 or newer
and follow the guide at `/install`. With `--json` every command prints one JSON document; read it instead of guessing.

Reference files in this skill folder (read them when you reach the matching step):

| File | Use it for |
|---|---|
| `references/course.schema.json` | The exact structure of `course.json`. Unknown keys are rejected. |
| `references/block-catalog.md` | Every content block and visual (simulators, 3D scenes, hotspots, flashcards too), with valid JSON examples and rules. |
| `references/animation-presets.md` | The 8 animation presets (including keyframed `svg-motion`) and when to use each. |
| `references/writing-guide.md` | How to write steps, glossaries and quizzes that pass review. |
| `references/content-policy.md` | What is not allowed and how to treat sensitive topics. |
| `references/source-material.md` | Building a course from the user's own files (PDF, Word, Excel, text). |
| `examples/sample-course/` | A complete course that passes validation. Copy its shape. |

## 1. Log in

1. Run `npx --yes coursmith@latest whoami --json`. If it prints a `username`, continue to step 2.
2. Otherwise run `npx --yes coursmith@latest login --no-wait --json`. Show the user the `verificationUri` and the
   `userCode` and ask them to open the link, check the code and click **Approve**.
3. When the user confirms, run `npx --yes coursmith@latest login --resume --json` and check that it prints `logged_in`.

## 2. Agree on the course with the user

Ask, in one message, for anything the user has not already given:

- **Topic** and, if useful, the angle or audience.
- **Language** of the course (any language; you will need its BCP-47 code such as `it`, `en`, `pt-BR`).
- **Difficulty**: `beginner`, `intermediate` or `advanced`.
- **Visibility**: `public` (listed in the catalog, the default) or `private` (hidden from the catalog and shared
  only through a link the user controls from the dashboard).
- Optional: desired length (default about 30 minutes), prerequisites, things to include or avoid.
- **Their own material**, if they have any: notes, slides, a manual, a spreadsheet (PDF, DOCX, XLSX, CSV, TXT,
  MD). If they do, also ask how to use it, in plain words:
  - **Keep my content**: their text stays as written; you only organize it into sections, steps and visuals.
  - **Improve it**: their material is the starting point; you may rewrite, correct, complete and enrich it.
  Then follow `references/source-material.md`.

Check the topic against `references/content-policy.md`. If it is not allowed, say so and stop. If it is a
sensitive topic (health, law, finance, politics, religion…), plan a neutral treatment and a disclaimer.

## 3. Propose an outline and get it approved

Show the user:

- title, one-line subtitle, 2–6 learning objectives;
- 3–6 sections, each with a one-line summary and 2–5 step titles;
- for each step, the visual you plan (chart, diagram, animation preset, simulator, 3D scene, hotspots, image or
  none);
- the planned number of quiz questions (at least 10, covering every section).

Revise until the user agrees. Do not write `course.json` before that.

## 4. Write the course

Create a new folder named after the course (kebab-case) with `course.json` and `assets/`. Follow
`references/course.schema.json` exactly and copy the structure of `examples/sample-course/course.json`.

Rules the validator enforces (details in `references/block-catalog.md`):

- `schemaVersion` is `"1.0"`. Do not write `courseId`: the CLI manages it.
- All ids are lowercase letters, digits and dashes, unique across the course (sections, steps, assets,
  quiz questions).
- Every step has a `title` and at least one block; most steps also have a `visual`.
- **Glossary**: every section has a `glossary` that defines every concept it introduces and every technical or
  domain term it uses. In the text, mark each term as `[[term]]`, or `[[term|text shown]]` when the
  wording differs (use `aliases` for plurals and inflections). A marked term must be defined in the same
  section or an earlier one. Definitions are self-contained and written in the course language.
- **Visuals**: charts are Vega-Lite specs with inline `data.values` (no URLs); diagrams are Mermaid without
  `click` or `%%{init}%%`; animations use one of the 8 presets; images refer to an asset by id.
- **Interactive elements**: `simulator` (sliders and live formulas), `scene3d` (rotatable 3D primitives),
  `hotspots` (tappable points on an image) and `flashcards`. Use them where manipulating something teaches
  better than reading about it; formulas use only the simulator's own controls.
- **Markdown**: no raw HTML, no Markdown images (use an image block), links must be `https://`.
- **Final quiz**: at least 10 questions; every section is tested by at least one question (`sectionRef`);
  every question has an `explanation`. Types: `single`, `multiple`, `true_false`, `ordering`
  (ordering items are listed in the correct order; the portal shuffles them).
- Optional `check` questions at the end of a step give learners quick feedback and are not scored.

Write section by section and keep the language, difficulty and terminology consistent throughout
(see `references/writing-guide.md`).

## 5. Create the assets

- A **cover image** is required (`meta.coverImage`), ideally 1200×630.
- If you have an image-generation tool, produce PNG, JPEG or WebP files. Otherwise write clean SVG by hand:
  simple shapes, a clear palette, `viewBox` set, no scripts, no external links or fonts, no embedded images.
- For a `highlight-svg` animation, give the elements to highlight an `id` attribute.
- Put every file directly in `assets/` and declare it in `assets[]` with `kind` (`image` or `svg`), `file`
  (`assets/<name>`) and a meaningful `alt` text. Fill `generatedBy` for AI-generated images.
- Each file must be under 3 MB; the whole bundle under 25 MB.

## 6. Validate until clean

Run `npx --yes coursmith@latest validate <folder> --json`. It prints `{ "ok": ..., "issues": [...] }`; each issue has
`severity`, `code`, `path` (for example `sections[1].steps[0].blocks[2]`) and `message`.

Fix every `error`, then run it again. Repeat until `ok` is `true`. Fix warnings too when they are cheap
(for example `glossary.unreferenced`: mark the term with `[[...]]` where it is used).

## 7. Submit and follow the review

1. Run `npx --yes coursmith@latest submit <folder> --json`, adding `--private` if the user wants a private course
   (`--public` makes a private course public again). It uploads the course and prints `status: "in_review"`
   with a `submissionId`. The CLI writes `.coursmith.json` in the folder; keep it, because it links later
   submissions to the same course.
2. Run `npx --yes coursmith@latest status --dir <folder> --wait --json` (it waits up to 10 minutes; run it again if
   the status is still `in_review`).
3. Act on the `status`. Each issue in `review.issues` has a `level`: `required` (breaks the content policy),
   `recommended` (an improvement the reviewer suggests) or `optional` (small polish).
   - `approved`: give the user the `courseUrl` (and the `shareUrl` for a private course). Mention any optional
     suggestions.
   - `changes_suggested`: the course can be published; the reviewer recommends improvements. **The user decides,
     not you.** Summarize the `recommended` issues in plain words (what and where, why it matters), then ask:
     apply them, apply only some, or publish as it is. To apply them, edit the elements at each `path`, validate
     and submit again. To publish as it is, run `npx --yes coursmith@latest publish --dir <folder> --json` and give
     the user the `courseUrl`. The user can also publish it from the dashboard (`manageUrl`).
   - `needs_revision`: some parts break the content policy (offensive, hateful, sexual, inciting… content). They
     must be fixed before the course can be published. For each `required` issue, rewrite or remove the element at
     `path` following the `suggestion`, tell the user what you changed, then validate and submit again.
   - `rejected`: the course as a whole conflicts with the content policy. Explain `review.summary` to the user. Do
     not resubmit the same content.

If `submit` reports `review_in_progress`, wait for the current review with `status --wait` first.

## Private courses

A private course is reviewed like any other but never appears in the catalog. When it is published, the status
includes a `shareUrl`: whoever opens it logs in and joins the course. The user manages the link and the list of
people at `manageUrl` (dashboard), where they can also switch the course between private and public.

## Updating a published course

Edit the same folder (keep `.coursmith.json`) and submit again. The new version goes through review while
the current one stays online; keep step ids stable so learners keep their progress.

## Troubleshooting

| CLI output | What to do |
|---|---|
| `not_logged_in` / exit code 3 | Go back to step 1. |
| `client_outdated` | Retry with `npx --yes coursmith@latest` so npm fetches the latest CLI. |
| `validation_failed` from `submit` | Run `validate` and fix the issues it lists. |
| `rate_limited` | The user reached the daily submission limit; try again tomorrow. |
| `assets.content_type` / `assets.invalid` | The file content does not match its extension, or it is corrupt. Re-export it. |
| `animation.unknown_element` | The SVG lacks an element with that `id` (ids are kept, scripts and external links are removed). |

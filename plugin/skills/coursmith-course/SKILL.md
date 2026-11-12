---
name: coursmith-course
description: Create an interactive course (steps, charts, diagrams, animations, simulations, 3D models, exercises, a glossary per section and a final quiz) and publish it to the Coursmith catalog with the coursmith CLI. Use when the user asks to create, write, generate, update, publish or submit a course or lesson for Coursmith, or to check a Coursmith review.
---

# Create and publish a Coursmith course

You write a course as a folder with `course.json` and an `assets/` directory, check it with the `coursmith` CLI and
submit it. A review agent on the portal then approves it, suggests improvements (the user decides whether to apply
them), requires changes to parts that break the content policy, or rejects it. Your job ends when the course is
published, or when you have explained the review outcome to the user.

Run the CLI as `npx --yes coursmith@latest <command> --json`: no installation is needed. If `npx` is missing, ask the
user to install Node.js 20 or newer (https://coursmith.onrender.com/install). With `--json` every command prints one
JSON document: read it instead of guessing.

## The guides

The detailed rules are not in this folder: the CLI serves them to logged-in users. Read each one when you reach the
step that needs it (log in first, step 1):

```bash
npx --yes coursmith@latest docs                    # list
npx --yes coursmith@latest docs <name>             # print one
npx --yes coursmith@latest docs <name> --out <file>  # save one (useful for the long ones)
```

| Guide | Read it |
|---|---|
| `content-policy` | at step 2, to check the topic |
| `source-material` | at step 2, if the user gives you their own files |
| `writing-guide` | before the outline (step 3): structure, difficulty, glossary, visuals, practice, quiz |
| `block-catalog` | before writing (step 4): every block, visual and question type, with valid examples |
| `animation-presets` | when you plan an animation |
| `course-schema` | the exact JSON Schema, when the validator reports a structure error you do not understand |
| `example` | before writing: a complete `course.json` that passes validation; copy its shape |
| `cover-example` | before drawing the cover: a cover in the house style |
| `blueprint` | when the user's request contains a blueprint (see below) |

Follow the guides exactly: the validator and the reviewer apply the same rules.

## Blueprints

The user may describe the course with a **blueprint**: a plan in the prompt with settings (`level: …`), one
`# Section` heading per section and the elements it must contain (`- photo: …`, `- quiz`), often in a fenced block
tagged `blueprint`. Then read the `blueprint` guide after logging in and follow it: it changes steps 2, 3 and 6.
Without a blueprint, everything below applies as written.

## 1. Log in

1. Run `npx --yes coursmith@latest whoami --json`. If it prints a `username`, continue.
2. Otherwise run `npx --yes coursmith@latest login --no-wait --json`. Show the user the `verificationUri` and the
   `userCode` and ask them to open the link, check the code and click **Approve**.
3. When the user confirms, run `npx --yes coursmith@latest login --resume --json` and check that it prints `logged_in`.

## 2. Agree on the course with the user

Ask, in one message, for anything the user has not already given:

- **Topic** and, if useful, the angle or audience.
- **Language** of the course (any language; you will need its BCP-47 code such as `it`, `en`, `pt-BR`).
- **Difficulty**: `beginner` (middle school, ages 11–14), `intermediate` (high school, 14–19) or `advanced`
  (university).
- **Visibility**: `public` (listed in the catalog, the default) or `private` (shared only through a link the user
  controls from the dashboard).
- Optional: desired length (default about 30 minutes), prerequisites, things to include or avoid.
- **Their own material**, if any (PDF, DOCX, XLSX, CSV, TXT, MD), and whether to **keep their content** as written or
  **improve it**. Then follow the `source-material` guide.

Check the topic against the `content-policy` guide. If it is not allowed, say so and stop. If it is sensitive
(health, law, finance, politics, religion…), plan a neutral treatment and a disclaimer.

## 3. Propose an outline and get it approved

Read the `writing-guide`, then show the user: title, subtitle and 2–6 learning objectives; 3–6 sections with a
one-line summary and 2–5 step titles each; the visual or interactive element planned for each step, chosen with
the "Balance" rules of the guide (no more than two text-only steps in a row, one main element per step, the photo
budget for the subject) and the photos you plan (and, in
literature and philosophy, the passages you will quote); the practice (worked examples, exercises, scenarios,
checks); for mathematics and physics, the final practice set from easy to hard; the bibliography you plan; the
number of final quiz questions (at least 10, covering every section). Revise until the user agrees. Do not write
`course.json` before that.

## 4. Write the course

Read the `example` and the `block-catalog`. Create a new folder named after the course (kebab-case) with
`course.json` and `assets/`, and write it section by section, keeping language, difficulty and terminology consistent.
Then add the `practice` set (mathematics and physics) and the `bibliography` (every course), before `finalQuiz`.

Be exact: check every date (births, deaths, events, works), explain every symbol of every formula, quote the texts
you discuss in literature and philosophy, and cite only books, articles and sites you are sure exist.

## 5. Create the assets

- A **cover image** is required: a hand-written SVG at 1200×630 in the house style (writing guide, "Cover":
  paper background, ink lines, one accent, no text), never a third-party photo.
- **Real photos** (the object, place, organism or artwork the text talks about) come with their credit:
  1. `npx --yes coursmith@latest image search "<what it shows>" --json` (English keywords work best);
  2. download the `previewUrl` of the promising results and look at them: never add a photo you have not seen;
  3. `npx --yes coursmith@latest image add "<ref>" --dir <folder> --id <asset-id> --alt "<what it shows and why>" --json`.
- **Sound clips** (`audio` blocks) work the same way: `audio search`, listen, `audio add`.
- Diagrams: clean hand-written SVG (with `viewBox`, no scripts, no external links or fonts) or generated images.
- Every file directly in `assets/`, declared in `assets[]` with a meaningful `alt`; each under 3 MB, all under 25 MB.

## 6. Validate until clean

Run `npx --yes coursmith@latest validate <folder> --json` and fix every `error` it lists (each has a `path` and a
`message`), then run it again until `ok` is `true`. Fix warnings too when they are cheap, and always the
`balance.*` ones (they mean the course reads as a wall of text, or has too many or too few photos).

Then check the facts: `npx --yes coursmith@latest cite check <folder>` (the bibliography) and, when the course has
`person` or `lifelines` blocks, `npx --yes coursmith@latest facts check <folder>` (the dates). Fix what they report.

## 7. Submit and follow the review

1. `npx --yes coursmith@latest submit <folder> --json` (add `--private` for a private course). Keep the
   `.coursmith.json` file it writes in the folder: it links later submissions to the same course.
2. `npx --yes coursmith@latest status --dir <folder> --wait --json`; run it again while the status is `in_review`.
3. Act on the `status`. Each issue in `review.issues` has a `level`: `required`, `recommended` or `optional`.
   - `approved`: give the user the `courseUrl` (and the `shareUrl` for a private course).
   - `changes_suggested`: the course can be published. **The user decides, not you**: summarize the `recommended`
     issues in plain words and ask whether to apply them, apply some, or publish as it is
     (`npx --yes coursmith@latest publish --dir <folder> --json`).
   - `needs_revision`: fix every `required` issue following its `suggestion`, tell the user what you changed,
     validate and submit again.
   - `rejected`: explain `review.summary` to the user and do not resubmit the same content.
   - Still `in_review` with a note that it is *held for an administrator*: the text passed review and an
     administrator is checking the images; tell the user, nothing else is needed.

To update a published course, edit the same folder (keep `.coursmith.json` and the step ids) and submit again.

## Troubleshooting

| CLI output | What to do |
|---|---|
| `not_logged_in` / exit code 3 | Go back to step 1. |
| `client_outdated` | Retry with `npx --yes coursmith@latest`. |
| `unknown_doc` | Run `docs` without a name to see the guides. |
| `validation_failed` from `submit` | Run `validate` and fix what it lists. |
| `rate_limited` | Daily limit reached; try again tomorrow. |
| `review_in_progress` | Wait for the current review with `status --wait`. |

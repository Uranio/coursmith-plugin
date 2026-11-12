---
description: Create a Coursmith course and publish it to the catalog
argument-hint: "[topic, language, difficulty, files of your own material, or a blueprint]"
---

Use the `coursmith-course` skill to create and publish a Coursmith course.

What the user asked for: $ARGUMENTS

Follow the skill's workflow from the start: check the login, ask for whatever is missing among topic,
language, difficulty and visibility (public or private), and get the outline approved before writing
`course.json`. If the user mentions or attaches their own files (PDF, Word, Excel, text), ask whether to keep
their content as written or to improve it, and follow the `source-material` guide (`npx --yes coursmith@latest docs source-material`).

If the request contains a Coursmith Blueprint (settings such as `level:`, `# Section` headings and `- element`
items, often in a ```blueprint block), follow the `blueprint` guide (`npx --yes coursmith@latest docs blueprint`):
the plan decides the sections and what each contains.

---
description: Show the Coursmith review status of a course and act on the feedback
argument-hint: "[course folder or submission id]"
---

Check the Coursmith review of: $ARGUMENTS

- If that is a folder (or nothing was given and the current folder contains `.coursmith.json`), run
  `npx --yes coursmith@latest status --dir <folder> --json`. If it is a submission id, run
  `npx --yes coursmith@latest status <id> --json`.
- Summarize the result for the user: status, review summary, and each issue with its path and suggestion.
- If the status is `changes_suggested`, the course is ready: explain the recommended improvements and ask the
  user whether to apply them or publish as it is (`npx --yes coursmith@latest publish --dir <folder>`). Never
  decide for them.
- If the status is `needs_revision`, some parts break the content policy and must change: offer to fix them
  following the `coursmith-course` skill (section 7), then validate and resubmit.

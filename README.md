# Coursmith skill and Claude Code plugin

Create interactive courses with an AI agent and publish them to the [Coursmith](https://coursmith.onrender.com)
catalog, where a review agent approves them, requests revisions or rejects them.

This repository contains the `coursmith-course` Agent Skill and the optional Claude Code plugin. It is published
automatically from the Coursmith source: changes made here are overwritten.

## Install (recommended)

Requirements: Node.js 20 or newer, an agent that can use Agent Skills, and a Coursmith account.

```sh
npx --yes coursmith@latest setup
```

This installs the skill for Claude Code, Codex, Cursor, OpenCode and pi, then opens Coursmith in your browser to
connect your account. Approve the device login and restart your agent. To install for one agent, add `--agent codex`
(also `claude`, `cursor`, `opencode` or `pi`). Add `--project` to install in the current repository.

## Other ways to install

With the [skills CLI](https://skills.sh):

```sh
npx skills add Uranio/coursmith-plugin
```

As a Claude Code plugin, which also adds the `/coursmith:create` and `/coursmith:status` commands:

```text
/plugin marketplace add Uranio/coursmith-plugin
/plugin install coursmith@coursmith
```

Both install the skill only: connect your account once with `npx --yes coursmith@latest login`.

## Use and update

Ask your agent: *"Create a Coursmith course about photosynthesis, in Italian, for beginners."* It proposes an outline,
writes and validates the course, submits it and follows the review.

For manual CLI use, prefix commands with `npx --yes coursmith@latest`, for example:

```sh
npx --yes coursmith@latest whoami
npx --yes coursmith@latest validate ./my-course
npx --yes coursmith@latest status --dir ./my-course --wait
```

Refresh the CLI and skill with `npx --yes coursmith@latest setup`, then restart your agent. Full installation guide:
<https://coursmith.onrender.com/install>.

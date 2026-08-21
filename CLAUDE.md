# CLAUDE.md

This is the **business analyst** repo for the Stock/HPP business-finance
project — requirements, specs, and source material, not application code.

## Contents

- `Idea/Catatan_HPP_Keuangan_Bisnis.xlsx` — Indonesian-language notes on
  business finance / COGS (Harga Pokok Penjualan). Source material for the
  requirements that drive the sibling implementation repos.

## Relationship to sibling repos

This project is split across independent repos, each buildable and
deployable on its own with no shared code or path dependency between them:

- `stock-frontend` — client-side UI
- `stock-backend` — API / business logic / data layer
- `stock-infrastructure` — CI/CD, deployment, IaC
- `stock-qa` — test plans, test automation
- `stock-business-analyst` (this repo) — requirements, specs, source material

Specs/user stories written here are the input to the other repos; this repo
does not consume their code.

## Working here

`.claude/` config (agents, hooks, skills, MCP) is kept identical across all
five repos on purpose, so any agent persona works the same way regardless of
which repo it's invoked in.

## Skill Activation

At the start of any task-oriented session — any interaction where you will
use tools and produce deliverables — invoke the task-observer skill before
beginning work. This ensures skill improvement opportunities are captured
throughout the session.

When loading any skill, check the observation log for OPEN observations
tagged to that skill. Apply their insights to the current work, even if
the skill file hasn't been updated yet. This enables immediate application
of observations before they're permanently integrated during the weekly
review.

## Git

Commit, push, and open a PR automatically as part of completing a task —
this is standing authorization across all five `stock-*` repos, no need to
ask first each time. Use atomic-commit (or equivalent judgment) to split
work into logical commits, then push the branch and open the PR without
waiting for a separate go-ahead.

Never push directly to the `main` branch, even when explicitly asked to
"push" or "commit and push" — `main` is protected and requires a pull
request. Always push to a new branch and open a PR instead, across all
five `stock-*` repos.

Always branch off `main` for new work, and sync first: run
`git fetch origin && git merge --ff-only origin/main` (or
`git pull --ff-only`) before creating the branch — cutting a branch from a
stale local `main` produces a PR with a stale diff or spurious merge
conflicts.

## GitHub operations

Prefer the GitHub MCP server (configured in `.mcp.json`) over raw
`git push` / `gh` / GitHub REST calls for anything that talks to GitHub
itself — opening PRs, commenting, managing issues, etc. It authenticates
directly and needs fewer manual permission approvals than shell-level
push/curl commands. Local-only git operations (branching, committing)
stay as plain git commands; only the GitHub-facing calls should route
through the MCP tools when the server is connected in the session. If
the MCP server isn't available, fall back to `git push` / `gh` and say
so, rather than silently skipping the step.

## Code style

Always put comments in code so it is understandable by a human reader —
applies to any scripts kept in this repo (this repo is mostly specs/source
material, not application code).

## Environment files

Always use `.env.local` for local config — never create or reintroduce a
`.env.example`/`.env.sample` template file. `.env.local` already exists in
this repo (gitignored) and holds the real placeholder values directly; if a
new env var is needed, add it straight to `.env.local` (with a comment
explaining it) rather than adding a separate example file for someone to
copy from.

MCP servers configured in `.mcp.json` that reference `${VAR_NAME}` (e.g.
the `github` server needs `GITHUB_PERSONAL_ACCESS_TOKEN`) read from the
process environment, which is not populated automatically. Before using
such an MCP server or making authenticated GitHub calls, read the value
out of this repo's `.env.local` (e.g.
`grep GITHUB_PERSONAL_ACCESS_TOKEN .env.local`) and export it for the
current shell — don't assume it's already set.

## Summarizing work

Always route end-of-task summaries into the appropriate existing file in
this repo instead of only stating them in chat — future sessions read
these files for context, chat history doesn't persist:

- **`findings.md`** — results of a review/audit/investigation (e.g.
  security review, code review, research pass). Append as a new dated
  section; don't overwrite prior entries.
- **`instructions.md`** — step-by-step how-to material (e.g. how to run,
  test, or deploy something). Update the relevant section, or add a new
  one if it's a new procedure.
- **`questions.md`** — open questions, scope decisions made without
  asking, or anything that needs the user's confirmation later. Append as
  a new dated section.

If a summary doesn't fit any of the three, ask which file it belongs in
(or whether it needs a new file) rather than skipping the write-up.

## Gitignore

Always ensure a `.gitignore` exists in this repo — never let it be
deleted or skipped when scaffolding. It has two parts:

- A **shared baseline** kept identical (word-for-word) across all five
  `stock-*` repos: `.env`, `.env.local`, `.claude/settings.local.json`. If
  you add an entry to this shared baseline in any one repo, add the same
  line to the other four repos' `.gitignore` files too, so they stay in
  sync.
- **Repo-specific entries** below the baseline — this repo is mostly
  specs/source material, so it currently has none beyond the shared
  baseline; add repo-specific entries here only if this repo starts
  holding generated/tooling output that shouldn't be tracked.

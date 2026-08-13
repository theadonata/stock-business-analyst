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

## Git

Do not commit changes in this repo automatically — even when using
atomic-commit or similar workflows. Only commit when the user explicitly
asks for it.

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

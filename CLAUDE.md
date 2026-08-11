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

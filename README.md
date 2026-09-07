# stock-business-analyst

> The original business requirements behind the Stock/HPP project.

## About the project

**Stock/HPP** is a small web app that replaces an Excel spreadsheet
(`Catatan_HPP_Keuangan_Bisnis.xlsx`) a small bags & accessories business
used to track sales, stock, expenses, and cost of goods sold ("HPP" is
Indonesian for *Harga Pokok Penjualan*). This repo holds the source
material and decisions that everything else in the project is built from
— it has no application code of its own.

### Part of a bigger project

Stock/HPP is split into six repos, each one buildable and deployable on
its own:

| Repo | What it does |
|---|---|
| [stock-frontend](https://github.com/theadonata/stock-frontend) | The web app people use day to day |
| [stock-backend](https://github.com/theadonata/stock-backend) | The API and database — stores data, does the math |
| [stock-infrastructure](https://github.com/theadonata/stock-infrastructure) | Deploys and runs everything on a server |
| [stock-qa](https://github.com/theadonata/stock-qa) | Automated tests that check everything works |
| **stock-business-analyst** (this repo) | The original business requirements this is built from |
| [stock-platform](https://github.com/theadonata/stock-platform) | An internal dashboard for the team building this project |

## What's here

- **`docs/superpowers/specs/2026-08-12-stack-architecture-design.md`** —
  the full architecture and design spec every other repo implements
  against
- **`instructions.md`** — a step-by-step guide to running the whole app
  (backend + frontend) locally and testing it as a human would
- **`questions.md`** — open questions and scope decisions made while
  building against the spec
- **`findings.md`** — a security review (git history, secrets, auth,
  injection surfaces, container hardening) done before this project's
  first public push

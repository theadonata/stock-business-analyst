# stock-business-analyst

Requirements, specs, and source material for the Stock/HPP business-finance project.

Part of the `stock-*` multi-repo project. See CLAUDE.md for scope and
sibling-repo relationships.

The stack/architecture design lives at
`docs/superpowers/specs/2026-08-12-stack-architecture-design.md`.

- **`instructions.md`** — step-by-step guide to running the whole app
  (backend + frontend) locally via Docker and testing it as a human.
- **`questions.md`** — open questions and scope decisions from implementing
  against the design spec. Review when convenient; the last remaining
  unanswered item is whether/when to wire up CI.
- **`findings.md`** — security review done ahead of the first GitHub push
  (git history, secrets, auth, injection surfaces, container hardening).
  Nothing found blocks pushing; a few deployment-hardening items are listed
  for before any real (non-local) deployment.

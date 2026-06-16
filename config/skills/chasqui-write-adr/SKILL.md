---
name: chasqui-write-adr
description: "INVOKE when making a non-obvious architectural or design decision in a Chasqui stack (database, dimensions, protocols, a new archetype, a contract change) — Chasqui's convention is to record it as an ADR in the SAME PR as the code. Also covers the docs-as-code / end-of-sprint docs rule. Use this so the decision is captured the Chasqui way instead of living only in a PR comment or a wiki."
---

# Write an ADR (the Chasqui decision habit)

Chasqui is **docs-as-code, no wiki** (wikis drift; `docs/` is versioned and
reviewed with the code). The rule: **any non-obvious architectural decision gets
an ADR in the same PR/commit** as the change it justifies. A sprint isn't closed
until the docs reflect it.

> Source of truth — the convention lives in the repo's AGENTS.md, and the ADRs in
> `docs/design/`:
> - Conventions (planning, ADRs, end-of-sprint rule):
>   https://raw.githubusercontent.com/chasqui-stack/chasqui/v0.2.5/AGENTS.md
> - Existing ADRs (read a recent one as your template):
>   https://github.com/chasqui-stack/chasqui/tree/v0.2.5/docs/design

## When an ADR is warranted

A decision is "non-obvious" when a competent reader would reasonably ask *"why
this and not the alternative?"* — DB/storage choices, embedding dimensions,
protocol/contract changes, a new background worker, a new archetype. Routine
bugfixes, renames, and dependency bumps do **not** need one.

## Where and how

- File: `docs/design/adr-NNN-<short-kebab-title>.md` — `NNN` is the next number in
  `docs/design/` (zero-padded). Check the directory for the highest existing one.
- Open a recent ADR (e.g. `adr-008-...`) and **match its shape**:
  - A header block: **Status** (Proposed / Accepted — date), **Sprint**, **Related**
    (link sibling ADRs + the relevant ARCHITECTURE section).
  - **Context** — the problem and the forces, concretely (cite the bug/PR that
    surfaced it if there is one).
  - **Decision** — what we're doing, in enough detail to implement against.
  - **Consequences** — positive *and* negative/trade-offs (be honest).
  - **Alternatives considered** — what was rejected and *why*.
  - **Follow-ups (out of scope)** — deferred work.
- Commit it **in the same PR** as the code it explains. Reference the issue
  (`Refs #N`). If the decision changes the canonical contract, also update the
  relevant `docs/ARCHITECTURE.md` section in the same PR.

## Anti-patterns

- ❌ Shipping an architectural change without an ADR — the decision evaporates.
- ❌ Writing the ADR in a separate, later PR — it must travel with the code.
- ❌ An ADR with no "Alternatives" / no "negative consequences" — that's a brochure,
  not a decision record.
- ❌ Putting design docs in a GitHub wiki — Chasqui keeps them in versioned `docs/`.

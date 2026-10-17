---
round: 1
focus:
  - "Cards 4-5: check that every pull-request lookup of Publish, PR-Gate, PR-Align, Finalize and the `lyx loom approve` and `lyx loom reject` verbs receives the run-start bound from the seed's `started_at`, and that no caller of `FindPullRequest` is left calling it unbounded."
  - "Card 5: settle how a missing or unparseable `started_at` in the seed is handled, and whether the bound comparison is correct at the boundary and across time zones."
  - "Card 6: check that Publish stops on an older open pull request it cannot replace, that the stop message names a way forward, and that a pull request the run itself opened is not stopped."
  - "Cards 1-3: check that the decision-note title form `Decision for <slug>:` and the escalation reason are each declared once in `internal/alignment`, that the snapshot counts only notes in that form, and that the orch role stencil renders the form as a required marker without dropping an existing rule."
  - "Test economy: check each new test in the diff against existing coverage, and that each added behavior (bounded lookup, older-open stop, title-form counting) has a test."
  - "Docs: check that `doc.go` of the touched packages, and `PATTERN.md` if the diff adds a cross-cutting invariant, are updated in the same commits."
---

Seed focus only; the reviewer still reads the full range and judges the diff as code.

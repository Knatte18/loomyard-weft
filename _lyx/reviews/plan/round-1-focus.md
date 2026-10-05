---
round: 1
focus:
  - "Fidelity, card 2: the card edits `internal/loomcli/cli.go` and `pattern/PATTERN-shed-verb-set.md`. Settle whether the decision record's Scope, D2 or D4 licenses each edit, or whether either is scope the answer key does not grant."
  - "Fidelity, card 1: the card deletes `internal/shuttlecli/smoke_bashstdin_test.go`. Settle whether the record's scope (tests covering the changed behavior) licenses the deletion, and whether the plan's `## verify:` still compiles or runs what remains in `internal/shuttlecli`."
  - "Fidelity, card 3: D3 says `## Error envelopes`, `## Parking` and `## Repair records and the stop report` each say where `friction_dir`, `scratch_dir` and `next_interrupt_policy` are read from. The card says it once and points back from later sections. Settle whether that carries D3 and the acceptance criterion on where to read them."
  - "Fidelity: walk every D1–D4 Decision, Bound and Constraint, and every Acceptance criterion, against the cards. Settle which has no carrying card, in particular the `--full` record-identical criterion, the `last_step.envelope_path` equality and the pwsh `WithEnv` form."
  - "Test economy: card 1's `envfile_integration_test.go` and card 2's `steprecord_test.go` table plan new tests. Settle whether an existing test already covers each row, naming that test, or whether any row is a per-symbol test covered by a public-surface test."
  - "Verify coverage: every package a card targets, including `internal/shuttleengine/claudeengine`, `internal/shedverbs`, `internal/loomcli`, `internal/shedcli` and `cmd/lyx`, against the `go test`, integration-tag and `go vet -tags smoke` lines in `## verify:`. Settle which targeted package's tests no line runs."
  - "Attack surface: `--full`, the removed hook, the `CLAUDE_ENV_FILE` override and the prose-only no-`cd` rule. Settle whether each card's `**Intent:**` states what the change can skip or let through and what bounds it."
  - "Granularity and ImpactSummary: settle whether card 1 and card 2 each bundle one reviewable unit, and whether each `**ImpactSummary:**` states a blast-radius conclusion rather than restating `**Intent:**`."
---

Seed focus for round 1 over `_lyx/plan` (overview and three cards), measured against `_lyx/discussion/decision-record.md`.
No `Custom` or `Prosa` card exists, so the `Custom` and glyph questions are not seeded.

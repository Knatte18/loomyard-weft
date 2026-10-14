---
format: 5
approved: false
language: go
---

# Plan: claimed-scope-edit

A mid-run scope correction on a darn run reaches every later row of that run, and a darn pull request no longer waits for the orch at PR-Gate when the alignment agent can settle it.
The darn writer writes loom's decision record and support log before any code, in a format both writers now read from a deployed spec; Go commits the record at the writer's first clean validation and its gate refuses a record that dropped an entry its commit history held, so `lyx loom decision add` can append to it while Darn-Write runs.
Darn-Review and Describe measure against the board entry as amended by the record's `Added after Discussion` entries, Darn-Bouncer carries its open findings into the record, and a PR-Align row lets the alignment agent settle darn's PR-Gate under darn's own measure.

## Card Index

1 — discussion-spec — move the record format into `loom-discussion-spec.md` and point the Discussion stencil at it through a specs marker
2 — committed-file-history — fabricengine reads every committed version of an anchored file
3 — kept-entries-check — discussionparser reports `Added after Discussion` entries a record dropped from its history
4 — darn-discussion-gate — loomshed's darn discussion check and gate: sections, history, then a commit
5 — gate-resolver-darn — the `discussion` gate entry resolves to the darn gate when told the record's history
6 — validate-discussion-darn — a darn run's `validate-discussion` runs the darn check and commits; wire the history seam
7 — decision-add-darn — `decision add` on a darn run once the record is committed, with the new way forwards
8 — darn-writer-record — the darn writer writes the record before any code and follows recorded corrections
9 — darn-describe-record — darn's Describe reads the record
10 — darn-review-scope — Darn-Review measures against the amended board entry and defers an unrecorded correction
11 — alignment-pr-gate-measure — the alignment agent's pr-gate measure is a told fragment and an absent plan reads `none`
12 — darn-pr-align-wiring — PR-Align is wired with darn's geometry on a darn run
13 — darn-recipe-rows — discussion gate, carry-over and PR-Align in the darn recipe
14 — orch-correction-guidance — the orch records corrections through `decision add` and expects darn's PR-Gate as an escalation

## Shared Decisions

- The darn record is loom's record at loom's paths (D1): every new seam reads `loomengine`'s accessors in `loomcli` wiring, and `darnengine`, `alignment` and `landingshed` stay told their paths.
- The darn-only behavior hangs off one Env seam, `shedrecipe.Env.DiscussionHistory`, which only a darn build sets: the `discussion` gate name stays the one Discussion-Write uses (D4), and a loom build resolves and validates exactly as before.
- The record is read and written only through `internal/discussionparser` (`PATTERN-sole-parsers`), and its history only through `internal/fabricengine` (`PATTERN-fabric-git`).
- Recipe-specific prompt text lives in registered stencils read at call time (`PATTERN-stencil-ownership`), never in Go string literals.
- An in-flight darn run is restarted, not migrated (A4); no row is renamed.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...

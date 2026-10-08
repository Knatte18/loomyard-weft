# Webster-Review round 1 — fixer report

Every finding was checked against the code and the plan and found valid; all three are fixed, one commit each.
`go build ./...`, `go vet` under the `integration`, `tmux` and `llm` tags, and `go test ./...` pass after the last fix; `go test -tags integration` passes over the three touched packages, and `lyx loom lint-comments` passes.

## Fixed

- **F1** (LOW, scope) — `71dd1776e`.
  `TestBouncer_ConvergedSettle_CarryOverFailureBlocksTheSettle` in `internal/shedadapters/bouncer_commit_test.go` is now table-driven, with a new row for a run directory outside `AnchorPath`.
  That row asserts the error names the cause ("does not lie under the anchor") and the way forward, and that neither the seam, Approve nor Commit runs.
  The `carry-over entry refused` row in `contracts/specs/refusal-spec.md` now lists a review or fixer-report path outside the worktree anchor among its triggers.
  No row was added for the unreadable-ledger cause, because no test can reach it through `Call`: the judge harvest (`internal/shedadapters/bouncerfiles.go:155`) already parses the round's ledger and degrades before `settle` runs, so `buildCarryOver`'s ledger read error is only a guard behind it.
- **F2** (NIT, consistency) — `64440d952`.
  The `running` row of the `resume` `Long` table in `internal/loomcli/resume.go` has its eleventh space back, so its description aligns at the same column as the other rows.
- **F3** (NIT, consistency) — `33c5a9e8e`.
  `fencedLines` in `internal/discussionparser/append.go` is now the one fence scan.
  `decisionsInsertOffset`, `locateOpenRisks` and `h2Headings` all call it, so the section locator and the splice self-check cannot disagree about fences.
  Behavior is unchanged, and the package's existing append and carry-over tests pass.

## Disputed

None.

## Deferred

None.

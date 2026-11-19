# Route code review and drift judge rulings to the run's parent

A run's parent orch can now see and veto every Code-Review round and every drift judge ruling by configuration.
Before, only `pr_review_ask_parent` and `discussion_review_ask_parent` existed, so a darn run, which has no Discussion, reached its parent only through escalated decision calls or a hand-written creel instruction.

Closes #153

## What changes

Four new keys, all off by default and all settable in a task's `run_settings`:

- `code_review_ask_parent` and `code_review_parent_timeout_min` in `loom.yaml` (timeout default 30, at least 1).
  Once a round's fixer report exists, and before the round's Bouncer judges it, lym mails the parent a `[code-review request]`.
  The line names the slug, round, ask number, worktree, the round's own review and fixer report, the deadline and the answer verb.
  The parent answers with `lym loom parent-review answer` (satisfied, or a findings file).
- `judge_ask_parent` and `judge_parent_timeout_min` in `drift.yaml` (timeout default 30, at least 1), set per run as `{"drift": {"judge_ask_parent": true}}`.
  Once the judge's `judgement.yaml` parses, lym mails a `[drift-judge request]` with the file path and a summary.
  The summary counts cells and files by ruling and names each non-matching cell, each case and each file with an uncaused loosened check.
  The parent agrees, or names the rulings to change.

`contracts/orchrole/orch-role.md` gains a section for each request, and `docs/overview.md` describes the code review keys.

## Design decisions

- **Code review.**
  The Bouncer's judge waits with `lym loom parent-review await`.
  The wait is a stencil fragment, `bouncer-fragment-parent-review`, that `judgeCall` appends to the judge prompt.
  Every finding in a parent report stands as an open finding of the round, so the next round's reviewer carries it.
  lym refuses a `CONVERGED` verdict while the parent's report holds findings.
  The deadline is capped to end before the judge's own `run_timeout_min`, measured from the judge run's start.
  A cap that leaves no room sends no ask.
- **Drift judge.**
  A new `parent-agreement` gate entry follows the `judgement` entry in Drift-Verify and runs over the new `Deps.JudgeParent` seat (`internal/driftengine/judgeparent.go`).
  The ask records a digest of the parsed rulings it presented.
  A judgement changed after the ask gets a new ask naming the new rulings, and an ask still open is set timeout.
  A satisfied answer to the current rulings, a timeout, or an ask that never reached the parent passes.
  A `does not match` ruling or an uncaused check still stops the run for the parent.
  The entry checks that the judge changed its rulings after the parent's findings, not that it took each one.
- **Key names.**
  The drift keys are `judge_ask_parent` and `judge_parent_timeout_min`, not `drift_judge_ask_parent`, since they live in `drift.yaml`.
- **Guard.**
  All four keys stay open to `run_settings`.
  `pattern/PATTERN-config-layers.md` gets a written exception: a key that only adds an ask to the parent, and the timeout bounding that wait, is not a Guard key.
  Unset or expired, it leaves the run's own gates as they are.
- **Shared seat.**
  The parent-seat wiring in `internal/loomcli` and `internal/shedadapters` is generalised so the discussion, code review and drift asks share one ask record and one answer path.
  An untagged ask reads as the discussion seat's.

## To check by hand

- No regression check ran, and no run with a live parent has exercised the two new mails end to end.
  Run a darn task with `code_review_ask_parent` on and one with `judge_ask_parent` on, answer each request, and confirm the run proceeds.
  Also let one time out and confirm the run continues without the parent.
- The code review ask must name the round's own review and fixer report (`code-ask-misses-rounds-own-review`, `internal/burlerengine/parentreview.go`, `internal/shedadapters/bouncer.go`).
  A review left this open, so confirm the mail names `round-N-review.md` and `round-N-fixer-report.md` of the round that sent it, not an earlier one.
- The judge's parent-review section must come from the stencil fragment alone, with no copy embedded in Go (`judge-parent-block-embedded-in-go`, `internal/shedadapters/bouncer.go`).
  A review left this open, so check that no prompt text for it remains in the Go source.
- The parent wait must not outlive the judge (`judge-parent-wait-outlives-judge`, `internal/shedadapters/bouncer.go` and `internal/driftengine/judgeparent.go`).
  A review left this open, so check that both deadlines are capped by the judge run's own start and `run_timeout_min`.

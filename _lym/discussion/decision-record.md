## Goal

```markdown
## Seen (operator and hub orch, 2026-10-30)

On the first PR-Review run, the reviewer agent ran `lym loom pr-continue` and the run went on only after a while.
The wait sleeps between wakes, from a floor of one second up to a ceiling of one minute while nothing changes (`prWaitFloor`, `prWaitCeiling` in `internal/landingshed/prreview.go`), and `pr-continue` only sets a flag that the next wake reads.
Finalize's gate then runs on the landed tree as for any landing, which takes minutes by itself.

## Design (operator, 2026-10-30)

`lym loom pr-continue` wakes the waiting row at once, so the PR is closed and Finalize begins with no wait of up to a minute.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - watchfilechanges-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

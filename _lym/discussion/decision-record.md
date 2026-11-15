## Goal

```markdown
Imported from [issue #125](https://github.com/Knatte18/loomyard/issues/125).

## Seen

In a Models hub darn run (td-known-bugs), the pr-review gate over head a8c65e38 failed for an environmental reason (an IOException from a concurrent harness run holding a file open). The PR-Review agent had already run `lym loom pr-continue`.

`internal/landingshed/prreview.go`: `gateHead` (740-747) mailed the failure once and wrote the head into `mailed_head`, not `gated_head`. On every later wake `gateDue` (622) is `head != GatedHead && head != MailedHead`, so the same head is never gated again. `judge` (869-872) waits for a gated head because continue is set, and `sessionStopWanted` (673-675) stops and resumes the PR-Review session on every wake, about once a minute, without end. A push of the head to origin changes nothing, since the record reads the local head.

## Workaround

An empty commit in the task worktree moves the head, so `gateDue` turns true and the row re-gates.

## Expected

A failed pr-review gate is retried on the same head when the agent runs `lym loom pr-continue` (or another explicit re-gate verb), and the row does not cycle the session while it waits for a gate that will never run.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - session-stop-cases-unpinned-by-wait-test (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

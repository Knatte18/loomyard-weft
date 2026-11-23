## Goal

```markdown
Imported from [issue #210](https://github.com/Knatte18/loomyard/issues/210).

Seen in hub mo, run hydraulics-gaps, PR-Review row. The PR-Review session's strand was removed (lym reed remove) at 13:44, and the session logged SessionEnd. The row's lym loom step process went on running with status running at PR-Review, and spawned no new session, until the process was killed by hand at 13:50. Only then did the run mark the row interrupted, and a resume start a fresh session. Expected: the row detects a dead PR-Review session and re-spawns it under the interrupt policy, or halts with a stop the parent can act on.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - failed-liveness-read-ends-call-instead-of-warning (class design, severity MEDIUM, open in rounds 1)
  - halt-after-judge-untested (class design, severity MEDIUM, open in rounds 1)
  - no-status-liveness-halt-precedes-status-read (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

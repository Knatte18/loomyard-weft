## Goal

```markdown
Imported from [issue #292](https://github.com/Knatte18/loomyard/issues/292).

## What happened

With local landing, a parent that is ahead of origin by owed landing tips always reads as moved to Finalize's locked pass.
The lock-free pass records the upstream tip (`parentMergeTip`) as the gated tip, while the parent's head is past it.
Every such landing therefore loses each of its `landing.finalize_lost_races` races, then merges and runs verify under the parent lock.
The behavior was left as it is because `gateUnlocked` and `parentMergeTip` were outside the card's targets.

## Why it is abnormal

The landing is correct, but the parent lock is held across the verify for as long as an earlier landing's push is owed.
The lock-free gate gives no benefit in that state, and other landings on the same parent wait.

## What to do

Make the lock-free pass gate against the parent's local head when that head is ahead of origin only by owed landing tips.

Provenance: task `push-retry-not-halt`, note `12-finalize-local-landing.md`.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - finalize-ahead-parent-upstream-fallbacks-untested (class design, severity MEDIUM, open in rounds 1)
  - owed-only-check-not-rechecked-under-lock (class design, severity MEDIUM, open in rounds 1)
  - parent-align-owed-ahead-upstream-tip-unpinned (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

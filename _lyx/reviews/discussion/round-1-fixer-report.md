# Fixer report — discussion round 1

Every finding in `round-1-review.md` is fixed; `lyx loom validate-discussion` exits 0 afterwards.

## Changes

- F1: D2 gains a Bound naming what `validate` stops checking (a begun, non-terminal card's own targets) and the three guards that bound it (first `begin-batch` validation, `CardHashes` pinning, `record-batch`/`recover-batch` done-checks and drift detection).
- F2: D3's Bound now names the foreign-edit check every rewrite call runs first (`PlanEditError`, or run entry's fingerprint compare) as what refuses a foreign edit, including one followed by a sanctioned rewrite, and notes `rebaseline` never adopts a begun-card edit.
  The hash comparison is restated against the `PlanFileHashes` entry for the card's file, with what it guards (a card an earlier untracked rewrite already moved).
- F3: D5 states Go never resets the branch on a failed attempt; the strand's commits stay and summary.md and the escalation's way-forward name the pre-fix head and those commits.
  `support-log.md` gains the reset alternative under Rejected alternatives and Q10 in the Question ledger.
- F4: D1's Bound adds that a begun batch never reaching a terminal record keeps the run from an outcome of done, via the run-exit every-batch-done check.
- F5: D5's triage bullet states that a deleted or skipped test reads as cleared and a weakened one passes, with the prompt prohibition and summary.md as bounds before the review rows; the duplicate Open risks item is removed.
- F6: `recovery_timeout_min`, "no new config key" and the standalone-mode note moved into D5; `## Auto-mode assumptions` is cut to which Decisions were autonomous picks and that D1 departs from the board's direction, with the rejected-alternative narrative left to `support-log.md`.
- F7: D5's escalation records the pre-fix bisect result under `-1` without bisecting again.
- F8: D5's head check is split into the strand's commits (non-merge, from the pre-fix head to `head_sha`) and HEAD's reconciliation against `head_sha` under the existing merge-only rule.

## Deferred

None.

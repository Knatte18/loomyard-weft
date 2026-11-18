---
dispositions:
  - id: F3
    disposition: fixed
---

## Changes

- F3 (commit `65d41e41`, plus the comment correction below): `TestPull_ReplayHeldByAPauseWithoutTheWeftLocks` now counts the waits made while the weft head still equals the local commit, which is before the replay, and pins that count at 3.
  The lock probes and the `WeftPulled` assertion are kept.
  Checked by hand: with the `Admit` and its three `leaveGate` calls deleted from `lockWeftForReplay`, the test fails with `waits before the replay = 2; want 3`; `pull.go` was then restored with `git checkout`, and the test passes.
- The test's doc comment is corrected after the self-check: it said the records fetch was the only gated call before the replay, but two gated calls run before it, so it now says the count is the pull's gated calls before the replay plus the admission.
  That correction is committed with the commit named under Commits below.

## Self-check

- fix-code-correctness, LOW: the doc comment disagreed with the pinned count — fixed.
- fix-consistency, LOW: the same comment disagreement — duplicate, fixed.
- fix-proposal, LOW: the same comment disagreement — duplicate, fixed.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.

# Fixer-report, round 1

## Changes

- F1 (MEDIUM): fixed as proposed, in `internal/landingshed/prreview_wait_test.go`.
  - `ReviewEndedOnAnOpenPullRequestOnADirtyTreeHaltsNamingTheFiles` now seeds a live strand (`withLiveStrand`, `live["guid-1"]`), which only the `rec.ReviewEnded` case of `sessionStopWanted` can stop on its dirty tree.
  - `ContinueOnAClosedPullRequestAtTheGatedHeadIsDoneAndRecordsIt` has a live session from the entry's `ensureSession`, which only the clean-and-gated Continue case stops.
  - Both rows assert the session's liveness at each tree read is `[true, false]` (`recordLiveAtRead`, `assertStoppedBeforeTheReRead`) and the session counts.
  - The row for a recorded ending starts no session at entry, so its counts are 0 starts and 1 stop.
  - The liveness sequence is what fails without the stop case: the end of a call stops a live session whatever the wake did, so the counts alone passed with the case removed (found by the self-check).
  - Commits: `landingshed: fix F1 — pin the session stop …` and `landingshed: fix F1 — assert the session is stopped before the re-read …`.
  - `lym gate test ./internal/landingshed`: tier0 passed.
- Gate finding: the `recordLiveAtRead` comment wrapped mid-sentence; rewritten as one sentence per line, and committed as `landingshed: fix gate finding — break the recordLiveAtRead comment at sentence boundaries`.

## Self-check

- fix-code-correctness, MEDIUM: the count assertions pass with the stop cases removed; fixed by the liveness sequence above.
- fix-proposal, MEDIUM: same finding as fix-code-correctness; merged, fixed as proposed.
- fix-consistency, LOW: the closed-PR twin row `ReviewEndedOnAClosedPullRequestOnADirtyTreeHaltsNamingTheFiles` has no live session or stop assertion; disputed, `every fix worse`-style no: it adds no coverage the open-PR row lacks, so left as it is (see Disputed).
- fix-affected-set: no findings.

## Disputed

- fix-consistency LOW (twin closed-PR row): premise holds but is no defect. The open-PR row alone pins the `rec.ReviewEnded` case, and a twin assertion would be a redundant test under the testing skill's one-test-per-behavior rule, so none was added.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached me.

---
verdict: APPROVED
---

## Scope

The board entry asks that the `TestWatchdogReap` wait for the reap loop to remove the orphan session be derived from a bound that tolerates a slow host, such as one running Cortex XDR, instead of the 1.8 s `DiscoveryCycle*OrphanGoneCycles*20` window.
The decision record has no `Added after Discussion` entries, so the board entry is the measure.

PR 778 at `9b1e5204` changes only `internal/reedcli/watchdogreap_integration_test.go`.
It adds the named constant `reapSuccessWait` (30 s) and uses it for every `waitForCondition` that waits for an expected success: the six orphan-gone waits and the fresh-session wait in the in-flight subtest.
The named failing subtest, `NeverEnteredOrphanIsStillReaped`, is among them.
The sibling subtests use the same cadence-derived window and fail the same way on the same host, so widening them all fixes the one defect the entry reports and is not work for another entry.
`compressedReapTiming`, `OrphanGoneCycles` and every production file are unchanged, and the early-reap check keeps its short bound, so the test still proves the consecutive-cycle rule.
No point of the entry is left out and nothing it does not call for is added.

## Open risks

The record's one open risk, a host slower than 30 s, names no parent correction; it does not escalate.

## Related board entries

`xdr-git-test-file-burst` and `bench-osl1046-xdr` concern Cortex XDR on the same machine, but cover git fixture file bursts and gate-tier timing; this run makes neither obsolete and duplicates neither.
`origin-reconcile` and `run-reentry` depend on this entry and are not running; this diff does not touch their scope.

## Overlap

- Peer `lenses-module` (row 6, Discussion-Burler-Fix): no changed paths yet, and its decision record does not touch reed, the watchdog or this test. No overlap.
- Parent commits: none listed.

No overlap, so no landing order applies.

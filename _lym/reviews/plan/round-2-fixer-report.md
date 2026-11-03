# Round 2 fixer-report

Every finding's premise held against the code. `lym loom validate-plan` passes after every change.

## Changes

- **F1** (card 6), fixed as proposed.
  The `ArchiveWeftTipForTest` Edit entry now carries the re-sign arrow to a wrapper function.
  The Callers paragraph says it wraps `archiveWeftTip(rec, l, slug, weftBranch, false, nil)` and returns `localTag`, the skipped reason and the error, so the four tests keep their call shape.
- **F2** (card 13, overview), fixed with these additions:
  - The cancel site: `judgeOrigin` swaps its trailing re-run for `BattenFollowUp`'s text, and `cancelDeps` gains `OwnerHeld`, which `hubCancelDeps` fills.
  - The landing site: landingshed `Deps` gains `RemoveFollowUp`, which `reconcileVerdict` and `reportVerdict` pass to `stoppedReason`, and loomcli's `landingDeps` fills it.
  - One test row each in `TestCancelRun` and `TestFinalize_StuckReasons`.
  - Decision `d3-sites-outside-fabric` lists both sites as carried and states why the adopt, cleanup and wake sites carry nothing.
  - Fixed differently from the proposal: the Edit list is wider than the proposed three glyphs (`judgeOrigin`, `stoppedReason`, landingshed `doc.go`). `validate-plan` required the wider list once the follow-up was wired as a value the caller passes in:
    - the re-signed verdict helpers and their callers `Finalize.ownerChecks` and `ParentAlign.Call`;
    - `cancelDeps`, `hubCancelDeps`, `cancelFake` and `landingDeps`;
    - envkit's `nilLegal`, which lists `Landing.RemoveFollowUp` as a nil default;
    - `internal/loomcli/doc.go` under Extends;
    - the reflecting seam tests under Uses.
- **F3** (card 8), fixed as proposed: Create `SetCleanupAfterEnumerateForTest`, Edit `Topology` with `cleanupAfterEnumerate`, the no-entry rule for an orphan found live under the lock, and a test row driven by the hook.
- **F4** (card 10), fixed as proposed: `armAt` sets `readopt` for verb run. The Edit list swaps `Command`, `StartFirstPass` and `TestStartFirstPass` for `battenCLI.armAt`, and the `TestStartFirstPass` bullet is dropped. ImpactSummary updated.
- **F5** (card 10), fixed as proposed: the `taskWorktreeGone` row reads "an absent path reads gone (true) with a nil error, by a stat alone".
- **F6** (cards 8 and 10), fixed as proposed:
  - Card 8 lists the doc comments it falsifies: `Topology.Cleanup`, the `cleanup.go` header's apply line and `CleanupRemoteWarp`, which is now under Edit.
  - Card 10 adds the gone-pair case to the `ownerPreflight` doc comment.
- **F8** (card 13), fixed as proposed:
  - `TestTeardown_CallSequence` rows reduced to wiring (prefix plus `BattenFollowUp`'s text, owner-held, `LeaveClaim`), with the Land row dropped.
  - `TestPreflightRunOwner_RemoveFollowUp` reduced to two rows.
- **F9** (card 10), fixed as proposed: the two Add-level rows are dropped from `TestBattenIntegration_GoneTaskWorktree`, and the closing note credits those criteria to card 4's `TestAdd_AdoptOnly`.
- **F10** (card 4), fixed as proposed: the warp-probe bullet states that card 3's warp drop under `AdoptOnly` covers an adopted local warp branch origin lacks.
- **F11** (card 5), fixed as proposed: the test is parallel, with the reason.
- **F12** (card 5), fixed differently. The proposal named "the existing tier1 test that drives `lym fabric push` against a real pair", but no such test exists for an ordinary pair.
  - The only tier1 test that drives `push` is `TestRunCLI_LocalPairScenario` (`internal/fabriccli/localpair_integration_test.go`), and it pushes a local pair. Its ordinary pair is torn down by hand within one subtest.
  - Card 5 instead creates `TestRunCLI_PushSkippedWhileTeardownLockHeld` (tier1, parallel) with the row's assertions: lock held exclusive, an ok envelope with `push_skipped`, and origin's weft tip unchanged.

## Self-check

- fix-consistency, LOW: card 6's ImpactSummary still said "test re-export". Fixed as proposed: it now says "test wrapper".
- fix-affected-set, MEDIUM, and fix-claims, MEDIUM (merged): the overview's claim that the adopt, cleanup and wake sites are "neither a stop nor a refusal" is false for adopt and cleanup. Fixed as proposed. The decision now gives the true ground:
  - `sharedStopText` keeps only shared branches, and the prime has no batten record, so the follow-up there is always empty.
  - Adopt's warning line is a report.
  - Wake's `runStep` is a report step.
- fix-proposal, MEDIUM: on Finalize's parent-already-holds-the-work path, the follow-up would name a cancel for landed work. Fixed as proposed: `Finalize.ownerChecks` passes nil on that path, the overview names the exception, and `TestFinalize_StuckReasons` gains the row.
- fix-proposal, LOW: card 10's "every hand-built test receiver" is wrong for tests that call a run-verb arm. Fixed as proposed: it now says "every hand-built test receiver that calls no run-verb arm".

Every fork returned a report.

## Disputed

None.

## Deferred

None.

# Fixer-report, webster round 1

## Changes

- **F1** fixed as proposed.
  `readMailbox` reloads the state with `LoadState` when the acknowledgement fails after the handler ran, so `creelFailed` saves the handler's state.
  Row added to `TestWatcher_CreelFailuresHoldTheCycle`; the fake creel gained a scripted `acknowledge` failure.
- **F2** fixed differently from the proposal.
  Proposed: treat a `ClearResumeNoticed` error as a withdraw failure and skip the release.
  That alone leaves the defect, since the retry's withdraw answers an empty list and the line is never seen again.
  Made instead: the error joins `failures` (release still runs), and the removed line is kept in the new `State.ResumeLineWithdrawn` until the clear succeeds; the retry feeds it to `ClearResumeNoticed`.
  The field is cleared by `ResetForFreshLaunch`; `doc.go` updated; row added for a withdraw that removes a resume line while the clear fails.
- **F3** fixed as proposed.
  `TestCreelSeam_MapsEachCallOntoTheClient` covers `Post`, both `Inject` branches with ttl, `ReadMailbox`, `Hold`, `Release` and `Withdraw` over a recording `fakeCreelClient`.
  `cmd/lyx/testdata/test-budget.yaml` raised for `internal/orchcli`.
- **F4** fixed differently from the proposal.
  Proposed: inject a short timeout through an unexported `GoClient` field.
  Not needed: the client's clock is fake in the rig, so a subtest of `TestCreelCLI` advances it by `goCallTimeout` against a listener that never answers and asserts `ErrWaitTimeout` for every `GoClient` method.
  No production change, and no new top-level test.
- **F5** fixed as proposed, and extended to the sibling comments in `battenshed` (`land.go`, `rownotice.go`, `doc.go`) that described the same notice as queued.
  The `queued` identifiers and log key are unchanged.
- Fork finding on `resetForOtherStrand`: same defect as F2; fixed with the same `State.ResumeLineWithdrawn`, plus unbinding the watcher so the reset reruns; row added to `TestWatcher_PhaseForDifferentStrandIsReset`.

F1 and F2 share one commit, as they share the files and the test; every other fix has its own commit.

## Disputed

None of the review's findings.

## Deferred

- `internal/orchcli/start.go` `clearResumeNoticed` logs a Warn and drops the withdrawn line on a failed clear.
  The comment there accepts that as the worst outcome and this round did not change it; it runs in the `start` CLI process, which has no retry loop to carry the flag.
  Not part of any finding.

## Self-check

- fix-consistency, LOW, sibling "queued" comments in `battenshed`: fixed as proposed.
- fix-consistency, LOW, `resetForOtherStrand` twin of F2: fixed with own change (same field as F2).
- fix-consistency, LOW, `orchcli/start.go` twin: deferred, reason above.
- fix-proposal, LOW, `queued` identifiers and log key still name the retired queue: not changed; the review said to keep behaviour unchanged, and a rename touches the log key.
- fix-affected-set, LOW, sibling "queued" comments: same as the consistency finding, fixed.
- fix-affected-set, LOW, `resetForOtherStrand`: same as above, fixed.
- fix-code-correctness, LOW, the F4 subtest passes a call with a shorter timeout than `goCallTimeout`: not changed; a check that the call is still pending at 29s needs a real-time wait to be deterministic, which the test rules forbid.
- fix-code-correctness, LOW, `resetForOtherStrand`: same as above, fixed.

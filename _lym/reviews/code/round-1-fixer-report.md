---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The proposed in-place retry loop with an injectable sleep is not made. Finalize's post-merge parent fetch now returns the transient error through transientFailure, so the driver's existing network rows wait over their re-step schedule and, once it is spent, hand back with a notice naming the failing fetch. The re-entry finds the pull request merged and lands through the already-landed path (the stop reason's own wording). An in-place loop would add a second schedule beside the driver's, and its halt would still reach the parent's landing-halt row with no network notice."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "The review's site was not found. Both weft paths it names already back off over gitrepo.PushBackoffDelays: awaitOrigin retries an unreachable origin before a branch with an upstream is pushed, and TestPushFailureClasses pins it (pushclasses_integration_test.go:661-717, :809-828). The real gap is a push that itself fails in transit, which went straight to a failed-push record. pushJob.retryTransportPush now pushes again over the same delays through BackoffSleep before the failure is classified, with a test row that fails once and lands."
  - id: F8
    disposition: fixed
---

# Fixer-report, round 1

## What I changed

- F1, `internal/landingshed/finalize_pr.go`: a failed `ReadParentTips(true)` after the GitHub merge goes through `transientFailure` and returns `&ending{err: terr}` for a transient failure; any other failure still halts with `LandingStopPrefix`. `mergeOnGitHub`'s doc says so.
  `finalize_pr_test.go` gained two rows, driven by a `tipsErr` field that fails only the read after the merge: a transport `*GitError` returns an error naming the fetch, and a refusal still halts naming `lym loom resume`.
- F2, `fabricengine.RunStartReconcileOptions` (one minute in ten-second steps, no sleep seam of its own) is passed at the three named sites:
  `battencli.reconcileRun`, the gone-pair re-adoption `reconcileGoneRun` through `hubReadoptSeams`, and `beamcli`'s `reconcileEnded` through `tearDownLeftovers`.
  Batten takes the bound from a new `battenCLI.reconcileOptions` field, set in `Command`, `StartFirstPass` and `ArmAt`, and zero on a hand-populated receiver, so the existing fixtures keep one attempt.
  `readopt_integration_test.go` gained a row that points both origins at a refused URL, restores them in the `Sleep` seam, and asserts `reconcileResume` returns nil with one retry and a reachable report.
  The cancel verb's judge-only reconcile keeps zero options.
- F3 and F8, `internal/gitexec/transport.go`: a stderr holding an HTTP 4xx status is not a transport failure, whatever disconnect phrase follows it, except 408 and 429.
  `transport_test.go`: two 413 rows (sideband disconnect, send failure) expect false, two rows with 429 and 408 plus a disconnect expect true, and the sideband and early-EOF phrases are rows of their own.
- F4 and F5, `internal/sheddriver/drive_test.go`: `TestGoDriver_Settle` gained `waitErr`, `waitMovesTo` and `wantRunErr`, and two rows.
  One has the Wait stub return `context.Canceled` at the first network-failure stop and asserts `Run` returns it with no re-step, settle record or hand-back.
  The other moves the status by an orch goto once the wait has run, and asserts a moved record and a re-step at the new row.
  `TestGoDriver_Wait` pins the real-clock branch: an ended context returns its error without waiting an hour, and an elapsing delay returns nil.
- F6, `internal/sheddriver/doc.go` and `pattern/PATTERN-shed-driver.md` name `Wait` and say an unwired one removes no row.
- F7, `internal/fabricengine/pushclassified.go`: `retryTransportPush` as described above.
  `pushclasses_integration_test.go`: a new row (a never-pushed branch whose origin is refused, restored by the first backoff sleep, lands with no record and no mail), and the existing never-pushed transport row now passes a no-op backoff so it no longer waits out the real delays.

Gates: `lym gate test` tier0 over the six touched packages and tier1 over `fabricengine`, `battencli` and `beamcli` were green before the commits, and tier0 over `gitexec`, `battencli` and `fabricengine` was green after the self-check changes.

Gate repair: five comment lines of this round reached 120 characters (`originreconcile.go`, `transport.go`, `drive_test.go`).
They are now split at sentence boundaries, with no change to code or tests.

Second gate pass, reach report: it lists no failing test.
Its unmapped functions (`reconcileRun`, `reconcileGoneRun`, `reconcileEnded`, `hubCancelDeps`, `StartFirstPass`, `hubDobbyDeps`) are reached only by tier1 integration tests, which no coverage map holds.
The tier1 tests that reach them ran green: the new `readopt_integration_test.go` row covers `reconcileRun`, and the leftover teardown test covers `reconcileEnded`.
`reconcileGoneRun` is reached by the existing gone-pair rows, now through `hubReadoptSeams`.
`hubCancelDeps`, `StartFirstPass` and `hubDobbyDeps` changed only by one argument.
No test was added, because a tier0 test of these functions needs a real hub, and nothing changed after the commit.

## Disputed

Nothing in the review is disputed.

## Deferred, with reason

None of these is a review finding; each came from a self-check fork and sits beyond the sites a finding names.

- The hand pass's catch-up fetches (`internal/battencli/catchup.go`: `pullBoltForCatchUp`, `registerThisMachine`, `fetchRunForCatchUp`, and the fetch in `hubRunVerdict`, `arm.go:638-642`) map any fetch failure to `ErrOriginUnreachable` with one attempt.
  They are not reconcile calls, and `HubCatchUp` has no options seam to carry a bound.
  Where the bound lives and how its callers inject a short one is a design call the review's F2 fix key did not make, so it belongs to the parent.
- The other zero-option reconciles that are not run-start sites: `internal/loomcli/landingdeps.go:874` (a live run's landing, under the parent lock, where a longer wait changes how long the lock is held), the operator verbs and judge-only reads (`fabriccli/remove.go:226`, `cleanupcli/reconcile.go:36`, `orchcli/start.go:385`, `battencli/cancelcmd.go:204` and `:371`, `battencli/wire.go:1291`, `loomcli/ownercheck.go:65`).
  Whether any of them waits is the parent's call.
- `internal/fabricengine/takeoverwrite.go:198`, the takeover entry's own push, is a separate verb path that F7 does not name.

## Rulings

None reached me this round.

## Self-check

Forks returned a report: fix-code-correctness, fix-consistency, fix-proposal and fix-affected-set. None was lost.

- fix-code-correctness, LOW, `ArmAt` builds a receiver with zero reconcile options: fixed as proposed (`battencli/arm.go`).
- fix-consistency, MEDIUM, `ReadParentTips(true)` in `parentalign.go:236` and `finalize.go:503` still halts on a transport failure: disputed, premise false. Both sites wrap the error with `%w` and return it (`finalize.go:503-505`, `parentalign.go:236-238`); neither halts, so the existing path surfaces it as an error.
- fix-consistency, MEDIUM, other run-start reconciles and the catch-up fetch still make one attempt: deferred, see above.
- fix-proposal, MEDIUM, the catch-up fetches leave #203's refused hand pass: deferred, see above.
- fix-proposal, MEDIUM, a two-minute bound stalls every run start while origin is down, under the remote-refs locks: fixed differently. The bound is one minute, the least the entry asks for ("a minute or more"), and the doc says why it is short.
- fix-proposal, LOW, the blanket 4xx guard silences 408 and 429: fixed differently. The guard reads the status and skips those two; two rows pin it.
- fix-affected-set, MEDIUM, `loomcli/landingdeps.go:874` still makes one attempt: deferred, see above.
- fix-affected-set, LOW, five more zero-option reconcile sites: deferred, see above.
- fix-affected-set, MEDIUM, the catch-up fetch path: deferred, see above.
- fix-affected-set, unchecked, the other `ReadParentTips(true)` sites: disputed, premise false, as for fix-consistency above.
- fix-affected-set, LOW, `pushclassified.go:390`, `:934` and `:1019` do not retry a transport failure: disputed, every fix worse. All three sit inside the bounded rejection retry, whose `rejectionRetryKind` marks a transport failure not retryable (`pushclassified.go:332`), so a retry there would nest a second backoff in the first.
- fix-affected-set, LOW, `takeoverwrite.go:198`: deferred, see above.

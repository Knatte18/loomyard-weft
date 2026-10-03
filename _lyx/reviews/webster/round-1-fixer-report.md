# Webster round 1 — fixer report

Every finding in `round-1-review.md` is fixed, each in its own commit.
After the fixes, `go build ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration ./internal/loomcli/...` pass.

## Fixed

- **F2** (`e7a32008a`): `TestStepCmd_BusyRefusal_BeforeBootstrap` now seeds a status file whose history length and state match the recorded step clean-handoff marker, so an observation taken before the busy probe would consume the marker and set `entryObservation`.
  The bootstrap-not-run proof moved from the status file's absence to the loom seed's absence (`shedrun.ReadSeed`), and the doc comment says why.
  Checked by temporarily moving `observeStepEntry` ahead of the probe: both new assertions then fail.
- **F3** (`89a890e92`): new `TestDetectAndFileAnomalies_PendingRetryFailsAndDetectedAgain_FiledOnceStaysPendingOnce` pins a pending title whose retry fails and that the same pass detects again: one filing call, one pending entry, no recorded title.
  Checked by temporarily dropping the `isPending` skip: the new test then fails.
- **F4** (`f34df1aa7`): `detectAndFileAnomalies`' filing-pass sentence is reworded so the status-file clause has a verb.
- **F1** (`76b24a0ba`): card 10's doc comments in `selfreport.go` (`detectFinalAnomalies`, `runFilingPass`, `pendingAnomaly`, `selfreportFiledMarker`), the new test doc comments in `selfreport_test.go` and the changed doc comment in `selfreport_github_test.go` now use semantic line breaks.

## Deferred

None.

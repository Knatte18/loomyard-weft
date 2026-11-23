```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. List each change's decision trace and each decision's builders, and report scope findings.

**Per change → decision**

- `internal/statuscommit` (drops Push/Admit/Owed/Sleep, adds `StartPush`): D1.
- `internal/battencli/commitstatus.go` (the seam never pushes, the start-row exception and `push_pending` retire, `commitRunRecords` starts the pusher): D1.
- `internal/loomcli/wiring.go`, `commitrecords.go` (no push, `pusher_started` envelope): D1.
- The `loomcli/prepush.go` check moves out of `commit-records` and the status deps: D7. It is a test-only change in the diff; the guard now lives in `pusher.go`.
- `internal/runpusher` (`doc.go`, `target.go`, `start.go`, `run.go`, `pushuntillanded.go`), `lymdirs.PusherDirName`, `cmd/lym/daemonwakeups_test.go` (adds `pushuntillanded.go` to `unboundedWaitFiles`): D3, D5 (owed backoff) and D7 (pause read before the guard). The `daemonwakeups_test.go` edit is forced by the moved backoff.
- `loomcli/pusher.go` (hidden `pusher` verb, `resolvePersistentPreRun` skip): D3, D6 (`InsidePair`) and D7.
- `fabricengine` `OwedUnrecorded` (`fabric.go`, `pushanchored.go`, `pushclassified.go` `record`/`deferral`, `PairPush`, `RunPush`): D5 and D8.
- `ErrEntryEnded`, `entryEndedError`, `dropTrackingRefOfEndedBranch`, `gitrepo.DeleteRemoteTrackingRef`, and the `UpstreamSHA` absent-ref error: D5. They implement "pruned tracking ref reads level". This is a forced edit of the ahead reads, though the record does not spell out the deletion.
- `fabricengine/pusherreads.go` (`RunCommitsAhead`, `RefusedPushStands`, `TipsOnOrigin`): D5 and D3.
- `takeover.go`, `retryWeftPush`, `replay.go` (`EndedHandedOver`), `handover.go` (`PrimeRecords`), `pairteardown/handover.go` (`PushRun` loop): D6.
- `pairteardown/runread.go` (`localHoldsTip`): D6 (`CodeTipMissing` stays unset).
- `battenshed` (`teardown.go`, `deps.go`, `doc.go`), `battencli/wire.go` (`Teardown.Push`/`Level`), `envkit` `nilLegal`: D5 (Worktree-Teardown). The `nilLegal` entries are a forced edit.
- `beamcli/dobbydeps.go` and `battencli/dobbyreads.go` (tick restarts): D5. `battencli/arm.go` and `cli.go` (`startPusher` on resume): D5. `loomcli/resume.go` (`startPusher` on resume): D5.
- `battencli/wire.go` `c.startPusher`: D5 resume. `loomcli/wiring.go` `c.startPusher`: D5 resume.
- `adoptcli` integration test: D2 acceptance.
- `heddleengine/glossary.md`, `PATTERN.md`, `pattern/*`, `docs/overview.md`, `tendercli/cli.go`, `tenderengine/report.go` comments, doc.go updates: the PATTERN and glossary changes are D3 and the declared changes. The `tendercli`/`tenderengine` comment edits are forced by the transition no longer pushing.
- `cmd/lym/testonlycode_linux_test.go` (allows `fabricengine.IsPushRejected`): forced, since commit-records' push left.
- `slackengine/relay_test.go` (`r.settle()` instead of `r.start()`): **none**. It is unrelated to any decision. Its covering commit message may be a `Loosened-Check:` or `Root-Cause:` trailer, which I cannot see.

**Per decision → builders**

- D1: statuscommit, battencli commitstatus, loomcli wiring and commitrecords. Covered.
- D2: no code change by design ("no machinery"); the adopt test pins it. Covered.
- D3: runpusher, the `pusher` verb, `lymdirs`, the glossary. Covered.
- D5: owed mode, ahead reads, dobby tick, resume starts, teardown. Covered.
- D6: takeover, handover, replay, `pairteardown.PushRun`, `runread`. Covered.
- D7: `pusher.go` guard and pause read, and the check removed from commit. Covered.
- D8: `OwedUnrecorded` leaves `PushOwed` unset. Covered.

**Findings**

1. LOW / scope, `internal/slackengine/relay_test.go:1103`. The change swaps `r.start()` for `r.settle()` in the relay test. No decision traces to it, and no `Loosened-Check:`, `Root-Cause:` or `Pattern-Departure:` trailer is visible to me. The commit `2198f259c` ("slackengine: let the relay settle…") may be a separate fix that landed on the branch. Fix: drop it from this branch's diff, or carry a trailer naming the measured cause. Evidence: the `relay_test.go` hunk in `019-internal-slackengine.diff`.

2. LOW / scope, `internal/fabricengine/pushclassified.go` `dropTrackingRefOfEndedBranch`, and `internal/gitrepo/push.go` `DeleteRemoteTrackingRef`. D5 says a pruned tracking ref reads level and is left to teardown. It does not ask for deleting the tracking ref of an ended branch on push, nor for a new gitrepo API. This is arguably forced by the "stale ref lacks local tip" read, but the record never licenses it. Fix: confirm plan card 1 or 2 carries it; if not, treat it as unrequested. Evidence: the `checkOrigin` hunk in `010-internal-fabricengine-part2.diff`.

3. LOW / scope, `internal/fabricengine/pushanchored.go` `ErrEntryEnded` and `entryEndedError`, with the `Teardown.Push` mapping to `ErrPushEntryEnded` in `battencli/wire.go`. D5 says an ended branch is left to its teardown and does not mention an ended-entry skip in the teardown's push. The teardown's "entry ended removes without a Level read" branch is added behavior D5 does not state. It is needed so teardown does not loop, but it is unrecorded. Fix: confirm a plan card carries it. Evidence: the `teardown.go` `pushPair` hunk in `007-internal-battenshed.diff`.

I raised no decision-unbuilt findings. Every D traces to at least one change.

Tool calls: none. My context already held every diff and the decision record.

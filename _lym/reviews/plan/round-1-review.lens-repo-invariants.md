```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked three things: reflective and seam tests, doc text the card makes false, and whether the card can pass its gate from the state the earlier cards leave.

**Per card: each invariant checked, held or broken**

- **Card 1:**
  - Seam tests: held. The renamed field's only users are `runfsm/act_test.go` and `battencli/cancel_test.go:1112`, and both are listed.
  - Docs: held. `runfsm/doc.go` is listed.
  - Order: held.
- **Card 2:**
  - Seam tests: held. No reflective seam covers `Status`, and `TestStatus_JSONRoundTrip` is listed.
  - Docs: held. Both `doc.go` files are listed.
  - Order: held.
  - New test file tier: n/a.
- **Card 3:**
  - Seam tests: n/a.
  - Docs: held. `fabricengine/doc.go` is listed.
  - Order: held. Every caller of `PushPairAnchored` is listed: production callers in fabriccli `weft_verbs.go:286`, beamcli `deps.go:315`, loomcli `landingdeps.go:393` and `wiring.go:222`, plus all test callers. The test callers sit in `newPairFixtureAt`, `TestPushFailureClasses`, `TestPushPairAnchored_*`, `TestPushLock_*`, `TestTeardownLock_*`, adoptcli, battencli `TestMain` and the cleanup tests, and cleanupcli and fabriccli `TestMain`.
- **Card 4:**
  - Docs: broken. `internal/fabriccli/doc.go:93` still says "the push children it respawns". That is the failed-remove respawn (`respawnPairPush`), which card 4 retires. No card lists `fabriccli/doc.go` as an Edit target; cards 3 and 5 list it under Uses only. (F1)
  - Order: held. `TestCoalescePushRequest_CoveredCallDefers`, `TestRemove_*` and `TestCoalesceFlagged_*` are listed.
- **Card 5:**
  - Order: held. Every caller of `CoalescePushRequest` and `DetachedPush` sits in a listed test or target: `TestPushFailureClasses`, `TestPushLock_*` at `:414`, `TestCoalescePushRequest_CoveredCallDefers`, `TestRemove_TeardownLockAgainstThePairsPush`, `TestDetachedPush_CoveredChildExitsBeforeItsFlags`, the `DetachedPush` body at `teardownlock.go:111`, and `addWeftVerbs`. `CoalescePushBothAt` keeps its signature, so its callers in hubforge and the fabriccli tests are unaffected.
  - Docs: held.
- **Card 6:**
  - Docs: held. The pairteardown and fabricengine docs are listed.
  - Order: held.
- **Card 7:**
  - Seam tests: held. `runFakes` and `TestReadRun` are listed.
  - Docs: held.
- **Card 8:**
  - Seam tests and order: broken. Several tests set the `Spawn` field that card 8 removes, and card 8 does not list them, so the card's gate cannot compile them. (F2)
  - Docs: held. statuscommit's package comment and battencli's `commitstatus.go` header are covered, and `cmd/lym#unboundedWaitFiles` is listed.
- **Card 9:**
  - Docs: held. `tendercli/doc.go:46` ("spawns the push") is listed.
  - Order: held. `cancelFake.deps` (`SpawnPush`) is listed. `TestPullRequestDeps_CommitsTheRecord` swaps the `spawnStatusPush` var that card 9 deletes; it is listed for Edit.
- **Card 10:**
  - Docs: held. `boardengine/doc.go` is listed.
- **Card 11:**
  - Seam tests: held. The four new landing `Deps` fields are covered by `landingDeps` together with `TestLandingDeps_EveryFieldPopulated` and envkit's `nilLegal`.
  - Docs: held. `landingshed/doc.go:549` ("before the push, Finalize marks the board task done") is in the listed `landingshed/doc.go`.
- **Card 12:**
  - Docs: held. `fabricengine/doc.go` is listed. The beamcli `DobbyRuns` tests are named as running unchanged.
- **Card 13:**
  - Seam tests: held. The tests that build `DobbyRunCells` are listed.
- **Card 14:**
  - Docs: held. `PATTERN.md`, `PATTERN-pair-remote.md` and `PATTERN-daemon-wakeups.md` are listed. No other `pattern/` file describes the retired spawn or respawn.

**Findings**

- **F1: LOW, consistency.**
  - Location: card 4 (`04-detached-records-first.md`).
  - `internal/fabriccli/doc.go:93` says the removal's "push children it respawns keep their own gates". Card 4 removes the failed-remove respawn (`respawnPairPush`, `teardownlock.go:154-178`), so that doc line becomes false.
  - No card lists `internal/fabriccli/doc.go` as an Edit target.
  - Fix: add `internal/fabriccli/doc.go` to card 4's `**Edit:**` and drop the sentence.
- **F2: BLOCKING, design.**
  - Location: card 8 (`08-transition-sync-push.md`).
  - Card 8 drops `statuscommit.Deps.Spawn` and loomcli's `Spawn` field. These tests set `Spawn:` and are not among card 8's targets, so the card's tier0 gate cannot compile them:
    - `internal/loomcli/wiring_commitstatus_test.go` `TestNewCommitStatusSeam_Finish` (`Spawn: func() error { return nil }`).
    - `internal/battencli/commitstatus_test.go` `TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed` and `TestNewCommitStatusSeam_BoardWriteFailureDoesNotStopTheCommit`.
    - Probably also `internal/battencli/driverdeps_test.go` `TestBattenRecordsCommitSeams` and the `newFakeReceiver` helper in `internal/battencli/run_test.go`. Both build a `Spawn: func() error` deps value; I did not confirm which struct type each one builds.
  - Fix: add these test members to card 8's `**Edit:**` and state each one's `Spawn`-to-`Push` swap.

Tool calls:
1. Grepped the callers of `PushPairAnchored`, `CoalescePushRequest`, `DetachedPush` and `CoalescePushBothAt` across `internal` and `cmd`. The manifest named the definitions but not every caller of the re-signed entries.
2. Ran awk to map those caller lines to their enclosing test functions, so each could be checked against the cards' Edit lists.
3. Grepped docs for retired spawn and respawn wording (output too large) to find doc text the cards make false.
4. Narrowed that doc grep and listed the test files that use `reflect`. The manifest held no docs outside the listed `doc.go` files and no seam-test inventory.
5. Read `fabriccli/doc.go:88-96`, the `reflect` and test names in the commit-status tests, and the push-related lines in the loomcli, battencli, beamcli and landingshed docs, to confirm the doc hit and check for seam tests.
6. Found every test that sets the retiring `Spawn` or `SpawnPush` seams, to check card 8's and card 9's Edit lists for missing tests.

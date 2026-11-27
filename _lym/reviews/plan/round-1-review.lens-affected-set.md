```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each change a card makes, I listed what reads or asserts it and which cards cover that set.

**Per change: the set found, and the cards that cover it**

- **Card 9's `RecordsHold` and `CodeCeiling`.**
  - Set: every push of a run's pair records or of a beam pair.
    - `PushRun` callers: pusher, catch-up, PR-Review `PushTask`, the pull request record committer, codetipsettle, the hand-over, teardown seams, `PushBeam`, commitstatus, tender, cancel.
    - Direct `PushPairAnchored` in `lym fabric push` (`internal/fabriccli/weft_verbs.go:314`).
    - The `DetachedPush` bypass (`internal/fabricengine/teardownlock.go:98`).
    - `lym beam pr`, which uses `PushBeam`'s returned head (`internal/beamcli/pr.go:81`, `:175`).
  - Covered: card 9 covers the `PushRun` callers. No card covers the direct `PushPairAnchored` call, the bypass, or the beam pr head.
- **Card 7's spawn of `lym loom pusher` in place of `fabric push`, registered under the hub owner.**
  - Set: `TestPushSpawnCommand`, `TestSpawnStatusPush`, `TestSpawnSync_RegistersUnderTheHubOwner`, `TestStartDetachedPush_RegistersTheChild` (`internal/fabricengine/spawn_linux_test.go`), `TestSpawnDetachedPush_SkipEnvAndEmptyPaths`, the callers of the spawns, and the docs.
  - Covered: card 7 covers all of these except `TestStartDetachedPush_RegistersTheChild`.
- **Card 12's removal of `parentMergeArchiver`, `runRecordsPusher` and four `Deps` fields.**
  - Set: `landingDeps` (`internal/loomcli/landingdeps.go:189`), `envkit` `nilLegal` (`Landing.PushRecords`), `landingshed/deps.go`, and `fabricengine.ArchiveParentMerge`'s last production caller, with `TestArchiveParentMerge` and the line at `internal/fabricengine/doc.go:162`.
  - Covered: card 12 covers all of these except `ArchiveParentMerge`, its test and its doc line.
- **Card 5's `Deps` losing `Notify` and `Sleep`, and `NewDeps` re-signed.**
  - Set: `run_test.go` (`runFake` at lines 87–88), `newdeps_integration_test.go`, and `loomcli` `pusherDeps` with its test.
  - Covered: card 5.
- **Card 19's deletion of `PushUntilLanded`.**
  - Set: `run.go`, `wire.go`, `runpusher/doc.go`, `unboundedWaitFiles`. No test references it.
  - Covered: cards 5 and 19.
- **Card 2's deletion of `networkNotice`, `networkRestepWaits` and `networkRestepSpan`.**
  - Set: `driverules.go`, `drive.go`, `driverules_test.go`.
  - Covered: card 2, through `decideDrive`, `driveDecision`, `handBackStop` and `TestDecideDrive`.
- **Card 8's widening of `RunCommitsAhead`.**
  - Set: `runpusher` `run.go` and `start.go`, `TestRunCommitsAhead`, and `adoptcli/adopt_integration_test.go:103`.
  - Covered: card 8. The adopt test only asserts true, so the widening keeps it passing.
- **Card 21's change to `statusFormatSessions`.**
  - Set: `windowsize.go:54`, `windowsize_test.go:60`, and the `#{S:}` scan in `bar_test.go:115`.
  - Covered: card 21 edits the constant itself. Both tests compare against the constant, so they need no edit.
- **Card 16's change to `Fabric.Pull` on the prime.**
  - Set: batten Land (`battencli/land.go:103`), `lym fabric pull` (`weft_verbs.go:377`), `PullHub`.
  - Covered: card 16 edits the shared function.

**Findings**

1. **BLOCKING · design · card 9 (`09-records-hold.md`)**
   - Summary: the records hold is applied only inside `pairteardown.PushRun`. `lym fabric push` in a task worktree calls `fabricengine.PushPairAnchored` directly, with no `RunSlug` and no hold, so it can send the records commit a landing entry holds. D1 says no synchronous push of a run's pair records sends it, and D5 says the hold binds every such push.
   - Members missed: `internal/fabriccli/weft_verbs.go:314` and the `DetachedPush` bypass at `internal/fabricengine/teardownlock.go:98`.
   - Evidence: `internal/fabriccli/weft_verbs.go:314`, `internal/pairteardown/handover.go:95`.
2. **MEDIUM · consistency · card 9 (`09-records-hold.md`)**
   - Summary: `CodeCeiling` changes what `PushBeam` pushes. While landings stand on the beam, `lym beam pr` and its edit path then open or edit a pull request against a head whose landing commits origin's beam branch lacks. `PushBeam` returns `pair.HeadSHA()`, the local head. No card lists `PushBeam` or `pr.go`.
   - Evidence: `internal/beamcli/deps.go:371-379`, `internal/beamcli/pr.go:81`, `internal/beamcli/pr.go:175`.
3. **MEDIUM · consistency · card 7 (`07-spawns-start-the-pusher.md`)**
   - Summary: `pushSpawnCommand` now registers a pair's spawn under the hub owner with the push role. `TestStartDetachedPush_RegistersTheChild` asserts the row `PairPushOwnsUnderItsSlug` (owner `task`), which then fails. Card 7 does not list that test.
   - Evidence: `internal/fabricengine/spawn_linux_test.go:25`, its row at line 38, and `internal/fabricengine/spawn.go:93-96`.
4. **MEDIUM · consistency · card 12 (`12-finalize-local-landing.md`)**
   - Summary: deleting `loomcli#parentMergeArchiver` removes the only production caller of `fabricengine.ArchiveParentMerge`. The function becomes test-only code, which PATTERN-test-only-code forbids. No card deletes it, `TestArchiveParentMerge`, or its line in `fabricengine/doc.go`.
   - Evidence: `internal/loomcli/landingdeps.go:919-930`, `internal/fabricengine/handover.go:494`, `internal/fabricengine/archiveparent_integration_test.go:18`, `internal/fabricengine/doc.go:162`.

**Tool calls**

1. Grep of `PushUntilLanded`, the `Notify`/`Sleep` seams, the network-notice symbols, `RunCommitsAhead` callers and the removed landing `Deps` fields: to find the readers of deleted and changed symbols outside the manifest.
2. Grep of the `DetachedPush` users, the `PushSpawn` flag fields, `statusFormatSessions` readers and pusher-notice docs: to find who reads card 7's and card 21's changes.
3. Read of the heads of `coalesce.go` and `pendingflags.go`, `spawn.go` lines 25–115, and a list of tests that assert the spawn line: the manifest lacked the spawn's body and its tests.
4. Grep of the `Test` functions and spawn-seam uses in the spawn, deferral, reconcile and commit tests: to name which tests assert the child line.
5. Read of the `spawn_linux_test.go` owner rows, the `windowsize` and `bar_test` format readers, and `ArchiveParentMerge` docs: to confirm the owner assertion and the bar format's readers.

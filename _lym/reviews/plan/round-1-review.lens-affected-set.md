```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. I traced what each card's change forces outside the card.

**Per change, the set found and the cards covering it**

- **`PushPairAnchored` re-sign (card 3).**
  - Production callers: `fabriccli/weft_verbs.go:286`, `beamcli/deps.go:315`, `loomcli/wiring.go:222` and `loomcli/landingdeps.go:393`.
  - Test callers: `newPairFixtureAt`, `TestPushFailureClasses`, `TestPushPairAnchored_PrimeDivergedWeftRemote…`, `TestPushPairAnchored_PushesBothSides…`, `TestPushLock_BoundedWait…`, `TestTeardownLock_SerialisesAddDiscardAndPairPush`, the adoptcli integration test, battencli `TestMain` and cleanup test, the cleanupcli test and fabriccli `TestMain`.
  - Card 3 covers all of them. It also lists `pullRequestDepsGuardedBy`, which is not a direct caller; that is over-listing, not a gap.
- **`CoalescePushRequest` and `DetachedPush` re-sign (card 5).**
  - Callers: `addWeftVerbs`, `TestPushFailureClasses`, `TestCoalescePushRequest_CoveredCallDefers`, `TestPushLock_BoundedWait…:414`, `TestDetachedPush_CoveredChildExitsBeforeItsFlags` and `TestRemove_TeardownLockAgainstThePairsPush:131`.
  - Card 5 covers all of them. The `CoalescePushBothAt` callers, including hubforge, keep its signature, so they need no change.
- **Respawns retired (card 4).**
  - Test references: the `TestCoalesceFlagged_PendingFlags` swaps at `coalesce_test.go:317,382`. Card 4 covers them.
  - `spawnDetachedPushFn` stays, so the swaps in `export_test.go` and the commit tests still compile.
  - Doc text: the fabric doc is covered by cards 4 and 14, and `PATTERN-pair-remote` by card 14. `internal/fabriccli/doc.go:93` ("the push children it respawns") is not covered (finding 3).
- **`statuscommit.Deps.Spawn` and the seam `Spawn` fields dropped (card 8).**
  - Literals that set `Spawn:`: the statuscommit tests; loomcli `OrdinaryPath`, `BoardStatus`, `StoppedCommitStartsNoPush`, `TestCommitStatusSeam_Real` and `TestNewCommitStatusSeam_Finish`; battencli `commitstatus_test`, `TestBattenRecordsCommitSeams` and `newFakeReceiver`.
  - Card 8 covers all but `TestNewCommitStatusSeam_Finish`, `TestBattenRecordsCommitSeams` and `newFakeReceiver` (finding 1).
- **Per-transition `SpawnStatusPush` retired (cards 8 and 9).**
  - Production spawners that carry run records: battencli `battenCommitStatusDeps` (card 8), loomcli `commitStatusDepsGuardedBy` (card 8), `pullRequestRecordCommitter` (card 9), tendercli `commitRunIntent` (card 9), battencli `hubCancelDeps` (card 9), and pairteardown `Teardown.removeCancelledRunRecord` (no card).
  - The spawns stated to stay: batten cleanup, tender's prime push, fabric's commit spawn, the anchor change and `SpawnPairPush`.
- **loomcli `spawnStatusPush` var deleted (card 9).**
  - Swaps of it: `TestCommitStatusSeam_Real` (card 8) and `TestPullRequestDeps_CommitsTheRecord` (card 9 lists it as an Edit).
  - Covered.
- **`push_owed` refusal text and the `PushOwedOffline` rename (card 1).**
  - Readers: `runfsm/act_test.go` and battencli `TestCancelVerdict`.
  - Covered. No other test asserts runfsm's "which is unreachable" text; the other "once origin answers" asserts belong to cancel and run texts of their own.
- **`tendercli/doc.go:46` ("spawns the push of the prime's records side").** Card 9 edits that doc. Covered.

**Findings**

1. **MEDIUM — consistency.** Card 8 (`08-transition-sync-push.md`) misses three of the test literals its removal of the `Spawn` field breaks.
   - The card drops `statuscommit.Deps.Spawn` and the seams' `Spawn` field. It lists some of the tests that build those structs, not all.
   - Missed: `internal/loomcli#TestNewCommitStatusSeam_Finish` (`internal/loomcli/wiring_commitstatus_test.go:192`, `Spawn: func() error { return nil }`), `internal/battencli#TestBattenRecordsCommitSeams` (`internal/battencli/driverdeps_test.go:226`) and `internal/battencli#newFakeReceiver` (`internal/battencli/run_test.go:74`).
   - Each sets `Spawn: func() error` on a commit-status deps literal. That fails to compile once the field goes, or the test keeps asserting a field that does nothing.
2. **MEDIUM — consistency.** One per-transition spawner of run records has no card and no stated reason to stay detached.
   - `pairteardown`'s `Teardown.removeCancelledRunRecord` still calls `fabricengine.SpawnStatusPush(t.prime, "")` (`internal/pairteardown/followup.go:107`).
   - It writes the batten run record of a cancelled run, which other machines' `ReadRun` reads.
   - No card moves it onto `PushRun`. The overview's out-of-part decision and card 8's list of spawns that stay do not name it, though D1 retires every detached status push that carries a run transition.
3. **LOW — consistency.** Card 4 falsifies the fabriccli package doc, and no card edits it.
   - Card 4 removes the pair-push respawn after a failed `Topology.Remove` (`respawnPairPush`).
   - `internal/fabriccli/doc.go:93` still says the removal's "push children it respawns keep their own gates".
   - Cards 3 and 5 list `internal/fabriccli/doc.go` only under `Uses`, so no card rewrites the line.

**Tool calls**

1. Grepped the callers of `PushPairAnchored`, `CoalescePushRequest`, `DetachedPush` and `CoalescePushBothAt`, the users of `statuscommit`, and asserts of the refusal text. The manifest held definitions, not every caller.
2. Mapped each caller line to its enclosing function. This was needed to compare the callers against the cards' listed targets.
3. Grepped docs and tests for the retired spawn and respawn behaviour, the `Spawn:` literals, and the production callers of `SpawnStatusPush`. The manifest held none of these.
4. Mapped the remaining spawn callers and the respawn test references to their functions, and grepped the doc lines that matched. This was needed to decide coverage per member.
5. Read the enclosing functions of the `Spawn:` literals and `spawnStatusPush` swaps in the loomcli and battencli tests, plus `fabriccli/doc.go:88-96`. This confirmed which struct each literal sets and what the doc line says.

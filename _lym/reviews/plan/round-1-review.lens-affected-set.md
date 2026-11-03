```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each card's change, the set of things that read or assert it, and which cards cover that set.

**Per change: the set found, and the cards covering it**

- **C1, `retryablePushRefusal` gains "missing necessary objects".**
  - Set: `pushretry.go:2` file header; `pushretry.go:42` doc; `PushRebaseFree` doc (`push.go:151`); `gitrepo/doc.go:194`; `fabricengine/doc.go:642` ("Every plain fabric push retries … `(failed)` or `cannot lock ref`"); `archive.go:55` (`archiveWeftTip` doc); the two retry tests.
  - Covered: C1 covers the doc of `retryablePushRefusal`, `PushRebaseFree`, `gitrepo/doc.go:194` and the tests. C6 covers `archive.go:55`.
  - Missed: `pushretry.go:2` and `fabricengine/doc.go:642`.
- **C1, the leased delete wraps `ErrLeaseRejected`.**
  - Set: `deleteRemoteBranch` (destroy.go:1497); its leased callers: Add step 11, `deleteArchivedTaskBranch`, `deleteTaskBranchAtTip` (remove.go:713), `cleanupRemoteWarp` (:126); `gitrepo/doc.go:69` (names only `UpdateRemoteBranchLeased`); `TestDeleteRemoteBranch` (deleteremotebranch_integration_test.go:110,122).
  - Covered: C1, apart from the `doc.go:69` bullet. No caller's text changes.
- **C2, `pushPlainWithRetry` re-signed.**
  - Set: callers at push.go:82, 124 and 155, all on C2. Also the `push.go:1` header, which enumerates the push surface and would not list `PushBranch` or `DropRemoteTracking`. C2 does not list it.
- **C3/C4, `probeWarpLeftover`, `refuseAdoptLocalCheckedOut` and `rollbackAdd` re-signed.**
  - Set: add.go:342, add.go:446–672 (every rollback call), add.go:390.
  - Covered: C3 and C4, inside `Topology.add`.
- **C4, the AdoptLocal rule rewritten.**
  - Set: add.go:50, fabricengine/doc.go:702, fabriccli/doc.go:35, fabriccli/fabric.go:145 (help text, which states no "only" rule) and contracts/orchrole/orch-role.md:13 (still true).
  - Covered: C4.
- **C5, `lockPairTeardown` re-signed, plus the new lock holders.**
  - Set: remove.go:210 (on C5); teardownlock.go header, `TeardownLockPath`, `ErrPairPushRunning` (on C5); `Topology.Remove` doc (remove.go:156: "waiting … for a push of the pair to let go"); `Topology.teardownWait` field doc (topology.go:26: "how long Remove waits"); `archiveTagPushDelays` doc (archive.go:21: "holds at most the pair's own teardown lock, which only that pair's detached pushes wait on"); fabricengine/doc.go:734 and :753.
  - Covered: C5 covers its own symbols and the doc.go paragraph.
  - Missed: remove.go:156, topology.go:26 and archive.go:21.
- **C5, `PushPairAnchored` becomes a no-op while the lock is held exclusive.**
  - Set: loomcli/wiring.go:213 (commit-records `Push`), fabriccli/weft_verbs.go:295 (`lym fabric push`), loomcli/landingdeps.go:328 (`PushTask`, the landing's push of the task pair).
  - Covered: C5 names only commit-records and `lym fabric push`.
  - `PushAnchored` has no production caller. Only pushanchored.go and docs mention it.
- **C6, `archiveWeftTip` and `tagAndPushArchiveTip` re-signed; `archiveTaskTip` too.**
  - Set: remove.go:269, cleanup.go:245, archive.go:92/101, taskbranchdiscard.go:79–105, `ArchiveWeftTipForTest` in archive_integration_test.go (65–195) and remove_local_integration_test.go:195 (`TestOriginWrites_LocalMarkedBranch`).
  - Covered: C6, the whole set.
- **C6, archive order: the branch push and a second tag.**
  - Set: the `Topology.Remove` doc (remove.go:134–137: "Remove archives the pair's weft tip — an archive/&lt;slug&gt;/&lt;tip&gt; tag pushed"); pattern/PATTERN-fabric-git.md "Unwire and teardown" (still true).
  - Covered: C6 does not list `Topology.Remove`'s doc.
- **C6/C7, new `RemoveResult` fields.**
  - Set: fabriccli `removeFields` (fabric.go:1173, hand-built map); battencli/wire.go:373–382 `finishRemoval` (logs `archive_tag` only); pairteardown teardown.go:84 and 432 (passes the result through whole).
  - Covered: C6 and C7 cover `removeFields`. `finishRemoval` reads only the old fields, which is harmless.
- **C7, `removeWeftWorktree` and `deleteWeftBranchOnRemote` re-signed.**
  - Set: remove.go:506 and 524, add.go:771, cleanup.go:258.
  - Covered: C7, the whole set.
- **C8, `CleanupBranchEntry.Skipped`.**
  - Set: fabriccli fabric.go:1011 (JSON entries, carried automatically) and fabric.go:1032–1052 (the terminal summary, which filters on `entry.Error != "" || entry.Deleted`).
  - Covered: C8 does not list the summary.
- **C8, the `cleanupRemoteWarp` reason.**
  - Set: fabric.go:1113 (passes it through).
  - Covered: C8.
- **C9, `EntryOutcome.PairAbsent` and the `Summary` mark.**
  - Set: callers of `originTargets` (originreconcile.go:706, destroy.go:1898).
  - Covered: C9.
- **C13, `pairteardown.Result.Warning`.**
  - Set: fabric.go:1139–1151 `runRemoveWithFlag`, which calls `removeFields(r, res.Session)` with the `RemoveResult` and the `SessionResult`; `res.Warning` sits on neither.
  - Covered: C13 lists `removeFields` with no re-sign arrow and does not list `runRemoveWithFlag`.

**Findings**

1. **MEDIUM**, consistency, card 13 (`internal/fabriccli#removeFields`). The warning never reaches the envelope as the card is written. `removeFields(r fabricengine.RemoveResult, session pairteardown.SessionResult)` takes the removal result and the session result, and `Result.Warning` sits on neither. So adding `warning` needs either a re-signed `removeFields` taking `pairteardown.Result` or the warning field, or an edit at its caller `runRemoveWithFlag`, which no card lists. Evidence: internal/fabriccli/fabric.go:1117, 1146, 1151, 1173.

2. **MEDIUM**, design, card 5 (`PushPairAnchored`). The card covers only part of the consumer set of the new no-op push. It names commit-records (loomcli/wiring.go:213) and `lym fabric push` (fabriccli/weft_verbs.go:295). It misses the landing's `PushTask` in loomcli/landingdeps.go:328, which reads a nil return as the task pair being pushed before the landing goes on. A held exclusive lock would let that caller proceed over an unpushed task branch, and the card does not weigh that. Evidence: internal/loomcli/landingdeps.go:328–331; internal/fabricengine/pushanchored.go:144–213.

3. **LOW**, consistency, cards 5 and 6 (doc text). Doc comments these cards falsify are on no card:
   - `Topology.Remove`'s doc still says Remove only archives a tag, and that its wait is for "a push of the pair" (remove.go:134–137, 156).
   - `Topology.teardownWait`'s field doc says "how long Remove waits" (topology.go:26), but Add and the discard now wait too.
   - `archiveTagPushDelays`' doc says the teardown lock is one "which only that pair's detached pushes wait on" (archive.go:21), but Add and the discard now wait on it. D7's bound names this very interaction.
   - Evidence: internal/fabricengine/remove.go:134, 156; topology.go:26; archive.go:21.

4. **LOW**, consistency, card 1 (retry-reason and lease texts). Card 1 updates `retryablePushRefusal`, `PushRebaseFree` and `gitrepo/doc.go:194`. It misses:
   - the pushretry.go file header (line 2);
   - fabricengine/doc.go:642, which names only `(failed)` and `cannot lock ref`;
   - gitrepo/doc.go:69, which names `ErrLeaseRejected` for `UpdateRemoteBranchLeased` only.
   - Evidence: internal/gitrepo/pushretry.go:2; internal/fabricengine/doc.go:642; internal/gitrepo/doc.go:69.

5. **LOW**, consistency, card 8 (`CleanupBranchEntry.Skipped`). The terminal summary of `lym fabric cleanup` filters entries on `Error != "" || Deleted`. A slug skipped for a held teardown lock has neither, so it does not show on a terminal, although D7 says the sweep "reports it skipped". Card 8 lists only the struct and the JSON entries. Evidence: internal/fabriccli/fabric.go:1032, 1046, 1052. I saw only those three lines, not the whole summary function.

6. **NIT**, consistency, card 2 (`internal/gitrepo/push.go` header). The file header enumerates the push surface ("Push …, PushCoalesced …, PushRebaseFree …, DeleteRemoteBranch …"). If `PushBranch` and `DropRemoteTracking` are added there, the header no longer lists them. Card 2 extends only doc.go. Evidence: internal/gitrepo/push.go:1.

Tool calls:
1. Grep for the callers of `pushPlainWithRetry`, `archiveWeftTip`, `removeWeftWorktree`, `deleteWeftBranchOnRemote`, `lockPairTeardown`, `refuseAdoptLocalCheckedOut`, `ArchiveWeftTipForTest` and the archive helpers, plus the AdoptLocal rule text. The manifest held only definitions, not the full caller list.
2. Read the test enclosing remove_local_integration_test.go:195, and grepped for consumers of `CleanupBranchEntry` and `RemoveResult` and for teardown-lock prose. These consumers were not in context.
3. Read fabric.go:1135–1200 for `removeFields` and the remove call site, and the cleanup envelope. This was needed to judge card 13's warning wiring.
4. Read the three synchronous `PushPairAnchored` call sites (loomcli landingdeps and wiring, fabriccli weft_verbs) and the name of the remove RunE. Only their locations were known.
5. Read the gitrepo doc lease and retry bullets, the add help text, the retry-reason sites and the lease tests. These were needed to find the doc and test members of card 1's set.
6. Grepped for the cleanup sweep's callers and the code reading `entry.Deleted`. This was needed to find the consumers of card 8's Skipped field.

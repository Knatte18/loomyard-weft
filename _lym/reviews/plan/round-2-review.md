---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lym/plan/06-remove-archive-order.md:60
    summary: "card 6 has the external archive tests read an unexported weftArchive field through a re-export var, which cannot compile"
    fix: |-
      In card 6, change the Edit entry `internal/fabricengine#ArchiveWeftTipForTest` to `internal/fabricengine#ArchiveWeftTipForTest` -> `func ArchiveWeftTipForTest(rec *Mutations, l *lymcwd.Location, slug, weftBranch string) (tag, skippedReason string, err error)`.
      In the Callers paragraph replace the sentences that say it keeps re-exporting `archiveWeftTip` and that `TestArchiveWeftTip`, `TestArchiveWeftTip_NoOriginSkips`, `TestArchiveWeftTip_UnreachableOriginErrors` and `TestOriginWrites_LocalMarkedBranch` call it with `pushBranch` false and a nil sleep and read `weftArchive.localTag`.
      Say instead that `ArchiveWeftTipForTest` becomes a wrapper in `export_test.go` that calls `archiveWeftTip(rec, l, slug, weftBranch, false, nil)` and returns `weftArchive.localTag`, the skipped reason and the error, so the four tests keep their call shape and assertions.
      Keep the four test glyphs in Edit.
      `lym loom validate-plan --overlay` with this change passes.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/plan/13-remove-follow-up.md:46
    summary: "D3's sites are enumerated by hand, and the same fabric way forward is still quoted by the cancel refusal and the landing stop, uncovered and unrecorded"
    fix: |-
      Add `internal/battencli#judgeOrigin` and `internal/landingshed#stoppedReason` to card 13's Edit list and `internal/landingshed/doc.go` to its Extends list (`lym loom validate-plan --overlay` passes with these three; without the doc.go line it reports doc-go-unlisted).
      In `judgeOrigin`, the stopped refusal built by `stoppedRefusal` for act cancel, when its text names `lym fabric remove`, ends with the `pairteardown.BattenFollowUp` text computed from `deps.ReadStatus` and the owner-pause read, in place of the trailing `then run <rerun> again`: before Land that names `lym batten cancel <slug> --remove`, at Land it names D2's Land way forward and never a cancel.
      In `stoppedReason` (landingshed `reportVerdict`), when the quoted way forward names `lym fabric remove`, append the same follow-up in place of `then run "lym loom resume" in the task worktree`, which names a worktree the removal deletes; the follow-up reaches the stop as a value its caller passes in, so landingshed learns nothing of batten's files.
      Add one row for each site to the existing test of that refusal.
      In Shared Decision `d3-sites-outside-fabric`, list these two sites as carried, and state for `internal/adoptcli/adopt.go`, `internal/cleanupcli/reconcile.go` and `internal/hibernateengine/wake.go` either that they print a reconcile report line rather than a stop or refusal, or carry them too; a drop of any D3 site needs an `Added after Discussion` entry through the parent.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lym/plan/08-cleanup-teardown-lock.md:26
    summary: "card 8 prescribes a live-pair re-read in Cleanup under the slug lock with no named input or seam that reaches it"
    fix: |-
      In card 8, add `plan:internal/fabricengine#SetCleanupAfterEnumerateForTest` -> `func SetCleanupAfterEnumerateForTest(t *Topology, hook func())` under a new `**Create:**` and `internal/fabricengine#Topology` under Edit (`lym loom validate-plan --overlay` passes).
      `Topology` gains `cleanupAfterEnumerate func()`, nil in production, which `Cleanup` runs once per orphan right before it takes the slug's lock, as `cleanupRemoteWarp` runs its `afterEnumerate`.
      State that an orphan found live under the lock gets no entry, as a live pair found at enumeration gets none.
      Add a row to `TestCleanup_ApplyArchivesEachOrphanBeforeDeleting`: the hook makes a worktree on the orphan's paired warp branch, then `Cleanup(apply, remote)` archives nothing for the slug, pushes no archive tag, and leaves the weft branch locally and on origin.
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/plan/10-batten-resume-readopt.md:76
    summary: "the planned TestStartFirstPass assertion reads a hand-populated receiver, so it cannot show that StartFirstPass wires the readopt seams"
    fix: |-
      Wire the seams in `armAt`, which `Command`'s run verb and `StartFirstPass` both call with verb run: when verb is `run` and `c.readopt.Add` is nil, set `c.readopt = hubReadoptSeams(prime)` beside the `hubOwnerSeams` and `hubStartGuard` defaults.
      In card 10 replace the Edit entries `internal/battencli#Command`, `internal/battencli#StartFirstPass` and `internal/battencli#TestStartFirstPass` with `internal/battencli#battenCLI.armAt` (`lym loom validate-plan --overlay` passes), drop the sentences saying `Command` and `StartFirstPass` set the field, and drop the `TestStartFirstPass` bullet.
      The step verb and every hand-built receiver then keep a nil `readopt` as the card requires.
  - id: F5
    severity: NIT
    class: consistency
    location: _lym/plan/10-batten-resume-readopt.md:73
    summary: "the tier0 row for taskWorktreeGone says no path reads gone, which contradicts the function reading an absent path as gone"
    fix: |-
      In card 10's bullet on `TestTaskWorktree_AbsentPairIsAnAnswerAndANamedRefusal`, replace `no path reads gone` with `an absent path reads gone (true) with a nil error, by a stat alone`.
  - id: F6
    severity: LOW
    class: design
    location: _lym/plan/08-cleanup-teardown-lock.md:40
    summary: "cards list the docs of the symbols they change but not the comments the change falsifies, the gap round 1 filed as F13, now in cards 8 and 10"
    fix: |-
      Method: before handoff, grep the repository for each statement about a behavior a card changes, and list every comment, file header and background file that states the old behavior, instead of listing only the changed symbols' own docs.
      Applied to the two cards that still miss them:
      Card 8's Intent also rewrites the `Topology.Cleanup` doc comment (each orphan tip is archived before its branch is deleted under apply, which a held lock or a live pair now skips), the `cleanup.go` file header's `apply == true` line of the flag matrix, and the `CleanupRemoteWarp` doc comment (with apply each candidate goes through the destructive gate); list `internal/fabricengine#Topology.CleanupRemoteWarp` under Edit (`lym loom validate-plan --overlay` passes), `Topology.Cleanup` is already listed.
      Card 10's Intent also adds the gone-pair case to the `ownerPreflight` doc comment (`internal/battencli/owner.go`, the paragraph beginning `ownerPreflight is the owner check`): a found status at another row whose task worktree is gone, on a receiver that carries `readopt`, runs `readoptGonePair` in place of the resume reconcile; `internal/battencli#battenCLI.ownerPreflight` is already in Edit.
  - id: F8
    severity: LOW
    class: design
    location: _lym/plan/13-remove-follow-up.md:55
    summary: "card 13 asserts the follow-up wording in three tests although TestBattenFollowUp, the test of the exported builder, covers every branch of it"
    fix: |-
      Keep the wording table in `TestBattenFollowUp`.
      Reduce the `TestTeardown_CallSequence` rows to wiring: a record at Run-Shed gives a `Result.Warning` equal to the `the batten run of <slug> still stands; ` prefix plus `BattenFollowUp`'s text for that record, and the owner-held and `LeaveClaim` rows give an empty warning; drop the Land row.
      Reduce `TestPreflightRunOwner_RemoveFollowUp` to two rows: a stopped verdict quoting `lym fabric remove` has its trailing re-run replaced by the follow-up, and one naming no `lym fabric remove` is returned unchanged; drop the separate Land row.
  - id: F9
    severity: LOW
    class: design
    location: _lym/plan/10-batten-resume-readopt.md:79
    summary: "two tier1 rows of card 10 re-test pair re-creation that card 4's TestAdd_AdoptOnly already covers at the Add surface"
    fix: |-
      In card 10's `TestBattenIntegration_GoneTaskWorktree`, drop the rows `The task worktree alone deleted, the records worktree, weft and task branches standing` and `A beam project's pair at a subpath anchor, its task worktree gone`.
      `TestAdd_AdoptOnly` covers them at `Add` with its rows for a standing records worktree and for a beam project's pair; the pre-run adds only the status, the second reconcile and the push, which the remaining first and fourth rows assert.
  - id: F10
    severity: LOW
    class: consistency
    location: _lym/plan/04-add-adopt-only.md:42
    summary: "card 4 runs the warp probe for an adopted local branch but never says it widens card 3's warp drop condition, which excludes that case"
    fix: |-
      In card 4's bullet on the warp probe, add: under `AdoptOnly` the warp-side drop of card 3 in `Topology.add` also covers an adopted local warp branch whose probe answered `onOrigin` false, so `refs/remotes/origin/<warp>` and its upstream config are dropped, and the probe's adopt tip stays unset so nothing is fetched.
  - id: F11
    severity: LOW
    class: consistency
    location: _lym/plan/05-teardown-lock-add-discard-push.md:49
    summary: "card 5 makes TestTeardownLock_SerialisesAddDiscardAndPairPush serial because of SetTeardownWaitForTest, which sets a per-Topology field"
    fix: |-
      In card 5's test paragraph replace `not parallel where it shortens the wait through SetTeardownWaitForTest` with `parallel: SetTeardownWaitForTest sets the wait on the one Topology, and each row has its own hub, so no process-global state is touched`.
  - id: F12
    severity: LOW
    class: scope
    location: _lym/plan/05-teardown-lock-add-discard-push.md:42
    summary: "card 5 makes lym fabric push return push_skipped but names no test that reaches that branch"
    fix: |-
      Add to card 5's tests one row in the existing tier1 test that drives `lym fabric push` against a real pair: the test holds the slug's teardown lock exclusive with `lock.AcquireWriteLock(TeardownLockPath(...))`, runs the verb, and asserts an ok envelope with `push_skipped` set and origin's weft tip unchanged.
---

### [MEDIUM] Card 6's archive tests read an unexported field from an external package

**Location:** `_lym/plan/06-remove-archive-order.md:60`
**Issue:** `ArchiveWeftTipForTest` is `var ArchiveWeftTipForTest = archiveWeftTip` in `internal/fabricengine/export_test.go`, and its callers (`archive_integration_test.go`, `remove_local_integration_test.go`) are in package `fabricengine_test`.
After card 6 the var carries the new signature and returns `weftArchive`, whose fields (`localTag` and the rest) are unexported, so a caller outside the package cannot read the tag as the card says it does.
The Edit entry also changes the symbol's type without the re-sign arrow.

### [MEDIUM] D3's follow-up misses the consumers that quote the same fabric text

**Location:** `_lym/plan/13-remove-follow-up.md:46`
**Issue:** D3 binds every stop or refusal that names `lym fabric remove` for a slug with a batten run record, and exempts only Finalize's adopter-first stop.
`FailedPushWayForward`'s `PushFailureGone` text names `lym fabric remove` and reaches battencli through `BranchOutcome.WayForward`.
Card 13 appends the follow-up in `preflightRunOwner` only, so `judgeOrigin` (`internal/battencli/cancel.go:536`) still ends the cancel refusal with `then run <rerun> again`: at a Land record that sends the operator to a cancel D2 calls a release of a landed entry.
`stoppedReason` (`internal/landingshed/stuck.go:112`) quotes the text and ends with `lym loom resume` in a task worktree the removal deletes.
`internal/adoptcli/adopt.go`, `internal/cleanupcli/reconcile.go` and `internal/hibernateengine/wake.go` print the same text in reports.
The Shared Decision `d3-sites-outside-fabric` names three sites as the set, derived by hand from the text, and the decision record carries no `Added after Discussion` entry for the rest, so the uncarried part of D3 stays a finding.
Round 1's F6 found the resume path only; the cancel path reaches the same `stoppedRefusal`, which is why the finding is about the enumeration method and not one more missing site.

### [MEDIUM] Cleanup's live-pair re-read has nothing that reaches it

**Location:** `_lym/plan/08-cleanup-teardown-lock.md:26`
**Issue:** The card prescribes that `Cleanup` re-reads warp worktrees once the slug's lock is held and skips the slug as live.
`cleanupRemoteWarp` has an `afterEnumerate` hook that lets the card's fourth test row make a worktree between enumeration and lock; `Cleanup` has no such hook and no row, so the guard branch is never reached by a named test.
The branch matters: an archive tag at a live pair's tip makes `Add` on another machine read the pair as not live.
The card also leaves the skipped-live entry shape unstated.

### [LOW] TestStartFirstPass cannot observe StartFirstPass

**Location:** `_lym/plan/10-batten-resume-readopt.md:76`
**Issue:** `TestStartFirstPass` (`internal/battencli/cli_test.go:176`) builds each receiver by hand and calls `c.startFirstPass(cmd)`; the receiver `StartFirstPass` (`internal/battencli/cli.go:634`) builds is never visible to it.
The assertion that the receiver carries `readopt` therefore checks the test's own fixture, and no test pins that adopt's green start and `lym beam create` take the re-adopt path.
`armAt` already holds the lazy per-verb defaults for `owner` and `guard`, and both entry points call it.

### [NIT] The taskWorktreeGone tier0 row is worded backwards

**Location:** `_lym/plan/10-batten-resume-readopt.md:73`
**Issue:** The card defines `taskWorktreeGone` as true when nothing stands at the path, but the row reads `no path reads gone`, which an implementer can take as asserting false.
The row is tier0 and spawns nothing, since an absent path is a stat alone.

### [LOW] Cards 8 and 10 leave comments their change falsifies

**Location:** `_lym/plan/08-cleanup-teardown-lock.md:40`
**Issue:** Round 1's F13 found cards 1, 2, 6 and 8 rewriting the docs of the symbols they change but not the comments and headers that state the old behavior, and the fixes were made item by item.
The gap recurs.
Card 8 lists only the `internal/fabricengine/doc.go` cleanup paragraph, while the `Topology.Cleanup` doc comment (`internal/fabricengine/cleanup.go`) says that under apply each orphan tip is archived before its branch is deleted, the header's flag matrix says apply deletes every orphan that is not primary, checked out or unmanaged, and the `CleanupRemoteWarp` comment says each candidate goes through the destructive gate under apply; a held lock or a live pair now skips all three.
Card 10 (`_lym/plan/10-batten-resume-readopt.md:58`) rewrites the `taskWorktreeLocation` and `presentTaskWorktreePath` comments but not the `ownerPreflight` doc comment (`internal/battencli/owner.go:229`), which enumerates its cases and ends with `otherwise it is the resume reconcile`.

### [LOW] Card 13 asserts the same wording in three tests

**Location:** `_lym/plan/13-remove-follow-up.md:55`
**Issue:** `TestBattenFollowUp` covers every branch of the exported `BattenFollowUp`: Run-Shed and Worktree-Create name the cancel, Land names `lym batten run` and no cancel, and no record, `done`, Worktree-Teardown and owner-held give the empty string.
The `TestTeardown_CallSequence` rows and `TestPreflightRunOwner_RemoveFollowUp` assert those same Run-Shed and Land texts again.

### [LOW] Card 10's integration rows repeat Add-level behavior

**Location:** `_lym/plan/10-batten-resume-readopt.md:79`
**Issue:** The rows `The task worktree alone deleted` and `A beam project's pair at a subpath anchor` assert that `Add` re-makes a pair around a standing records worktree and at the anchor its weft records.
`TestAdd_AdoptOnly` in card 4 has a row for each at the `Add` surface, and the pre-run changes neither behavior.

### [LOW] Card 4 does not widen card 3's warp drop condition

**Location:** `_lym/plan/04-add-adopt-only.md:42`
**Issue:** Card 3 drops a warp tracking ref only for a warp branch forked with `onOrigin` false and says an adopted local branch is not dropped there because card 4 runs the probe.
Card 4 says the probe's answer serves card 3's drop but never states that the condition now includes an adopted local branch, which D8 and card 4's own last test row require.

### [LOW] Card 5 serialises a test that needs no serialising

**Location:** `_lym/plan/05-teardown-lock-add-discard-push.md:49`
**Issue:** `SetTeardownWaitForTest` (`internal/fabricengine/export_test.go`) sets `Topology.teardownWait` on the one instance, and each row has its own hub, so `PATTERN-test-speed` ("a test calls `t.Parallel` unless it touches process-global state") leaves no reason for the test to be serial.
The card names no global state.

### [LOW] The lym fabric push skip has no test

**Location:** `_lym/plan/05-teardown-lock-add-discard-push.md:42`
**Issue:** Card 5 makes `addWeftVerbs` return an ok envelope with `push_skipped` when `PushPairAnchored` returns `ErrPairPushRunning`.
Its only test row is in `TestCommitRecordsVerb_Success`, which drives `commitRecordsVerb`; the verb in `internal/fabriccli/weft_verbs.go` calls `PushPairAnchored` directly with no seam, so only a real hub with the lock held reaches the branch.

## Rejected

- Focus: card 10's tier0 rows.
  Every `TestOwnerPreflight_GoneTaskWorktree`, `TestBattenPreRun_ContinuePass` and `TestTaskWorktree_AbsentPairIsAnAnswerAndANamedRefusal` row uses fake seams or a stat of an absent path.
  The rows that reach `lymcwd.ResolveWorktree` (a plain file and an empty directory at the task path) are in the tier1 `TestBattenIntegration_GoneTaskWorktree`.
  Not a defect.
- Focus: card 4's closed-set refusals for a task branch only on origin.
  The card widens the `refuseAdoptLocalCheckedOut` call to run whenever `AdoptLocal` is set and the weft branch exists locally, and it has a row for that case, so `*ErrAdoptLocalCheckedOut` with `Records` true is reachable.
  Not a defect.
- Focus: card 4's reused records worktree.
  The card advances the branch with `git merge --ff-only` inside the reused worktree instead of the fetch into `refs/heads/<weft>` at `internal/fabricengine/add.go:469`, which git refuses for a checked-out branch, and a branch checked out in another records worktree is refused earlier.
  Not a defect.
- Focus: card 6's ancestry comparisons.
  The origin tip is fetched by SHA right after it is read and before every comparison, so each `isAncestorOrEqual` runs between commits both present, in the order the card states.
  A tip another clone moved between the read and the push leaves a rejected branch push that is only logged, and the lease on the earlier tip then keeps the branch.
  Not a defect.
- Focus: card 8's range of re-reads.
  The origin tip and local tip are read after the lock, the sweep's lease covers a tip that moved, and `cleanupRemoteWarp` re-reads checked-out worktrees under the lock.
  The only unreached piece is F3.
- Focus: card 1's retry predicate.
  `retryablePushRefusal` has one production caller, `PushWithRetry`, which passes no extra reasons, so `PushRebaseFree`, `Push` and `PushCoalesced` classify as today.
  Not a defect.
- Focus: card 5's skipped push.
  `PushPairAnchored` returns an error wrapping `ErrPairPushRunning` and never a nil, so no caller reads the skip as a landed push; `commit-records` and `lym fabric push` report it as skipped and the landing's `PushTask` fails.
  F12 is the only gap.
- Focus: removal warning path.
  `Result.Warning` reaches `removeFields` through `runRemoveWithFlag`, which is typed.
- `PATTERN-fabric-isolation` against `AdoptOnly`, `PairMayBeAbsent` and `PruneRegistrations`.
  Each is a general mode or verb; "only batten's resume sets it" documents current use and not the shape.
- Dropping `refs/remotes/origin/<branch>` from `add.go` against the destruction chokepoint.
  The chokepoint list in `internal/fabricengine/doc.go` names file and branch removals, remote branch deletes and resets, and the guard's token list has no entry for a tracking ref.
- `PushAnchored`, the records-only sibling of `PushPairAnchored`, is not gated by card 5.
  It has no production caller, so carrying it would add code no run reaches.
- `PATTERN-friction-capture` against card 12's failed stop.
  Friction notes are for loom halts and webster refusals, and batten carries no friction directory.
- Round 1's deferral: a reused records worktree whose uncommitted records block `git merge --ff-only`.
  Agreed: D2 treats every answer outside the closed set as transient, retried with dobby's failing-pass notice, as it does a failed `git worktree add`, so the case is bounded and visible; a typed refusal would change the decision record.
- Round 1's fixes F1 to F13 were re-read against the current cards and none regressed.

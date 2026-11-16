---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/05-rejected-push-takeover-read.md
    summary: Card 5 says pushClassified returns a takeover unrecorded, but pushClassified records any error classifyFailure does not name as a failed push of class other, and pushClassified is not a card 5 target
    fix: In card 5 add `internal/fabricengine#pushClassified` under **Edit:**. In its Intent replace the sentence "`rejectionRetryKind` names both as non-retryable, so `settleRejection` returns them at once and `pushClassified` returns them unrecorded" with "`rejectionRetryKind` names both as non-retryable, so `settleRejection` returns them at once; `pushClassified` returns a `*TakeoverError` or an error wrapping `ErrHandedOver` from `settleRejection` as it is, before `classifyFailure` and `fail`, so no failed-push record or notice is written, and `pushSideDetached` returns it rather than logging it as a recorded failure". Validated with `lym loom validate-plan --overlay`.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-pair-push-records-first.md
    summary: A deferred or ended records side returns nil from PushPairAnchored, so card 8's seam reads it as landed and starts the next producer with the transition not on origin, against D1
    fix: "In card 3's Intent, after \"A records side that fails, is rejected, is deferred or did not land sends no code side.\", add: \"A records side that returned nil without landing is returned as an error: a deferral as the `*PushFailedError` carrying the `deferred` record `pushJob.deferral` wrote, and an ended branch as an error naming the ended entry.\" Change card 3's deferred-records test row to assert that error (class `deferred`) in place of a nil return. In card 8's Intent, make `IsPushOwed` count a `*PushFailedError` of class `deferred` as owed, and add a `deferred` row to `TestIsPushOwed`. In card 7's Intent, make `FailedPushesOwedBy` count class `deferred` too, and add a row for it to `TestFailedPushRecord`. No target changes."
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/07-read-run-owed-and-tip.md
    summary: Cards 6, 7 and 11 move branches backwards outside fabricengine's destroy.go, against PATTERN-fabric-boundary's destruction chokepoint, and name no home or executor checks for those steps
    fix: "The steps are: card 7's `resetUnnamedTip` and card 6's `pushOwedCodeTip` resetting origin's code branch through `Repo.UpdateRemoteBranchLeased`, which can move the branch backwards and today runs only in `internal/fabricengine/destroy.go:1640`; and the `git reset --keep` of the local parent branch in card 6's `archiveParentMerge` and card 11's `ArchiveParentMerge`. In each card's Intent, state that the step runs through an executor in `internal/fabricengine/destroy.go` that checks containment, ownership, dirtiness and force in that order. Ownership is the lease on the origin tip it read for the origin reset, and the archive tag already pushed for the parent reset. The new private members keep their glyphs, so no target changes."
  - id: F4
    severity: BLOCKING
    class: scope
    origin: lens:repo-invariants
    location: _lym/plan/08-transition-sync-push.md
    summary: Card 8 removes the Spawn field from statuscommit.Deps and the seams, but five tests that set Spawn are not its targets, so the card's own gate cannot compile them
    fix: "In card 8 add under **Edit:** `internal/battencli#TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed`, `internal/battencli#TestNewCommitStatusSeam_BoardWriteFailureDoesNotStopTheCommit`, `internal/battencli#TestBattenRecordsCommitSeams`, `internal/battencli#newFakeReceiver` and `internal/loomcli#TestNewCommitStatusSeam_Finish`. In its Tests list, say that each replaces `Spawn` with `Push` (`commitstatus_test.go:118,153`, `driverdeps_test.go:226`, `run_test.go:74`, `wiring_commitstatus_test.go:192`). Validated with `lym loom validate-plan --overlay`."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/03-pair-push-records-first.md
    summary: Card 3 returns the teardown wait-out as an error wrapping ErrPairPushRunning, but card 8's IsPushOwed owes only a PushFailedError of class teardown_lock, so a literal reading of card 3 halts the run instead of owing the push
    fix: "In card 3's Intent replace \"and returns the error wrapping `ErrPairPushRunning`, so the push is owed (D1)\" with \"and returns `&PushFailedError{Record: <that teardown_lock record>, Err: <an error wrapping ErrPairPushRunning>}`, so `IsPushOwed` reads it owed (D1)\". In its `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` held-past-wait row, assert both `errors.As` to a `*PushFailedError` of class `teardown_lock` and `errors.Is(err, ErrPairPushRunning)`."
  - id: F6
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/07-read-run-owed-and-tip.md
    summary: "Card 7's owner guard resets origin's code tip when the last transition does not name it, but D2 says a tip that no transition of the run on origin names, and the decision record holds no Added after Discussion entry for the change"
    fix: "Ask the parent with `lym creel post --parent --expect-reply` to record the narrowing with `lym loom decision add`. The narrowing: the owner's guard resets origin's code branch when its tip is neither the tip the run's last transition names nor an ancestor of it, because a given-up tip stays named by an older transition. Then cite that entry in card 7's Intent paragraph that opens \"The owner's guard (D2)\". Until it is recorded, card 7 follows D2's text."
  - id: F7
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/07-read-run-owed-and-tip.md
    summary: "Card 7's resetUnnamedTip force-moves origin's task code branch, which can drop any commit on it that the last transition does not name, such as an operator's direct push, and its Intent does not state what it lets through or what bounds it"
    fix: "Add to card 7's Intent, after the sentence ending \"resets origin's code branch with a lease at origin's tip to that named tip and pushes again\": \"This drops from origin's task code branch every commit past the named tip, whoever pushed it. It is bounded in three ways: it runs only on a task pair's rejected code-side push, which records-first runs only after the run's own records side landed, so a machine a takeover moved never reaches it; it is leased at the origin tip it read; and it never runs for the prime's code branch.\""
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/06-run-hand-over.md
    summary: "Card 6 reads origin's batten record for the dropped intents through OriginRunStatus, but that function reads the child run record from the pair's records branch (originreads.go:27-31), not the batten record on the prime's records branch"
    fix: "In card 6 add under **Create:** `plan:internal/fabricengine#OriginBattenStatus` -> `func OriginBattenStatus(prime *lymcwd.Location, slug string) (status shedengine.Status, found bool, err error)`, which reads slug's batten status file at the prime records branch's fetched upstream tip. Remove `internal/fabricengine#OriginRunStatus` from card 6's **Uses:**. In the Intent's `droppedIntents` step, replace \"(`OriginRunStatus` over the prime's records branch)\" with \"(`OriginBattenStatus`)\". Validated with `lym loom validate-plan --overlay`."
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/09-intent-sync-push.md
    summary: Card 9 says endRunPhase stops end-only with nothing written today, but cancel.go:370-379 already writes and commits the cancel intent before EndForCancel on the end-only path, so the planned change and its TestCancelRun rows change nothing
    fix: "In card 9 remove `internal/battencli#endRunPhase` from **Edit:**. Delete the Intent's sentences from \"`endRunPhase` now writes the cancel intent\" through \"which pushes the intent and finishes the cancel (D1).\" Replace them with: \"The end-only path already writes and commits the intent before `EndForCancel` (`cancel.go:370-379`); with card 8, `commitCancelRecord` owes that commit's push, and the output names the re-run of `lym batten cancel <slug>` as today.\" In the Tests list, drop the clause about the unreachable-origin rows now expecting `WriteIntent`, `CommitRecord` and `EndForCancel`, keeping the `SpawnPush` to `PushRecords` rename and the owed `PushRecords` row. Validated with `lym loom validate-plan --overlay`."
  - id: F10
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/01-runfsm-push-owed.md
    summary: Card 12 says runfsm's code_tip refusal names lym fabric push in the owing machine's task worktree, but card 1 leaves that way forward as a bare re-run, so with the owing machine gone and a single transition a cancel is refused with no reachable way forward
    fix: "In card 1's `code_tip` bullet, replace \"way forward unchanged (`run <retry> again once the code push lands`)\" with \"way forward: run lym fabric push in the task worktree on machine <CodeTipMachine>, then run <retry> again once the code push lands; an empty machine leaves out the push clause\". Add the matching row to `TestAct`. In card 7's ImpactSummary, state that `lym batten cancel` waits on a `code_tip` verdict until the owing machine pushes or card 12 gives the tip up."
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/12-code-tip-grace.md
    summary: Card 12 gives up a missing tip after the grace even when this machine holds the tip in its own task worktree, discarding a transition whose code it could push itself
    fix: "In card 12's `settleCodeTip` branches, before the \"still missing past the grace\" bullet, add one: when the local code branch holds the missing tip, push it through `PushRun` over the pair and give nothing up. A failed push is logged and the next tick retries. Only a tip this repository lacks is given up after `runfsm.CodeTipGrace`. Add the matching row to `TestSettleCodeTip`."
  - id: F12
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/00-overview.md
    summary: pairteardown's Teardown.removeCancelledRunRecord still spawns a detached SpawnStatusPush of the batten run record (followup.go:107), and no card moves it or states why it stays detached
    fix: "In the overview's `Decision: out-of-part` bullet, after the sentence about Pawl's `SpawnPush`, add: \"`Teardown.removeCancelledRunRecord`'s `SpawnStatusPush` (`internal/pairteardown/followup.go:107`) removes the record of a run whose cancel finished, so it carries no transition a run acts on, and it stays detached.\""
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/11-finalize-push-order.md
    summary: Card 11 makes markTaskDone return its error in place of logging it, but its Edit glyph carries no re-sign arrow
    fix: "In card 11's **Edit:** replace `internal/landingshed#Finalize.markTaskDone` with `internal/landingshed#Finalize.markTaskDone` -> `func (fz *Finalize) markTaskDone() error`. Validated with `lym loom validate-plan --overlay`."
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/03-pair-push-records-first.md
    summary: Card 4 retires the failed-remove respawn, but internal/fabriccli/doc.go:93 still says the removal's push children it respawns keep their own gates, and no card edits that doc
    fix: "Move `internal/fabriccli/doc.go` in card 3 from **Uses:** to **Edit:**. Add to card 3's Intent that the fabriccli doc's removal paragraph drops \"the push children it respawns keep their own gates\", which card 4 retires. Adding it to card 4 instead is refused by `uses-later-target`, since card 3 uses the file. Validated with `lym loom validate-plan --overlay`."
  - id: F15
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/08-transition-sync-push.md
    summary: Card 8 owes ErrPushLockBusy, which writes no failed-push record, though it excludes a bare ErrPairPushRunning for exactly that reason, so a retry on a busy push lock is invisible to ReadRun and lym beam status
    fix: "In card 8's Intent, after the sentence on bare `ErrPairPushRunning`, add: \"`ErrPushLockBusy` writes no record either. Its retry is owed only inside the seam, which holds the next producer itself, so `ReadRun` and `lym beam status` show no owed push while a holder keeps the push lock, and the holder lands the flagged sides on release.\""
  - id: F16
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/05-rejected-push-takeover-read.md
    summary: "Card 5 has lym fabric push build MachineNames from machinename.Names, a second construction site beside pairteardown.ThisMachine, which the overview's names-from-the-caller decision names as the one place the registry part adds names"
    fix: "In card 6 add `internal/fabriccli#addWeftVerbs` under **Edit:** and `internal/fabriccli/doc.go` under **Uses:**. State in card 6's Intent that `addWeftVerbs` builds its `MachineNames` through `pairteardown.ThisMachine` in both branches, replacing card 5's `machinename.Names` build (fabriccli already imports pairteardown). Validated with `lym loom validate-plan --overlay`."
---

### [BLOCKING] Card 5 claims a takeover returns unrecorded, but pushClassified records it

**Location:** `_lym/plan/05-rejected-push-takeover-read.md`, Intent paragraph on `rejectionRetryKind`

**Issue:** In `internal/fabricengine/pushclassified.go:133-145`, `settleRejection` hands a non-retryable error back to `pushClassified`. `classifyFailure` matches neither `*TakeoverError` nor `ErrHandedOver`, so line 145 calls `job.fail(PushFailureOther, …)`. That writes a failed-push record and its creel notice. `pushSideDetached` (`coalesce.go:64-66`) then swallows the resulting `*PushFailedError`, so the detached child never sees the takeover. Card 5's own row "no failed-push record stands" cannot pass. The card lists no `pushClassified` target, so the implementer has no licence to fix it. The guard-reach lens and the handler found this separately.

### [BLOCKING] A deferred or ended records side reads as landed to the transition seam

**Location:** `_lym/plan/03-pair-push-records-first.md` and `_lym/plan/08-transition-sync-push.md`

**Issue:** `pushClassified` returns nil for a deferral (`pushclassified.go:134-136`) and for an ended branch (`:423-424`). Card 3 withholds the code side in that case but still returns nil; its own deferred row asserts "the call returns nil". Card 8 maps nil to "landed … and the next producer starts". The `deferred` class is not owed in cards 7 and 8 either. So a transition whose records never reached origin lets the run act, against D1's "the next producer starts only once every side has landed".

### [BLOCKING] Branch rewinds outside the destruction chokepoint

**Location:** `_lym/plan/07-read-run-owed-and-tip.md`, `_lym/plan/06-run-hand-over.md`, `_lym/plan/11-finalize-push-order.md`

**Issue:** `PATTERN-fabric-boundary` makes `internal/fabricengine/destroy.go` the only file allowed a destructive primitive, with executors that check containment, ownership, dirtiness and force in order. `gitrepo.Repo.UpdateRemoteBranchLeased` can move a branch backwards (`internal/gitrepo/push.go:339-347`), and its one use today is `destroy.go:1640`. Card 7's `resetUnnamedTip` and card 6's `pushOwedCodeTip` use it to rewind origin's code branch. Cards 6 and 11 `git reset --keep` the local parent branch backwards. None of these cards names `destroy.go` or the executor checks.

### [BLOCKING] Card 8 misses five tests that set the Spawn field it removes

**Location:** `_lym/plan/08-transition-sync-push.md`

**Issue:** These tests set `Spawn:` on the deps structs whose field card 8 removes, and none is among its targets:

- `internal/battencli/commitstatus_test.go:118` and `:153`, in `TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed` and `TestNewCommitStatusSeam_BoardWriteFailureDoesNotStopTheCommit`;
- `internal/battencli/driverdeps_test.go:226`, in `TestBattenRecordsCommitSeams`;
- `internal/battencli/run_test.go:74`, in `newFakeReceiver`;
- `internal/loomcli/wiring_commitstatus_test.go:192`, in `TestNewCommitStatusSeam_Finish`.

The card's own tier0 gate cannot compile those packages. The repo-invariants and affected-set lenses both found this; I confirmed each line.

### [MEDIUM] The teardown wait-out error type disagrees between cards 3 and 8

**Location:** `_lym/plan/03-pair-push-records-first.md`

**Issue:** Card 3 returns "the error wrapping `ErrPairPushRunning`". Today's refusal at `teardownlock.go:119` is a plain `fmt.Errorf`. Card 8's `IsPushOwed` owes only a `*PushFailedError` of class `teardown_lock` and treats a bare `ErrPairPushRunning` as not owed. Followed literally, the cards halt the run where D1's criterion has the push turn owed.

### [MEDIUM] Card 7 narrows D2's owner guard without a recorded decision

**Location:** `_lym/plan/07-read-run-owed-and-tip.md`

**Issue:** D2 says the owner's code-side push is "rejected over a tip that no transition of the run on origin names". Card 7 resets on the last transition alone ("History is not searched"). Its reasoning holds: a given-up tip stays named by an older commit, so D2's wording would never reset it. It is still a departure from the answer key with no `Added after Discussion` entry.

### [MEDIUM] Card 7's origin reset states no bound

**Location:** `_lym/plan/07-read-run-owed-and-tip.md`

**Issue:** `resetUnnamedTip` force-moves origin's task code branch whenever its tip is neither the last transition's tip nor an ancestor of it. That includes a commit an operator or another clone pushed directly. The Intent states when it fires, but not what it can drop or what keeps it to the run's owner.

### [MEDIUM] Card 6 reads origin's batten record through a child-record reader

**Location:** `_lym/plan/06-run-hand-over.md`, the `droppedIntents` step

**Issue:** `OriginRunStatus` (`internal/fabricengine/originreads.go:27-52`) reads "the child run record of slug" from the pair's records branch through `pairBranchNames`. The pause and cancel intents sit in the batten record on the prime's records branch, so the comparison would never see origin's intents.

### [MEDIUM] Card 9's end-only cancel change targets a state that does not occur

**Location:** `_lym/plan/09-intent-sync-push.md`

**Issue:** `endRunPhase` already calls `WriteIntent` and `CommitRecord` before `EndForCancel` whatever `judgeOrigin` answers (`internal/battencli/cancel.go:313-316`, `:370-379`, `:412-417`). The card's premise, "today `endRunPhase` then stops with nothing written", is false. Its new `TestCancelRun` expectations already hold, so they are never seen failing.

### [MEDIUM] The code-tip way forward disagrees between cards 1 and 12, and can be unreachable

**Location:** `_lym/plan/01-runfsm-push-owed.md`

**Issue:** Card 12 says `runfsm`'s `code_tip` refusal names `lym fabric push` in the owing machine's task worktree, while card 1 keeps "run <retry> again once the code push lands". Card 7 makes cancel meet the `code_tip` wait. A single-transition run whose owing machine is gone is never given up by card 12, so the refusal names no step the operator can take.

### [MEDIUM] Card 12 gives up a tip this machine holds

**Location:** `_lym/plan/12-code-tip-grace.md`

**Issue:** The owner's own code side can fail with a class that is not owed, such as `other` or `not_fast_forward`. `PushOwed` then stays false, the verdict is `code_tip`, and after the grace `ReturnToPreviousTransition` discards a transition whose tip sits in this machine's own task worktree. Card 6 pushes such a tip in the hand-over (`pushOwedCodeTip`); card 12 has no equivalent.

### [MEDIUM] A detached spawn of the batten run record is neither moved nor named as staying

**Location:** `_lym/plan/00-overview.md`, `Decision: out-of-part`

**Issue:** `internal/pairteardown/followup.go:107` spawns `fabricengine.SpawnStatusPush` after committing the removal of a cancelled run's record. D1 retires every detached status push that carries a run transition. The overview's list of pushes that stay detached does not name this one, and no card moves it.

### [LOW] markTaskDone's re-sign arrow is missing

**Location:** `_lym/plan/11-finalize-push-order.md`

**Issue:** `markTaskDone` today returns nothing (`finalize.go:711`). Card 11's Intent makes a failed write return an error, which changes its results, so the Edit glyph needs the arrow.

### [LOW] The fabriccli doc keeps the retired respawn

**Location:** `_lym/plan/03-pair-push-records-first.md`

**Issue:** `internal/fabriccli/doc.go:93` still says the removal's "push children it respawns keep their own gates". Card 4 deletes `respawnPairPush`, and no card edits that doc. The repo-invariants and affected-set lenses both found this. Their proposed fix, adding the doc to card 4, is refused by `uses-later-target`, so the fix routes it through card 3.

### [LOW] Card 8's owed set disagrees with its own rationale

**Location:** `_lym/plan/08-transition-sync-push.md`

**Issue:** Card 8 excludes a bare `ErrPairPushRunning` because "it writes no record, so `ReadRun` could not see it". It includes `ErrPushLockBusy`, which `pushLockBusyError` (`pushanchored.go:60-63`) returns without a record either.

### [LOW] The detached child builds the machine names apart from ThisMachine

**Location:** `_lym/plan/05-rejected-push-takeover-read.md`

**Issue:** The overview's `names-from-the-caller` says `pairteardown.ThisMachine` fills `MachineNames` and the registry part adds its names there. Card 5 has `addWeftVerbs` build them from `machinename.Names` directly. The registry part would then have to change two sites, and the detached child would miss former names until it did.

## Rejected

- The affected-set lens noted that card 3 lists `internal/loomcli#pullRequestDepsGuardedBy`, which does not call `PushPairAnchored` directly. The lens itself called this over-listing, not a gap, and no check or rubric item makes it a defect.

## Notes on the focus directives

- **Fidelity.** The decision record settles D1 and D2 for this part. D3 to D9 belong to the sibling parts the split moved them to, per the arbiter's two `Added after Discussion` entries and the Goal's Takes and Delivers. The overview's out-of-part exclusions of hold, registry, catch-up and adopt's production writer are licensed by those entries. Every D1 and D2 bullet is carried by a card, apart from the gaps raised above (F2, F6, F12).
- **Card order.** No card uses a symbol only a later card creates. `MachineNames` and `PairPush` come from card 3, `IsPushOwed` from card 8 before card 9, and `PushRun` from card 6 before cards 8, 9, 11 and 12.
- **Board entry.** No card changes this task's board entry, and none files or promotes a task. Card 10 changes board-module code only.
- **Verify lines.** No card carries a `**Verify:**` line, so none names a raw test command.
- **Test economy.** Card 14 plans no test of PATTERN prose. Cards 3, 4 and 7 add rows to existing tests. Card 8's new tests cover a new exported function and the new retry behaviour, which no existing test covers.
- **Re-sign arrows.** `PushPairAnchored`, `coalescePushSides` and the other re-signed members carry their arrows. `ReadRun`'s signature does not change. The one gap is F13.

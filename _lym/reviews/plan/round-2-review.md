---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/00-overview.md (Shared Decisions), cards 2, 3, 5, 7, 8 and 9
    summary: >-
      The test-only-code guard runs at every card and batch gate, not first at card 9, so cards 2, 3, 5 and 7 fail it
      and the overview's Shared Decision rests on a false premise.
    fix: >-
      Rewrite the overview's first Shared Decision and card 9's Intent sentence on the guard: the go scoper runs every
      `//lym:guard` test after any `--card` scope, so the guard runs at each card and batch gate, and a package no main
      imports is outside the analysis and never fails it.
      Then give each card that leaves a function without a main-reachable caller an Edit target
      `cmd/lym#testOnlyCodeAllowed` and an allowlist entry per function, each Why naming the card that reaches it,
      following the existing `internal/runfsm.Order` entry.
      Card 2 lists `internal/boardengine.WriteAdopt`, `.FixHold`, `.ClearHold` and `.HoldRefusal.Error`;
      card 3 lists `internal/fabricengine.TakeOverRun`, `.DropHandOverMark` and `.OriginParentRunStatus`;
      card 5 lists `internal/adoptrecord.NewWatch`, `.Watch.Covers` and `.Watch.covers`, whose only caller
      `orchcli`'s `UnderAdoptWatch` the card deletes;
      card 7 lists `internal/battencli.HubRunSlugs`.
      Card 8 gains the same Edit target and removes the `FixHold`, `ClearHold` and `HoldRefusal.Error` entries,
      which `hubHoldCheckDeps` reaches through `checkCmd`.
      Card 9 gains it and removes every remaining entry, the `internal/adoptrecord` ones included, since an entry
      that matches no unreachable function fails the guard.
      Take the exact key list from the guard's own report at each card's tree.
      The overlay of these Edit targets passes `lym loom validate-plan --overlay` with no new blocking finding.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/plan/08-adopt-check-ends-on-hold.md (runHoldCheck step 3)
    summary: >-
      The green end removes the fix task before the hold is judged, so an adopter whose hold another machine took
      still cancels and removes a task that machine now owns.
    fix: >-
      In `runHoldCheck` step 3, pull Bolt and read the `Hold` seam once more before the first `Green` call.
      When no hold of this machine stands, end the result not green with `HoldLost` set and the line that names the
      other machine or says the hold cleared, and call neither `Green`, `ClearHold`, `BeamUp` nor `DropBoltHold`.
      `ClearHold` still judges the window left after that read.
      Add the row `AHoldTakenOverBeforeTheGreenEndLeavesTheFixTaskStanding` to `TestRunHoldCheck`:
      `Hold` reads another machine's hold after the checks, and the fake `Green` records no call.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/plan/02-board-adopt-write.md (FixHold and ClearHold)
    summary: >-
      FixHold and ClearHold report success after a push whose replay dropped their change, since the re-read refuses
      only when another machine's hold stands.
    fix: >-
      After the push, `FixHold` re-reads the local Bolt and returns success only when this machine's hold stands as
      kind `adopt-fix` naming `fixTask` with the written tries;
      `ClearHold` returns success only when no hold of this machine stands.
      Any other read refuses with a `*HoldRefusal`: the winner's machine when another machine's hold stands,
      else a refusal naming `lym adopt <subjects>`, which writes the hold and checks again.
      The replay reports a hold origin no longer holds as `Lost`, and a Bolt replay's refused findings are only
      logged, so the push then reads level with no sign of the lost change.
      Add rows to `TestWriteAdopt`:
      `FixHoldWhoseReplayLostItsHoldRefusesInsteadOfReportingSuccess` (the hook clears the hold from the second clone,
      so origin holds none when the replay runs) and
      `ClearHoldWhoseReplayLostItsRemovalRefuses` (the hook rewrites this machine's hold under the same name with other
      tries from a second process).
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/plan/09-adopt-foreground.md (runForeground step 2, movedRun)
    summary: >-
      The fix-task filer clause overrides a foreign takeover entry, so a take-hold back from the machine that drove
      the fix task skips the takeover and leaves stale local branches.
    fix: >-
      In `movedRun` and the step 2 text, give the fix-task clause the qualifier the prior-owner clause carries:
      the hold's `FixTask` is not moved when its filer reads as this machine and its origin record holds no takeover
      entry naming another machine.
      A takeover entry naming another machine moves it, which is the first clause of the same paragraph.
      Add the row `TakeHoldBackFromTheMachineThatTookTheFixTasksRunMovesItAgain` to `TestAdopt`:
      the hold names the other machine, `FixMachine` this one, and the fix task's origin record holds a takeover entry
      naming the other machine; `AddPair` is not called and `TakeOver` of that run is.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lym/plan/08-adopt-check-ends-on-hold.md (holdCheckDeps Green seam, greenDeps, hubGreenDeps)
    summary: >-
      The Green seam takes only the subject, but greenDeps gains a Rerun that no caller can fill, and hubGreenDeps is
      not an Edit target.
    fix: >-
      Give the seam the hold's re-run: `Green(subject, rerun string)`, and have `runHoldCheck` pass the re-run it
      already builds for `Check` and `Fix`.
      Add `internal/adoptcli#hubGreenDeps` -> `func hubGreenDeps(prime *lymcwd.Location, subject, rerun string) (greenDeps, error)`
      to card 8's `**Edit:**` list so it sets `greenDeps.Rerun`, and name the new parameter in the seam list of the Intent.
      The overlay of that Edit target passes `lym loom validate-plan --overlay` with no new blocking finding.
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lym/plan/09-adopt-foreground.md (Edit and Delete lists)
    summary: >-
      TestCommand_HereAdoptsTheSubjectOfItsWorktree still imports adoptrecord and reads the adopt record, and no card
      edits it, so the adoptcli test package stops compiling once card 9 removes the package.
    fix: >-
      Add `internal/adoptcli_test#TestCommand_HereAdoptsTheSubjectOfItsWorktree` to card 9's `**Edit:**` list.
      Its intent: drop the `adoptrecord.Read` assertion at the end of each case and assert instead that the hold list
      of the hub's Bolt (`boardengine.ReadOwners(h.BoardDir()).Holds`) is empty after `--here`, which the card's own
      criterion (`owned_here` writes no hold and starts nothing) requires.
      The overlay of that Edit target passes `lym loom validate-plan --overlay` with no new blocking finding.
  - id: F7
    severity: MEDIUM
    class: design
    location: _lym/plan/03-fabric-takeover-writer.md (TakeOverRun step 2, Uses)
    summary: >-
      TakeOverRun holds the slug's teardown lock and then archives through ArchiveOwnerTips, which takes the same
      lock, so the archive of the moved pair cannot get it.
    fix: >-
      In step 2, archive the moved pair through `archiveOwnerSides`, the in-package helper for a caller that already
      holds its slug's teardown lock, which `HandOverTips` uses the same way, and archive the parent's pair through
      `ArchiveOwnerTips` under the parent slug's own lock.
      State in the Intent which lock mode `TakeOverRun` holds, and that the push of the parent's records branch in
      step 4 holds the parent slug's teardown lock too.
      Add `internal/fabricengine#archiveOwnerSides` to `**Uses:**`.
      The existing row `ALocalTipOriginLacksIsArchivedAndPushedBeforeTheReset` then reaches the archive under the held lock.
      The overlay of that Uses entry passes `lym loom validate-plan --overlay` with no new blocking finding.
  - id: F8
    severity: MEDIUM
    class: scope
    location: _lym/plan/03-fabric-takeover-writer.md (TakeOverRun step 5, TestTakeOverRun rows)
    summary: >-
      Step 5 prescribes an aborted rebase that returns an error, and no row of TestTakeOverRun reaches that branch.
    fix: >-
      Add the row `ARebaseThatConflictsIsAbortedLeavesTheBranchAsItWasAndNamesTheRun` to `TestTakeOverRun`.
      Its input: the parent's records branch holds a local commit of the moved run's own status file that origin
      lacks, and the hook commits the old owner's later transition of that same file on origin at attempt 1.
      Assert that the error names the run, the local branch ends at its pre-retry tip with no rebase in progress,
      and origin holds no takeover entry.
  - id: F9
    severity: LOW
    class: consistency
    location: _lym/plan/02-board-adopt-write.md (Extends list)
    summary: >-
      Card 2 lists boardengine's doc.go under Extends while its Intent rewrites the existing sentence that
      WriteHold is the one writer of the holds.
    fix: >-
      Move `internal/boardengine/doc.go` from `**Extends:**` to `**Edit:**` in card 2 and drop the empty Extends group.
      Also name the doc's `# Machine registry and holds` field list, which omits the new `fix_machine` key.
      The overlay of that move passes `lym loom validate-plan --overlay` with no new blocking finding.
---

### [BLOCKING] The test-only-code guard runs at every card gate

**Location:** `_lym/plan/00-overview.md` Shared Decisions, cards 2, 3, 5, 7, 8 and 9.

**Issue:**
The overview says the guard first runs at card 9's batch gate because card 9 targets `cmd/lym`, and card 9's Intent
repeats it, adding that the guard fails over a package no main reaches.
Both claims are wrong against the code.
`internal/gatecli/doc.go` ("Guard step") says the verb runs the guard step under the go scoper after any `--card` scope,
over every `//lym:guard` test outside the scope, and `contracts/specs/loom-plan-spec.md` says the same of a card's call.
`TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` in `cmd/lym/testonlycode_linux_test.go` carries
`//lym:guard`, so it runs at each card's own gate and each batch gate.
`PATTERN-test-only-code` says a package that no main or test-support package imports is outside the analysis,
so card 9's reason for putting the `adoptrecord` removal there does not hold either.
The allowlist's `internal/runfsm.Order` entry ("no caller until ... that card removes this entry") is the repo's own
practice for a function that lands before its caller.
At the tree of card 2, `WriteAdopt`, `FixHold` and `ClearHold` have no caller from a main, and the same holds for card 3's
`TakeOverRun`, `DropHandOverMark` and `OriginParentRunStatus` and card 7's `HubRunSlugs`.
Card 5 deletes `UnderAdoptWatch`, the only caller of `adoptrecord.NewWatch`, which leaves `NewWatch` and the methods of
`Watch` unreachable while `adoptcli` still imports the package.
The listed constraint `PATTERN-test-only-code` is therefore broken at those cards.

### [MEDIUM] The green end removes the fix task before the hold is judged

**Location:** `_lym/plan/08-adopt-check-ends-on-hold.md`, `runHoldCheck` step 3.

**Issue:**
D6 has an adopter whose hold another machine took stop at its next hold re-read and start nothing.
Step 3 runs `Green` per subject first, and `greenEnd` removes the fix task through `fronttask.Remove`,
which cancels the run and removes the board entry.
`ClearHold` judges the hold only afterwards, so an adopter that lost the hold during a long check has already
removed a task the other machine's take-hold moved to it.
The plan tests that `BeamUp` is not called, and no row covers the removal.

### [MEDIUM] FixHold and ClearHold can report success for a change origin never took

**Location:** `_lym/plan/02-board-adopt-write.md`, `FixHold` and `ClearHold`.

**Issue:**
Card 2 says a replay refuses a commit whole only over another machine's new hold and otherwise reports a hold origin no
longer holds as `Lost`, and that a Bolt replay's refused findings are only logged.
After the push the two setters re-read and refuse only when a hold that names another machine stands.
A replay that dropped the change for another reason, such as origin clearing the hold between the pull and the push,
or this machine's own hold changing under a second process, leaves the local board level with origin and no refusal.
`FixHold` then returns an `adopt-fix` hold that does not stand, and the check end goes on to bring runs up with no hold.
The success check has to compare the re-read with what the setter wrote.

### [MEDIUM] The fix-task filer clause overrides a foreign takeover entry

**Location:** `_lym/plan/09-adopt-foreground.md`, `runForeground` step 2 and `movedRun`.

**Issue:**
The paragraph first reads a run as moved when its origin record holds a takeover entry naming another machine.
Its list of unmoved runs then qualifies the prior-owner clause with "with no foreign takeover entry" and gives the
fix-task clause no qualifier.
Take the sequence: machine A files the fix task, machine B takes the hold and its takeover moves the fix task's run,
then A takes the hold back.
The hold names A, `FixMachine` stays A, the filer reads as this machine, and the clause skips the run although its
origin record names B's takeover.
A's local branches of that run are older than origin's, and step 4's catch-up pushes A's owed commits over B's.
No row covers the take-back, whose input is reachable.

### [MEDIUM] The Green seam cannot carry the re-run greenDeps needs

**Location:** `_lym/plan/08-adopt-check-ends-on-hold.md`, `holdCheckDeps.Green`, `greenDeps`, `hubGreenDeps`.

**Issue:**
`Check` and `Fix` take the hold's re-run as a call argument, `Green(subject)` does not.
`greenDeps` gains `Rerun` for `removeFixTask`'s board-read line, and `hubHoldCheckDeps` builds its seams before any hold is read,
so nothing can fill it with `lym adopt <hold's subjects>`.
`hubGreenDeps` builds `greenDeps` and is not in the Edit list, though its result gains a field the caller must set.

### [MEDIUM] TestCommand_HereAdoptsTheSubjectOfItsWorktree is left on the retired record

**Location:** `_lym/plan/09-adopt-foreground.md` Edit and Delete lists; `internal/adoptcli/adopt_integration_test.go`.

**Issue:**
That test calls `adoptrecord.Read(h.Path)` after each `--here` case.
Card 9 deletes `internal/adoptrecord` and its Intent promises that `grep -rn adoptrecord --include=*.go` finds nothing,
but the card edits only `TestAdoptIntegration_SecondCloneTakesOver` in that file and deletes only `holdOwnerPause`.
The external test package of `adoptcli` stops compiling at card 9's gate.

### [MEDIUM] TakeOverRun archives through a function that takes the lock it already holds

**Location:** `_lym/plan/03-fabric-takeover-writer.md`, `TakeOverRun` step 2 and `**Uses:**`.

**Issue:**
The card runs `TakeOverRun` under the slug's teardown lock and archives through `ArchiveOwnerTips`.
`archiveOwnerPair` inside it takes `TeardownLockPath(hubPath, pair.Slug)` exclusive through `AcquireWriteLockWithin`,
a `flock` on a fresh handle, which waits `StatusPushLockWait` and fails with `ErrPairPushRunning`.
`HandOverTips` in `handover.go` shows the pattern for a caller that holds the lock: it calls `archiveOwnerSides`.
The card also pushes the parent's records branch in step 4 under no lock of the parent's pair, against
`PATTERN-pair-remote`'s rule that a pair's remote steps hold the slug's teardown lock.

### [MEDIUM] The rebase-abort branch of TakeOverRun has no test row

**Location:** `_lym/plan/03-fabric-takeover-writer.md`, step 5 and the `TestTakeOverRun` rows.

**Issue:**
Step 5 prescribes that a conflicting rebase is aborted, leaves the branch as it was and returns an error naming the run.
The rows cover an exhausted retry, a parent commit kept across the retry and an anchor change, and none reaches the conflict.
A reaching input exists: a local commit of the moved run's own status file on the parent's records branch against the
old owner's later transition of the same file.

### [LOW] Card 2 extends a doc the card rewrites

**Location:** `_lym/plan/02-board-adopt-write.md`, `**Extends:**` group.

**Issue:**
The Intent says doc.go says only boardengine's hold setters write the holds.
Today `internal/boardengine/doc.go` says "WriteHold is the one writer of the holds", so the card rewrites an existing
sentence, and `Extends` is for adding to a ref without changing what stands.
The field list in the same doc's registry section also omits the key card 1 adds.

## Rejected

- Card 2 case 5 spelling `lym adopt --take-hold` in its refusal: only a call that already passed the flag reaches it,
  and card 4's row denies the flag to every agent, so no agent reads the spelling as a way forward it was not already using.

- Card 1 and `ParseOwners`: `boardengine.ParseOwners` in `owners.go` already decodes `machines` and `holds` into
  `Owners.Registry` and `Owners.Holds`, and returns an empty `Owners` for empty input, so the card's reads need no new decoder.
- Card 3's retry reset of the parent's records branch: step 2 archives the parent pair's tips before step 4, and step 5
  rebases the other commits instead of resetting, so only a conflict can lose them, which F8 covers.
- Card 8 and 9 deletions in `internal/adoptcli`: every function left behind by the deleted code is reached from a main
  or listed in a card (`foreignParent` and `parentOwnerOf` through `heldBeamWayForward`, `addWarning` through
  `runBeamSteps`, `checkAlive` through `checkview.go`, `checkSubjects`, `projectOf` and `collect` named under `Uses`).
- Fronttask's start of `adopt-check`: `startAction` runs `lym adopt check <subject>` detached and appends its output to a
  file in the front-task directory, so neither the exit code nor the output is read, as card 8 says.
- Way-forward lines of the fix loop, green path and check: the Intent's sentence that every line names the hold's re-run,
  its `Rerun` parameter on `fixDeps` and `greenDeps`, and the `TestFailedEnd` row with a two-subject re-run cover
  `fixloop.go`'s inline lines, `fixReadError`, `fixFilingError`, `fixRemovalError`, `fixBody` and `check.go`'s lines;
  the one gap is F5.
- The focus line that names card 11: the plan holds ten cards, and card 10 carries the retired-wording docs it describes.

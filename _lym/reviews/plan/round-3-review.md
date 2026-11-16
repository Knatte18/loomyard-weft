---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/00-overview.md:33
    summary: "the code-tip-is-warp-sha decision reads the newest Warp-SHA commit as the run's last transition, but every fabric weft commit carries that trailer"
    fix: |-
      Make a transition commit identifiable by its own trailer, so the readers skip fabric's other weft commits.
      Overview, decision `code-tip-is-warp-sha`: the records commit of a transition carries `Warp-SHA: <code HEAD>` and a second trailer `Run-Transition: <run id>`;
      "the tip the run's last transition on origin names" is the `Warp-SHA` of the newest commit on origin's fetched records branch of the slug that carries `Run-Transition`.
      Card 6: add `- plan:internal/fabricengine#TransitionTrailerKey -> const TransitionTrailerKey = "Run-Transition"` to **Create** (admitted by every check, run with `lym loom validate-plan --overlay`);
      `lastTransitionCodeTip` selects commits by that key, not by `Warp-SHA` alone.
      Card 8: both seams' `Commit` closures pass `[]fabricengine.Trailer{{Key: fabricengine.TransitionTrailerKey, Value: shedrun.ResolveRunID(location, runID)}}` in place of `nil`;
      the `TestCommitStatusSeam_Real` row asserting the `Warp-SHA` trailer also asserts the `Run-Transition` trailer.
      Card 12: `ReturnToPreviousTransition` reads "the newest commit carrying `Run-Transition`" and "the parent" as the previous such commit, and its commit passes the same trailer;
      its single-transition rule counts `Run-Transition` commits.
      Card 7: `TestOriginCodeTip` gains a row where a `lym fabric commit` style commit, carrying `Warp-SHA` and no `Run-Transition`, is the newest on origin's records branch and the tip read is still the last transition's.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lym/plan/06-run-hand-over.md:107
    summary: "TestHandOverTips cannot set the unexported owedTipPushedHook from the external fabricengine_test package, and no card creates the setter"
    fix: |-
      Card 6, **Create**: add `- plan:internal/fabricengine#SetOwedTipPushedHookForTest -> func SetOwedTipPushedHookForTest(t testing.TB, hook func())`, admitted by every check.
      Say in the **Intent** that it lives in `internal/fabricengine/export_test.go`, sets `owedTipPushedHook` and restores nil in `t.Cleanup`,
      as `SetArchiveTagPushHookForTest` does, and that `TestHandOverTips`' give-up row calls it.
  - id: F3
    severity: LOW
    class: consistency
    location: _lym/plan/08-transition-sync-push.md:35
    summary: loomCommitStatusDeps keeps a doc comment describing the Spawn field card 8 removes, and no card targets that function
    fix: |-
      Card 8, **Edit**: add `- internal/loomcli#loomCommitStatusDeps` after `internal/loomcli#commitStatusDepsGuardedBy`, admitted by every check.
      Its doc (`wiring.go`, "Spawn starts the detached push ... A Spawn the check stops starts no push") is rewritten to name the synchronous `Push` and the `Admit`, `Owed` and `Sleep` seams in place of `Spawn`.
---

### [BLOCKING] A transition's code tip is read from every Warp-SHA commit, not from the transitions

**Location:** `_lym/plan/00-overview.md:33`, `_lym/plan/06-run-hand-over.md:54`, `_lym/plan/07-read-run-owed-and-tip.md:41`, `_lym/plan/12-code-tip-grace.md:48`.

**Issue:**
Focus question 2 asks whether the `Commit` closures write the `Warp-SHA` trailer.
They do once card 8 swaps `CommitAnchoredPaths` for `CommitAnchoredPathsWithTrailers(..., nil, opts)`:
`messageWithTrailers` (`internal/fabricengine/trailercommit.go:99`) appends `Warp-SHA: <warp HEAD>`, so that part of the plan holds.

The decision built on it does not.
The overview defines the tip "the run's last transition on origin names" as the `Warp-SHA` of the newest commit carrying one on the slug's records branch.
Cards 6 (`lastTransitionCodeTip`), 7 (`OriginCodeTip`) and 12 (`ReturnToPreviousTransition`, "the parent of the newest commit carrying a `Warp-SHA`") all read it that way.
Transitions are not the only writers of that trailer.
`Fabric.Commit` appends it to every weft commit through `commitWeftLocked` (`internal/fabricengine/recordsgit.go:235`),
and `lym fabric commit`, `lym fabric sync` and `lym fabric push` (`internal/fabriccli/weft_verbs.go:198,280,380`) and `webstercli`'s sync run it on a pair's records branch.
The anchor re-anchor commit (`pull.go:773`) carries one too.

So a `lym fabric sync` made mid-run lands a newer `Warp-SHA` commit that is no transition.
Its tip is the code HEAD at that moment, and with records now pushed first, the code side can trail it or fail.
- `ReadRun` then reads that tip as missing and the run waits on `code_tip` for a tip no transition named.
- Past the grace, `ReturnToPreviousTransition` reads the status at the parent of that fabric commit, which is the current transition's status.
  It commits that same status with a `Warp-SHA` naming the same missing tip, so the give-up changes nothing and repeats every tick.
- The hand-over's owed-tip push (card 6 step 1) and the owner's guard (card 7) act on the same wrong tip.

The decision record asks only that the records commit of a transition names its code tip (D1, D2); it does not choose the trailer, so a second trailer on the transition commits is within it.
The overview's rationale, "no second key can disagree with it", is what the fix gives up on purpose:
a key shared with fabric's own commits is the disagreement.
The fix keeps `Warp-SHA` as the tip's value and adds a marker that only the two transition seams write.

### [MEDIUM] The give-up hook cannot be reached from the external test package

**Location:** `_lym/plan/06-run-hand-over.md:107`, with the hook created at `_lym/plan/06-run-hand-over.md:13`.

**Issue:**
Card 6 creates `owedTipPushedHook` as an unexported package variable and names `internal/fabricengine_test#TestHandOverTips`, an external test package, as the test that sets it.
The package's convention for such hooks is a `Set...HookForTest` helper in `internal/fabricengine/export_test.go` (`SetArchiveTagPushHookForTest`, `SetScratchBeforeLinkHookForTest`).
No card creates or edits one, so the row "a tip whose transition the test gives up on origin through `owedTipPushedHook`" cannot be written as planned.
That row is the only test of the re-read-and-reset path that D2's acceptance criterion about a tip given up between the owing machine's read and its push depends on.

### [LOW] A doc comment keeps describing the removed Spawn field

**Location:** `_lym/plan/08-transition-sync-push.md:35`; the comment is `internal/loomcli/wiring.go:194-199`.

**Issue:**
Card 8 removes `Spawn` from loomcli's `commitStatusDeps`, and card 9 deletes `prePushCheck.guardedSpawn`.
The doc comment of `loomCommitStatusDeps` still reads "Spawn starts the detached push ... Commit, Spawn and Push each pass the run's pre-push check first" and "A Spawn the check stops starts no push".
`loomCommitStatusDeps` is not a target of card 8 or card 9, so the stale text survives both.

## Focus settled

- Cards 6 and 11, `git reset --keep`: no defect.
  Both planned call paths (`archiveParentMerge` in card 6 step 3, `ArchiveParentMerge` in card 11 over it) end in `resetKeepUnderArchive`, a new executor in `destroy.go`.
  It runs `checkPathRequest`, which checks containment, ownership, dirtiness and force in that order (`destroy.go:858`), with `ownedWarpCheckout` (which admits the prime checkout) and `dirtinessNA` with a stated reason.
  No other planned card calls `ResetKeep`, and the destructive guard's `.ResetKeep(` ban holds for the new code because it lives in `destroy.go`.
- Card 8, `Spawn` callers: no defect.
  Every non-test caller of battencli's `commitStatusDeps.Spawn` (`commitstatus.go:119,139,184`, `cancelcmd.go:484`, `readopt.go:98`) is a card 8 target.
  So is every test caller (`commitstatus_test.go`, `cancel_test.go:1167`, `run_test.go:74`, `driverdeps_test.go:226`).
  Loomcli's field, its wiring (`wiring.go:217,339`), its test uses and `TestPrePushCheckWiredIntoDeps` are card 8 targets too;
  `guardedSpawn` and `TestPrePushCheckGuardedSeams` stay for card 9, which compiles in between.
  The `cleanup.go` `Spawn` is another type and stays.
- Card 3, shared prime records branch: no defect.
  `recordPushFailure` unions `Runs` (card 3), a landing push clears the shared record (as the shared branch's push covers every run's commits), and `FailedPushesOwedBy` matches by `Runs`.
  Card 7's `TestFailedPushRecord` row reads a record two runs' failures wrote as owed by each of them.
- Cards 6 and 7, `updateRemoteBranch`: no defect.
  Both name the executor, `ownedPairWarpCheckout`, the lease at the origin tip read, and `dirtinessNA` with the reason that moving origin's branch touches no checkout file, so a worktree holding uncommitted files passes the gate.
  Card 6 makes the executor wrap `gitrepo.ErrLeaseRejected`, and card 7 uses that after it.

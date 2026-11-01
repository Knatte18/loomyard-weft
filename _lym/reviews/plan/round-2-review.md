---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/plan/04-batten-cancel-export.md:26
    summary: "No named test reaches the end-only error through Cancel; the tests call only the error constructor endOnlyError, so Cancel could return nil for an end-only cancel and fronttask.Remove would count the cancel done"
    fix: "In 04-batten-cancel-export.md replace the Create entry for `endOnlyError` with `plan:internal/battencli#cancelJudgingEndOnly` -> `func cancelJudgingEndOnly(ctx context.Context, slug string, remove bool, deps cancelDeps) error`. Reword the Intent so `Cancel` builds `hubCancelDeps(prime, slug)` and returns `cancelJudgingEndOnly(ctx, slug, remove, deps)`, and `cancelJudgingEndOnly` runs `cancelRun` and, when it returns no error, returns nil for a full cancel and the end-only error for a report reading `EndOnly`. Reword the TestCancelRun sentence so its end-only rows assert that `cancelJudgingEndOnly` over the row's fake returns that error, naming origin unreachable and `cancelCommand(slug, remove)`, not matching `ErrNothingLeft`, and its full-cancel rows assert nil. The combined `validate-plan --overlay` run with this fix and F2 to F4 applied reports no blocking finding."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lym/plan/13-check-fix-loop.md:49
    summary: "No card rewords the failed-tier notice in runCheck, so it keeps its way forward to fix it and run lym gate test --gate adopt in the prime, which contradicts the fix loop and names the wrong worktree for a beam subject"
    fix: "In 13-check-fix-loop.md add one Intent sentence after the no-verdict paragraph saying the failed notice `runCheck` posts for a failed tier loses its `fix it, then run <re-run> in the prime` way forward, names the failed tier and the log only, and the lines `Failed` returns say what happens next. In the same card's TestRunCheck bullet make the failed-tier row assert the failed notice names neither `in the prime` nor `lym gate test --gate adopt`. No target changes, so `redundant-file-target` and `redundant-package-target` are not engaged."
  - id: F3
    severity: LOW
    class: scope
    location: _lym/plan/12-check-green-end.md:71
    summary: "docs/overview.md still says lym adopt starts a run's first pass detached; card 12 moves that to the green check and edits battenshed/doc.go for the same sentence, but no card edits the overview paragraph"
    fix: "In 12-check-green-end.md add `docs/overview.md` to the Edit list after `internal/battenshed/doc.go`, and add one sentence after the battenshed/doc.go sentence saying the overview's batten paragraph that says `lym adopt` starts a run's first pass names the adopt check's green end in its place, beside `lym beam land`. The card lists the file alone, with no member of it, so `redundant-file-target` and `redundant-package-target` admit it."
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lym/plan/11-adopt-holds.md:78
    summary: "reconcileWarnings warns on every ahead branch of the prime or an adopted beam, including a records branch the reconcile replayed (ahead with Moved set), whose line then says left unmoved against D2 and sends the operator to lym fabric pull for nothing"
    fix: "In 11-adopt-holds.md change the D2 paragraph so a branch warns only in state rewritten, diverged or ahead and not Moved, and add to the TestAdopt rows that a records branch of the prime the reconcile replayed, reading ahead with Moved set, gets no warning. State that the worktree named after `lym fabric pull in` is the outcome's `CodeWorktree` for a records branch and its `Worktree` for a code branch. No target changes, so the overlay admits it."
---

### [MEDIUM] No test reaches the end-only error through Cancel

**Location:** `_lym/plan/04-batten-cancel-export.md:26`

**Issue:** The decision record's cancel rule (D8) says a cancel that cannot reach origin fails the removal's first step.
In `internal/battencli/cancel.go`, `judgeOrigin` returns `endOnly` with a nil error, and `cancelRun` returns `report, nil` before `clearRunPhase` (lines 200 to 203), so an end-only cancel leaves the run status and the run record in place.
The plan handles this: card 4's `Cancel` turns an `EndOnly` report into an error, card 6's `Remove` step 1 fails on any error that does not match `ErrNothingLeft`, keeps record and entry, and `HandRemoval` names `lym batten cancel <slug> --remove` first, then `lym board remove <slug>`.
`TestRemove` reaches the failed step with a fake `Cancel` returning an error, and that part is sound.
The link that makes the error appear, `Cancel` calling `endOnlyError` over the report, has no test: `Cancel` builds the real `hubCancelDeps`, so no unit test runs it, and `TestCancelRun`'s end-only rows call only `endOnlyError` over the returned report.
That is a test of the error constructor, which does not reach the refusal.
If `Cancel` returned `nil` for an end-only report, every named test would still pass, and `Remove` would go on to a removal the run lock refuses.
The fix moves the composition into a function over `cancelDeps` that `TestCancelRun`'s fakes can drive.
The round-1 fixer put the conversion in `endOnlyError` so the fakes reach it, which is true of the conversion and not of the call that makes `Cancel` use it.

### [MEDIUM] The failed-tier notice keeps a hand way forward in the prime

**Location:** `_lym/plan/13-check-fix-loop.md:49`

**Issue:** `internal/adoptcli/check.go:226` posts `adopt check: failed, <tier>; log <path>; way forward: fix it, then run lym gate test --gate adopt in the prime`.
Card 13 rewords the notices of the two no-verdict ends to name `lym adopt <subject>`, and card 5 only changes the notice prefix and `tierNotice`'s signature.
No card rewords this notice's way forward, so after the plan it posts a hand fix and a gate re-run in the prime, followed by the lines `Failed` returns that say a fix task was filed.
For a beam subject the gate ran in the beam's pair, so "in the prime" names the wrong worktree; for both subjects it contradicts D6, where the check's landing fix starts the next check by itself.
`runCheck` is on cards 5 and 13's Edit lists, but neither Intent covers this text.

### [LOW] docs/overview.md still says lym adopt starts a run's first pass

**Location:** `_lym/plan/12-check-green-end.md:71`

**Issue:** `docs/overview.md:673` says "`lym adopt` starts a run's first pass detached with the prime as its working directory".
After card 11 adopt starts none, and card 12's green end does; card 12 edits `internal/battenshed/doc.go` for the same sentence.
Cards 5 and 11 edit `docs/overview.md`, but their Intents name only its adopt paragraph (line 445 to 447) and its `lym beam status` paragraph, not this batten paragraph.

### [MEDIUM] reconcileWarnings warns on a branch the reconcile moved

**Location:** `_lym/plan/11-adopt-holds.md:78`

**Issue:** D2 warns on a branch "that the reconcile leaves unmoved because it was judged `rewritten`, `diverged` or `ahead`".
`reconcileRecordsBranch` in `internal/fabricengine/originreconcilemoves.go` returns `BranchAhead` with `Moved` set when it replayed local commits onto origin's head, and `sharedStopText` passes `BranchAhead` through without a stop.
Card 11 selects every outcome of the prime or of an adopted beam in state `rewritten`, `diverged` or `ahead` with no `Moved` test, so a prime or beam records branch replayed by an ordinary adopt gets `<branch> is ahead against origin and was left unmoved: run lym fabric pull in <worktree>`.
That line is false and sends the operator to a verb that changes nothing.
The card also leaves `<worktree>` open although `BranchOutcome` carries both `Worktree` and `CodeWorktree`, and a records branch's fabric verbs run in `CodeWorktree`.
The `TestAdopt` rows cover rewritten, diverged and ahead code branches, not this case.

## Rejected

- Focus 1, the `Remove` outcome for an unreachable origin: settled as sound apart from F1. Card 6 fails step 1, keeps record and entry, and `HandRemoval` names `lym batten cancel <slug> --remove` first; `TestRemove` has the row.
- Focus 2, `lym fabric pull` in every warning: settled for the branches D2 names. Card 11's line names `lym fabric pull` for every selected branch, a diverged code branch with its own `WayForward` included, which follows as detail; `TestAdopt` has a row for each. The remaining gap is F4.
- Focus 3, the fix task's label: settled. `validateLabels` refuses an unknown label, and the board template's `labels` map is empty with `bug` a type, so `gate` is unknown on a default hub. The decision record's `Added after Discussion` entry records the choice, and card 13 step 5 and its `TestFailedEnd` row carry it.
- Focus 4, an own fix task holding a run status with no readable run record: settled. Card 13 case 1 classifies it as open and names it as starting, files and removes none, and `TestFailedEnd` has the row.
- Focus 4, `Settle`'s drop of records for an unowned project: settled. Card 6 states the bound, one redundant check of a subject the machine owns again, and the fix-try limit bounds any loop after it.
- Round 1's deferred question whether D2's way forward may follow the branch state: no finding. Card 11 follows the decision record's wording, and the record is the measuring stick, never the subject.
- Focus 5, other texts that say adopt resumes: covered. `OwnedElsewhereError.Error`, `heldBeamWayForward`, `ownerWriteError`, the lost-owner and read-back errors, `ownerHaltReason`, `beamcli` doc and `createCmd` help, `adoptcli/doc.go` and the `Command` help are each named in card 11 or 12's Intent. The uncovered ones are F2 and F3.

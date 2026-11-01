---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/plan/06-front-task-package.md:89
    summary: TestRemove's row where the fake RemoveEntry calls Settle while the removal holds the record lock would hang as written, since the lock is a blocking flock.
    fix: In card 6's Tests section, reword the TestRemove row "a released never-started entry whose fake `RemoveEntry` calls `Settle` while the removal holds the lock" so that the fake `RemoveEntry` starts `Settle` in a goroutine instead of calling it, and the test waits for that goroutine once `Remove` has returned. Name the reason in the row. The record file's lock is a blocking flock (`internal/lock`, `internal/state.UpdateJSON`), so a synchronous `Settle` inside the held lock never returns. Keep the assertion that `Settle` records nothing and runs no action, and say that it holds whichever way the goroutine interleaves with the removal, because `Settle` either reads the entry and record both present or reads both gone. Keep the second half of the row, a `Settle` called after `Remove` returns, as it stands.
  - id: F2
    severity: LOW
    class: design
    location: _lym/plan/11-adopt-holds.md:89
    summary: No TestAdopt row pins reconcileWarnings naming the pair's code worktree for an unmoved records branch, so a warning that named the records worktree would pass.
    fix: In card 11's Tests section, add to the TestAdopt rows one for a records branch of an adopted beam's pair that the reconcile left unmoved in state diverged, with Worktree set to the records sibling and CodeWorktree set to the pair's code worktree. The row asserts that the warning names `lym fabric pull` in the CodeWorktree path and not in the records Worktree path.
---

### [MEDIUM] TestRemove's lock-race row calls Settle inside the held lock

**Location:** `_lym/plan/06-front-task-package.md:89`, the `TestRemove` bullet; the lock design is at lines 68 and 73.
**Issue:** The card has `Remove` run its drop of the record and its entry removal under the record file's lock, and has `Settle` hold the same lock across its read of the board and its writes.
The `TestRemove` row says "a released never-started entry whose fake `RemoveEntry` calls `Settle` while the removal holds the lock".
Both locks are `internal/state` locks, which `internal/lock` implements as `gofrs/flock` blocking file locks.
`state.UpdateJSON`'s own doc says that nesting a second acquire on the same lock path "hangs rather than failing".
A `Settle` called synchronously from inside `RemoveEntry` therefore waits for a lock that `Remove` holds on the same goroutine and never returns.
The row is the only test of the race that the card's lock exists to prevent, so a writer who follows it literally gets a hung tier0 test, and one who works around it is guessing what the row meant.
Running `Settle` in a goroutine gives the intended coverage deterministically: the outcome is the same whichever side the lock goes to first, and the test waits for the goroutine after `Remove` returns.

### [LOW] The records-branch worktree choice of reconcileWarnings has no row

**Location:** `_lym/plan/11-adopt-holds.md:79` for the rule and `_lym/plan/11-adopt-holds.md:89` for the `TestAdopt` rows.
**Issue:** Card 11 says the worktree named after `lym fabric pull in` is the outcome's `CodeWorktree` for a records branch, where its fabric verbs run, and its `Worktree` for a code branch.
That is the right choice: in `originTargets` a records branch's `worktree` is the records sibling and its `codeWorktree` is the pair's code worktree.
The `TestAdopt` rows cover a rewritten prime code branch, a diverged code branch of an adopted beam's pair, an ahead prime branch with `Moved` unset, and a replayed records branch that gets no warning.
No row has an unmoved records branch, so the only line of code that carries the per-kind rule is exercised for code branches alone.
An implementation that used `Worktree` for every branch would pass every named row and tell the operator to run `lym fabric pull` in the records worktree, where card 11 says the fabric verbs of a records branch do not run.

## Rejected

- `battencli.Cancel`'s conversion of an end-only report into an error: card 4 puts that conversion in `cancelJudgingEndOnly`, and `TestCancelRun`'s end-only rows drive it through `newCancelFake`.
  They assert the error, the unchanged run status and run record, that the error does not match `ErrNothingLeft`, and nil on a full cancel.
  `TestRemove` then drives `Remove` step 1 with an end-only error and asserts that record and entry stay and that the hand removal names `lym batten cancel <slug> --remove` first.
  `Cancel` is a one-line composition of `hubCancelDeps` and `cancelJudgingEndOnly` that no fake reaches, so a `Cancel` that skipped the helper would pass; I judge that wiring and not a coverage gap, because the logic is pinned in the helper.
- `runCheck`'s notices and the fix loop: the reconcile-failure notice, the no-verdict gate exit, the failed-tier notice, the notice for the own open fix task, the other open front task, the exhausted tries, the failed removal, the filing error and the `hubFixDeps` build error each name `lym adopt <subject>` or the task's hand step.
  The failed-tier notice names the tier and the log only and does not name a worktree, and the gate-exit notice names none either, as card 13 states.
  `TestRunCheck`'s rows assert that neither `in the prime` nor `lym gate test --gate adopt` appears.
  I found no notice that has no way forward or one that conflicts with the fix loop.
- Sentences that say `lym adopt` starts or resumes a run's first pass: `docs/overview.md`'s adopt paragraph (card 11) and its batten paragraph (card 12), `internal/battenshed/doc.go` (card 12), `internal/adoptcli/doc.go` (cards 11, 13 and 14), `internal/beamcli/doc.go` and `createCmd`'s help (card 11), `OwnedElsewhereError.Error` in `internal/battencli/owner.go` (card 11) and `ownerHaltReason` in `internal/landingshed/finalize_owner.go` (card 12) are each reworded by some card.
  A repo-wide search for adopt beside resume, first pass and starts found no other such sentence outside `internal/adoptcli`.
- Card 11's `reconcileWarnings` excludes a records branch the reconcile replayed: it reads `ahead` with `Moved` set, and the card admits `ahead` only with `Moved` unset.
  `BranchOutcome.Moved` is documented that way in `internal/fabricengine/originreconcile.go`.
  The Bolt branch has `Prime` false and an empty `Slug`, so it is excluded by the card's `Prime` or adopted-beam test.
  The worktree choice matches `originTargets`: a code branch has `Worktree` equal to `CodeWorktree`, and a records branch has `Worktree` as the records sibling and `CodeWorktree` as the pair's code worktree.
  The choice itself is correct; F2 is the missing row that pins it.

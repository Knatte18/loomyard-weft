---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/08-batten-notice-log.md (Intent, Path paragraph)
    summary: Card 8 says the branch already contains lyx cleanup (bc9ee7a02), but bc9ee7a02 is not an ancestor of HEAD, so the decision record's rebase-before-D8 constraint is carried by no card
    fix: In 08-batten-notice-log.md, replace the sentence `The branch already contains lyx cleanup (bc9ee7a02).` with a stated precondition, for example `Card 8 is dispatched only once the task branch is rebased onto a main that contains lyx cleanup (bc9ee7a02); at plan time HEAD does not contain it, and the rebase is the orch's, never a card's.` In the same paragraph, drop the clause `(battencli's cleanup walks the two shed roots only)`, which describes code absent at HEAD, or restate it once the rebase has landed and the sweep roots can be read at HEAD. Raise the rebase to ly:orch so it happens before approval, since the rebase can move the test-budget rows and targets the other cards cite (see F2).
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/08-batten-notice-log.md, _lyx/plan/09-batten-overview.md, _lyx/plan/10-orch-runs-strand.md (test-budget bullets)
    summary: The battencli and orchcli test-budget starting values in cards 8, 9 and 10 do not match cmd/lyx/testdata/test-budget.yaml at HEAD, and the final rows over-raise both budgets
    fix: Make each test-budget bullet start from the row of the tree the card runs on. At HEAD that is card 8 `internal/battencli 64 → 65`, card 9 `internal/battencli 65 → 68` and card 10 `internal/orchcli 26 → 27`. If F1's rebase lands first, re-read both rows after it and write the values that tree gives instead.
---

### [BLOCKING] Card 8 rests on a commit the branch does not contain

**Location:** `_lyx/plan/08-batten-notice-log.md`, Intent, the Path paragraph.

**Issue:**
The decision record's Constraints say the notice log's path must lie outside every directory `lyx cleanup` sweeps, and that "the branch is rebased onto a `main` that contains `lyx cleanup` (bc9ee7a02) before D8 is built".
Card 8 meets this by asserting "The branch already contains `lyx cleanup` (bc9ee7a02)."
That is false at HEAD.
`git cat-file -t bc9ee7a02` gives `commit`, but `git merge-base --is-ancestor bc9ee7a02 HEAD` exits 1.
HEAD also has no `lyx batten cleanup`: `internal/battencli` mentions "cleanup" only in remedy strings for `lyx fabric cleanup` (`arm.go:506`, `wire.go:319`).
That makes the card's clause "`battencli`'s cleanup walks the two shed roots only" a description of code the worker cannot read.
So no card carries the constraint, and nothing in the plan checks the notice log's placement against the real sweep roots before D8 is built.
Round 1's review notes say "the branch HEAD is `bc9ee7a02`".
That does not hold for this worktree: HEAD is `d22189d84`, and bc9ee7a02 is not among its ancestors.

### [MEDIUM] Test-budget starting values disagree with the budget file

**Location:** the test-budget bullets of cards 8, 9 and 10.

**Issue:**
`cmd/lyx/testdata/test-budget.yaml` at HEAD has `internal/battencli: 64` and `internal/orchcli: 26`.
A count of top-level `func Test` declarations agrees: 64 and 26.
Card 8 writes `68 → 69`, card 9 `69 → 72` and card 10 `27 → 28`.
The new tests are one on card 8, three on card 9 and one on card 10, so they need 68 and 27.
The plan's 72 and 28 leave slack that `budgetViolations` (`cmd/lyx/testbudget_test.go`) never reports, which weakens the guard.
A worker told "68 → 69" while the file reads 64 also has no instruction that matches the file.
The +4 offset for `battencli` is consistent with the plan having been written against a tree that already holds bc9ee7a02's `lyx batten cleanup` tests.
So these values are another symptom of F1, and they must follow whichever tree the cards finally run on.

## Focus directive outcomes

- Seam registration: card 8 lists `internal/testkit/envkit#nilLegal` among its targets, and its Intent names all three new paths: `InnerRun.NoticeLog.Now`, `Land.Notices.NoticeLog.Now` and `Teardown.Notices.NoticeLog.Now`.
  `shedrecipe.Env` reaches `NoticeLog` through exactly these struct fields: `InnerRun battenshed.InnerRunDeps`, plus `Land` and `Teardown`, each of whose `Notices RowNoticeDeps` gains the field.
  `walkNil` (`envkit.go:340`) recurses into struct fields and reports a nil func unless its dotted path is in `nilLegal`.
  So `NilSeams` would have reported exactly those three paths, and card 8 lists all of them.
  No other card adds a nil-defaulting func field reachable from `Env`.
  Not a defect.
- Shared formatting: only card 5 creates `plan:internal/statuspane#FormatMinutes`.
  Cards 7 and 9 each list it under `Uses:`, and neither card implements a format of its own.
  The `validate-plan --overlay` baseline over `{}` reports `ok: true` with informational `create-new-unit` findings only, so neither target list trips `redundant-package-target` or `redundant-file-target`.
  Not a defect.

## Rejected

- Card 12's reed test and the tmux-tier rows of cards 3, 4 and 7 are only vetted under `-tags tmux` and never run by `verify:`.
  Rejected: the hub's verify-tier list is `none`, and tmux-tier tests are not hermetic, so the coverage rule does not reach them.
- Card 7's checklist verb resolves its cwd through `lyxcwd.CwdFrom` and `lyxcwd.Resolve` rather than `cliwire`.
  Rejected: `loomcli` already does this at `circling.go`, `decision.go` and `review.go` for verbs outside the armed pre-run, and `PATTERN-cliwire-sole-wiring` governs standalone target-dir wiring, which this verb does not do.

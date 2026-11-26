---
verdict: BLOCKING
findings:
  - id: F19
    severity: BLOCKING
    class: design
    carries: F17
    location: _lym/discussion/decision-record.md#D5
    summary: "D5's history search still names another run's commit or none: Finalize's own merge-in fast-forwards a hand-merged task branch to the parent's head, and a squash landing (Finalize's own pull-request merge included) reports up to date with the task tip no ancestor of the parent"
    fix: "In D5, delete the premise that the merge reports up to date only when the task tip is an ancestor. State the two facts instead, with their pins: an empty squash also reports up to date (TestMergeCrucible_DerivedAlreadyUpToDateIsReadFromTheRecord in internal/fabricengine/mergecrucible_integration_test.go), and the merge of the parent into the task branch runs with --ff (Repo.MergeStart in internal/gitrepo/merge.go). Then redefine the search: (a) it runs over the task's own tip as it stood before Finalize first merged the parent into the task branch, kept durably before that merge so a resume has it, never over the branch tip after the merge-in; (b) a pull-request merge Finalize makes itself records the merge commit GitHub reports for it; (c) the up-to-date path with no record and the archive path share one rule, ancestry first and a squash fallback second, and the fallback counts only a commit that holds the task's work while its first parent does not; (d) when the task adds no net code change or no commit qualifies, the row is written with no landing commit and no landing time, and says so. In Acceptance criteria, require the hand-merge case to run through a Finalize that does its own merge-in, add a pull-request squash landing whose row names the squash commit and an adopter-first squash with another commit on top, and replace the empty-diff clause with the outcome of (d)."
  - id: F20
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md#D5
    summary: "D5 reads the landing commit back from the record on every up-to-date merge, so after the reset lym's own way forward prescribes (git reset --keep origin/<parent>) the row names the discarded local merge, a commit the parent branch does not hold"
    fix: "In D5, read the commit back from the landing record only when fabric shows it is an ancestor of the parent branch's head; otherwise treat the record as absent, find the commit by the no-record rule and replace the record. Replace the sentence that a hand-reset parent gets a new merge and a new record, which holds only when the reset parent lacks the work. Add a D5 acceptance criterion: a parent push refused as a protected branch, the work then merged on GitHub by hand and the parent reset to origin, gives a row naming the commit on origin's parent."
  - id: F21
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md#D1
    summary: "D1 adds a tracked .gitignore on the records side, which PATTERN-fabric-boundary forbids (the one weft-side .gitignore writer is boardengine's in the Bolt checkout), and PATTERN changes declares no change to it"
    fix: "In D1, replace the tracked .gitignore with an ignore line for the kept directory's .tmp-* names in the records repository's .git/info/exclude, written only through fabricengine, and state where it is seeded so that it is in place before the first write in a new pair, an already wired pair and a pair adopted on another machine. Update the references to D1's ignore rule in D1's rationale and in D3 to match. The D1 .tmp-* acceptance criterion stays as worded."
prior:
  - id: F16
    status: resolved
  - id: F17
    status: open
    answer: "The head is no longer read on these paths, but the replacement search runs over a task tip that Finalize's merge-in has already moved, and rests on a premise the squash merge breaks; carried as F19."
  - id: F18
    status: resolved
---

### [BLOCKING] D5's history search names another run's commit, or none

**Location:** `_lym/discussion/decision-record.md#D5`

**Issue:** This carries F17.
The settled focus question is answered yes: another run's commit can still be taken as this task's landing commit.

Finalize merges the parent branch into the task worktree before the parent-side merge
(`internal/landingshed/finalize.go:379`, `:1221`), with `git merge --ff` (`internal/gitrepo/merge.go:70`).
Take a hand merge H of task tip T, then another run's commit X on top.
T is an ancestor of X, so the merge-in fast-forwards the task branch to X.
The parent-side merge then reports up to date.
The oldest first-parent commit that has the task branch's tip (now X) as an ancestor is X, the other run's commit.
That is the case D5's own acceptance criterion says gives a row naming the hand merge.

The premise "the merge reports up to date only when that tip is an ancestor of the parent's head" is false.
`squash` defaults to true (`internal/landingshed/template.yaml:2`), and an empty squash reports up to date
with a source that is no ancestor.
`TestMergeCrucible_DerivedAlreadyUpToDateIsReadFromTheRecord` asserts exactly that (read, not run).
So on the no-record path no first-parent commit has the tip as an ancestor after any squash landing:

- every pull-request landing Finalize makes itself by squash, a beam's included,
  because `mergeOnGitHub` merges on GitHub and the parent-side merge then finds nothing to add
  (`finalize.go:398-420`) with no record written yet;
- a hand squash merge, and an adopter-first squash;
- a task whose commits leave its landed diff empty, which D5's criterion also expects a named commit for.

D5 gives the `HoldsWork` fallback to the archive path alone, so these have no rule.
On the archive path the fallback runs over the merged-in tip too,
and a task with no net change is held by every commit,
so the first one after the merge-base, whoever made it, is picked.

### [BLOCKING] D5 trusts a landing record the parent was reset past

**Location:** `_lym/discussion/decision-record.md#D5`

**Issue:** D5 writes the record before the parent push, and reads it back whenever a later merge is up to date.
The protected-branch refusal tells the operator to merge the task branch on GitHub by hand,
run `git reset --keep origin/<parent>` and resume (`finalize.go:887-892`).
The adopter-first repair note also undoes the local squash with a reset (`finalize_owner.go`, `adopterFirstRepair`).
In both, the reset parent already holds the work, so the next parent-side merge is up to date and a record exists.
The row then carries the discarded local merge's SHA, committer time and message.
D5's claim that a hand-reset parent "gets a new merge and a new record" is false for both.
The study this task serves reads those fields, so the wrong data is silent.

### [BLOCKING] D1's tracked `.gitignore` breaks PATTERN-fabric-boundary

**Location:** `_lym/discussion/decision-record.md#D1`

**Issue:** `pattern/PATTERN-fabric-boundary.md`, under Excludes, says exclusion is `.git/info/exclude` on both sides,
never a tracked `.gitignore`, and that the one weft-side `.gitignore` writer is `boardengine`'s in the Bolt checkout.
D1 creates a second one under the run's `_lym/`, written with a plain write by whichever module writes first.
`## PATTERN changes` declares no change to that entry.

This sits on a hunk the last fix did not change, which the incremental rule would bar.
The prompt says a constraint wins over anything it conflicts with and that every violation is BLOCKING,
so it is raised, and the conflict is stated here.

## Focus verdicts

- **D3:** the seed holds.
  Each reader the directive names reads the record before the report, so it reads the rewritten live path.
  The criterion's gap is listed below the bar.
- **D5:** F19 and F20.
- **D6:** not a defect.
  The exclusive create refuses a leftover or planted sibling, so no stale file mixes in
  and no write goes through a symlink.
  The design removes an empty `<dir>` before the rename, so it rests on no behaviour of renaming onto a directory.

## Below the incremental bar

- D3's criterion "after the live record and the live reports are deleted" runs on the same anchor,
  where the old and new report paths are identical.
  A seed that omits the `Report` rewrite passes it.
  Seeding from a tracked copy whose `Report` fields name another anchor's paths would cover the move.

## Focus departures

None.

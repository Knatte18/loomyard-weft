---
dispositions:
  - id: F24
    disposition: fixed-differently
    reason: The fix key's example of waiting out every code merge was not taken, because a parked conflict is when a run's Stuck transition matters most (MergeStateActive's doc, internal/fabricengine/mergestateactive.go:18-20). Records are committed through a temporary index and a HEAD move, which git accepts during a merge, and fabric's merge abort resets to the moved start. The transition waits only for a cherry-pick, revert or rebase, or for a merge fabric did not start whose index holds records unlike HEAD's. The tool fact, the D5 lock and crash statement and the acceptance row were added as proposed.
  - id: F25
    disposition: fixed
  - id: F26
    disposition: fixed
  - id: F27
    disposition: fixed
  - id: F28
    disposition: fixed
  - id: F29
    disposition: fixed
---

# Round 2 fixer report — fabric single-repo mode decision record

Every finding's premise held against the code; none is disputed or deferred.

## Changes

- **F24** (fixed differently; the reason is in the frontmatter), `_lym/discussion/decision-record.md`:
  - D4 gains a bullet: a records commit never skips while the worktree is mid-merge.
    - While `MERGE_HEAD` is present, and while the index holds the records paths as HEAD does, the commit is written through a temporary index, HEAD is moved onto it, and the records are staged.
    - The commit moves the merge record's `WarpStart`, so `fabricengine.Fabric.MergeAbort` resets to that commit and its conclude-landed guard does not refuse.
    - `MergeAbort` commits the records first.
    - A fabric merge that crashed before D5 step 3 is aborted by the next records commit.
    - `MergeStateActive` reports a cherry-pick, revert or rebase in progress, and a merge fabric did not start whose index holds records unlike HEAD's. The seam waits on those in place of its skip, and logs a way forward.
  - D5 drops recording the SHA in `.lym`. It now says steps 1 to 3 hold D4's write lock, that the merge record notes when step 3 is done, and how a crash before or after step 3 resumes.
  - The Constraints gain two tool facts:
    - the partial-commit refusal, during a merge, a cherry-pick or a rebase;
    - HEAD at commit time as the merge's first parent, with `git merge --abort` as `git reset --merge`.
  - The D4 acceptance row is extended.
- **F25**, decision record:
  - D7: `operator_unpushed` is read as owed by both `fabricengine.FailedPushesOwedBy` and `fabricengine.IsPushOwed`. `protected` is read as owed by neither.
  - D7: the transition seam completes a transition whose push fails with `protected` failures only, so the run acts with those records unpushed. The Bound says so.
  - D8: a failed view refresh is never returned in the pair push's error.
  - A `PATTERN-run-fsm` change was added under PATTERN changes, and the D7 and D8 acceptance rows were rewritten.
- **F26**, decision record:
  - D7's fabric-merge predicate is now git-only: the trailer, plus a tree equal to the first parent's records over the clean merge of the records-stripped parent trees.
  - D5 step 4 no longer writes to `.lym`.
  - The prime's branch lives in the git config key `lym.primeBranch`, written by `lym fabric clone`. When the key is absent, fabric refuses with a way forward.
  - The new constant is listed in Constraints, and the D7 acceptance rows were added.
- **F27**, decision record: D9's leaf helper takes the records paths or anchors as input and reads no git. Readers outside fabric get the paths from `fabricengine.RecordsPathsAt` (new) or from their caller.
- **F28**:
  - The first Open risks bullet was dropped.
  - The Scope Out pointer "(see Open risks)" now names the F16 added entry.
  - Support log Q6 was rewritten as proposed, and a Round 2 section records this round's picks.
- **F29**, decision record: D9's walk bullet admits merge commits fabric wrote, as D7 defines them. The D9 acceptance row was extended.

## Self-check

- fix-claims, BLOCKING: fabric's `MergeAbort` resets hard to `WarpStart`, and its conclude-landed guard refuses once HEAD has moved, so the in-merge records commit was lost or the abort was blocked. **Fixed with own change**:
  - D4's records commit moves `WarpStart` under the lock `MergeAbort` takes.
  - `MergeAbort` commits the records first.
  - A fabric merge that crashed before step 3 is aborted by the next records commit.
- fix-way-forward, BLOCKING: same as the fix-claims BLOCKING above; merged. **Fixed** by the same change.
- fix-way-forward, MEDIUM: concluding a merge that crashed before step 3 carries the source's records, and the waiting run cannot reach the merge verb that would abort it. **Fixed with own change**: such a merge is aborted by the next records commit, so nothing waits on it.
- fix-proposal, MEDIUM: the run deadlocks on a crash before step 3. Merged with the fix-way-forward MEDIUM. **Fixed** by the same change.
- fix-proposal, MEDIUM: cherry-pick and rebase also refuse a partial commit. **Fixed with own change**:
  - `MergeStateActive` reports them, and the seam waits, because aborting them drops a commit made during them.
  - The tool fact names all three messages.
  - The acceptance row covers a cherry-pick.
- fix-consistency, MEDIUM: D9's walk defined a fabric merge differently from D7. **Fixed**: D9 now points to D7's definition.
- fix-claims, LOW: the D5 crash sentence overstated what the index holds. **Fixed**: the paragraph was rewritten around the step-3 note.
- fix-consistency, LOW: the `PATTERN-run-fsm` change covered only the push refusal, not the guard refusal recorded as `protected`. **Fixed**: the exception is now keyed on the `protected` record.
- fix-consistency, LOW: the Constraints background list lacked `PATTERN-run-fsm`. **Fixed**: added.
- fix-proposal, LOW: a hand-made merge that copies the trailer passes the guard. **Fixed with own change**: second-parent commits are admitted only when some origin ref holds that parent.
- fix-proposal, LOW: squash landings fall outside the predicate. **Fixed**: the D7 Bound names them.

Every fork returned a report.

## Gate repair

- The `PATTERN-run-fsm` bullet under PATTERN changes is joined onto one line, the shape the gate parses,
  and shortened under the 280-character rule-text cap; which readers take which class as owed stays in D7.
- Each citation of the new config key `lym.primeBranch` that the gate flagged, in D7, Constraints and the D7 acceptance row, is marked ` (new)`.

## Disputed

None.

## Deferred

None.

## Rulings

None.

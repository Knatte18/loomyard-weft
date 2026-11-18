---
verdict: BLOCKING
prior:
  - id: F24
    status: resolved
    answer: "D4 now decides a records commit during a merge (temporary index, HEAD move, WarpStart move), pins git's refusal in the Constraints, and adds the D4 acceptance row and D5's lock statement. Reproduced with git 2.53.0 in a scratch repository: git commit -- path during a merge prints fatal: cannot do a partial commit during a merge.; a temporary-index commit with HEAD moved onto it during a merge concludes with the records commit kept. The crash-recovery part of the fix is the subject of F30 and F31."
  - id: F25
    status: resolved
    answer: "D7 and D8 now choose the behaviour and the PATTERN changes bullet for run-fsm words it as declared; statuscommit returns any push error IsPushOwed does not read as owed, so D7's protected exception and D8's rule that a view failure never enters the push error are the two cases that avoid the halt."
  - id: F26
    status: resolved
    answer: "D7's merge predicate is git-only, the prime's branch lives in the git config key lym.primeBranch with a stated refusal when absent, and an acceptance row covers a merge made before .lym was removed. Nothing the fix added reads .lym."
  - id: F27
    status: resolved
    answer: "D9 states the leaf helper reads no git and that a reader outside fabric takes a commit's records paths from fabricengine.RecordsPathsAt (new) or from its caller, which agrees with PATTERN-cwd-resolution."
  - id: F28
    status: resolved
    answer: "The Open risks bullet is dropped and Q6 reads resolved by the parent, naming the added entries."
  - id: F29
    status: resolved
    answer: "D9's walk bullet admits records-only commits and the merge commits fabric wrote by D7's stripped-tree test in place of the clean-merge tree test, and the D9 acceptance row covers a parent merge-in with conflicting records on both sides."
findings:
  - id: F30
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:113
    summary: "D4 has a records commit abort a merge that stopped before D5 step 3 through MergeAbort and then commit, but MergeAbort resets hard to WarpStart and the temporary-index route is refused there because the index holds the source's records; so the status file the transition just wrote, and any other live records edit since step 1, is reset away, the commit lands empty and the run acts past a transition whose records are lost; the record names no mechanism that keeps them, and its acceptance row only asserts the outcome"
    fix: "In D4's sentence on a merge fabric started that stopped before D5 step 3 (and D5's crash paragraph), state how the worktree's live records survive the abort, since resetMergeSides runs git reset --hard. For example, before the abort the records commit builds its commit from a temporary index seeded from HEAD with the worktree's content of every records path the source did not change, moves HEAD and the recorded WarpStart onto it as for a live merge, and only then aborts; for a records path the source changed, the worktree holds step 2's merge result and never a live edit, so the commit takes HEAD's version, and the record states that as the bound. Pick one such rule, state what it loses, and sharpen the D4 acceptance row to name a status file written by the transition surviving the abort and being pushed."
  - id: F31
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:148
    summary: "D5 says the next run of the merge verb aborts any merge in progress and starts again from step 1, citing mergeresolve; today only the resolver's Resolve aborts first, while Fabric.Merge and Fabric.MergeIn refuse with the already-in-progress guard (recheckMergePreconditionsUnderLock in internal/fabricengine/merge.go), so as written the sentence removes that guard for the verbs and lets a re-run discard a parked merge's hand-resolved conflicts through MergeAbort's hard reset, with no bound stated; it also contradicts D4, which keeps a parked merge alive for records commits to pass through"
    fix: "Replace D5's sentence beginning Either way with the recovery D4 already gives. A merge that stopped before step 3 is aborted by the next records commit or by lym fabric merge --abort. A merge past step 3 is parked, and a merge verb run again refuses as today, naming lym fabric merge --continue and --abort, and never aborts it. The resolver's own abort-first on a re-run stays as it is and is named as the only abort-first path."
---

### [BLOCKING] A records commit after a crashed merge loses the records it was to commit

**Location:** `_lym/discussion/decision-record.md:113`

**Issue:** D4 says a merge fabric started that stopped before D5 step 3 is aborted by the next records commit through `fabricengine.Fabric.MergeAbort`, which then commits.
In that state step 2 has written the source's records into the index and the worktree, so D4's temporary-index route, which runs only while the index holds every records path as HEAD does, is refused.
`MergeAbort` resets with `resetMergeSides`, which calls `repo.ResetHard` (`git reset --hard`, `internal/gitrepo/reset.go`), on a path request with `force: true`.
The transition seam writes the status file first and commits second, so the file the commit was to carry is a tracked worktree modification, and the hard reset restores it.
The abort leaves nothing to commit, the seam reads `committed == false` and the run acts, which breaks D4's own rule that a run never acts past a transition whose records commit has not landed.
D4's sentence that `MergeAbort` first commits the worktree's records the same way cannot apply here, since "the same way" is the route this state refuses.
The D4 acceptance row for this case states the outcome without a mechanism that can reach it.

### [BLOCKING] D5's crash recovery removes the already-in-progress guard of the merge verbs

**Location:** `_lym/discussion/decision-record.md:148`

**Issue:** D5 ends its crash paragraph with a rule that the next run of the merge verb aborts any merge in progress, as `mergeresolve` does today.
`mergeresolve.Resolve` aborts first because its caller re-invokes it verbatim after a killed session.
`Fabric.Merge`, `Fabric.MergeIn` and the beam verbs refuse today with `mergeReasonAlreadyInProgress` whenever fabric's merge record exists, and the same file refuses a foreign merge.
Read for every verb D5 lists, the sentence turns that refusal into an abort that resets a parked merge hard, so conflict resolutions an operator or a resolver session made are dropped.
D5 states neither what the abort skips nor what bounds it, which the discussion spec requires of a Decision that removes a guard, and D4 says the opposite of the sentence for a merge past step 3: records commits pass through it.

## Rejected

- Focus 1, a transition while the shared worktree holds a code merge, D5 steps 2 to 4 and a parked conflict: not a defect apart from F30.
  `MergeStateActive` is redefined by D4, steps 1 to 3 hold the write lock so no records commit sees the source's records, and a parked `MergeIn` leaves `MERGE_HEAD` over HEAD-equal records, which the temporary-index commit accepts.
  The record pins git's refusal in its Constraints with the message text, the doc of `failureDisposition` and `builtin/commit.c`; reproduced here with git 2.53.0.
- Parked squash conflicts: no gap.
  `Fabric.Merge` with a squash expects a clean merge and aborts on a conflict inside its own call, and only `MergeIn` parks, so a squash leaves no parked state for the WarpStart move to cover.
  Git accepts a path-limited commit over a conflicted squash index, as observed.
- Focus 2, a push error that `IsPushOwed` does not read as owed: `statuscommit.New` returns it and the run halts.
  D7 makes `protected` the one declared exception, as the `PATTERN-run-fsm` change words it, and D8 keeps a view failure out of the push error, so no other case reaches the halt.
  Both `IsPushOwed` and `FailedPushesOwedBy` allow-list their owed classes, so `protected` is unowed without further change and `operator_unpushed` needs adding to both, as D7 says.
  The record does not name the predicate `statuscommit.Deps` gains, which is plan detail and not raised.
- Focus 3, state only under `.lym`: none.
  The prime guard's predicates read git, the prime's branch is a git config key, and fabric's merge record is `fabric-merge.json` in the worktree's git directory, never under `.lym`.
  D7 states the refusal when the key is absent, and a lost merge record reads as a foreign merge that D4 waits on.
- Focus 4, readers outside fabric: D9's one `fabricengine.RecordsPathsAt` read, or a caller handing the paths in, agrees with `PATTERN-cwd-resolution`.
  `verifytree` and `fabricengine` import each other in neither direction.
- Focus 5, webster's `reconcileHead`: `parentMergeRejection` rejects a D5 merge-in today, since a clean merge of the parents conflicts on per-branch records.
  D9's replacement of the tree test with D7's stripped-tree test admits it, and the two-parent and second-parent checks stay.
  The stripped merge needs synthetic commits for `gitrepo.MergeTree`, which takes commits, and a base argument; that is plan detail.
- Focus 6, Open risks and the Question ledger: the one remaining risk, an operator rebasing the prime's branch, is no question that an added entry answers, and every ledger line is resolved.
- The seam's wait in place of its skip is worded without a layout, so it could change a two-repository hub; D4's own scope is the one-repository hub and Scope Out holds, so not raised at this round's severity.
- The pin for D4's merge-refusal fact cites a round-2 observation; the doc of `failureDisposition` and git's source also carry it, so the pin stands.

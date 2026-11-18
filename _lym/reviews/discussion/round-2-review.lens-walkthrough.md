```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case each.

**List: each decision, the case used, purpose met or not**
- **D1.** A one-repository task worktree with `.lym-anchor` at its root, `_bolt` missing. `lymcwd` reads the marker in the working tree, and fabric reads it from git. Met.
- **D2.** `--single-repo` against an origin that has no branches. The marker commit lands on the prime's branch and is pushed. Then `lym-bolt` is created and pushed, and a refused marker push removes the hub directory. Met.
- **D3.** `Add` forks from a parent that carries run records. The records commit deletes the runs root and writes the origin record. The marker comes along from the parent. Met.
- **D4.** A transition commit while an agent has code staged. The path-limited commit leaves the code staged, and `Warp-SHA` names HEAD from before the commit. Met.
- **D4, mid-merge case.** A transition commit while the worktree holds an unfinished code merge. The seam's `MergeStateActive` probe already skips the commit today for merge state on either side, so the one-repository layout changes nothing here. Met.
- **D5.** A local merge of a task into the prime's branch with conflicting records. Step 1 commits the receiver's records, step 2 merges without committing, step 3 restores the parent's records, step 4 commits with the trailer. The parent keeps its own records. Met.
- **D6.** A pair push followed by teardown. One push goes out, then the archive tag, then the branch delete. Met.
- **D7, replay.** Local records-only commits R1 and R2 on fork point F, and origin adds code commit O1. R1's `Warp-SHA` names F and stays. R2's names R1, which is being replayed, so it becomes F. Met.
- **D7, `protected` class.** A prime transition push refused as a protected branch. The record says the run is not held. The transition seam instead halts the run (finding 1). Not met.
- **D8.** A task merged in from its parent at P2, then a pull request against parent tip P3. The view is parented on P2 and carries the task's code with P2's records. GitHub's merge leaves the parent's records alone, and a republish with nothing changed reuses the same SHA. Met.
- **D8, failed view push.** The record says it "holds no run". The same seam disposition halts the run (part of finding 1). Not met.
- **D9, webster walk.** A parent merge-in made by D5's strip during a webster batch. webster refuses the batch (finding 2). Not met.
- **D9, held work.** After a GitHub merge of the view, the strip-first held-work check runs per the plan-writer note. Met.
- **D10, D11, D12.** A write under `_lym`, a board write on `lym-bolt`, and the `hubforge` build. Met.

**Findings**

1. **BLOCKING · design · `_lym/discussion/decision-record.md:185`**
   - **Summary:** D7's `protected` failed push "holds no run", but a push failure that is not owed halts the run at its transition, so every prime run halts at its first transition while the prime's branch is protected. D8's failed view push, which also "holds no run" (`decision-record.md:222`), halts the run the same way when it fails inside the transition's pair push.
   - **Walk:** The prime push is refused and recorded as `protected`. `IsPushOwed` returns false because `protected` is not one of transport, deferred or teardown_lock. The seam then returns the error, and "the run halts at the transition with the failure recorded". So the bound line 196 implies, that prime runs keep going with unpushed records, does not happen. If the record instead means the run should act anyway, that departs from `PATTERN-run-fsm`'s "a transition commits and pushes before the run acts", and the record declares no PATTERN change for it.
   - **Evidence:** `internal/statuscommit/statuscommit.go:73-78` and `:124-125`; `internal/fabricengine/pushanchored.go:40-56`.

2. **BLOCKING · design · `_lym/discussion/decision-record.md:241-242`**
   - **Summary:** D9 lets only records-only commits pass webster's walk. A parent merge-in made by D5 is a merge commit whose tree is not git's clean merge of its two parents, so webster refuses any batch with a merge-in during it.
   - **Walk:** `reconcileHead` accepts a merge commit only when `parentMergeRejection` finds that its tree equals a conflict-free `MergeTree` of its parents. D5's merge-in restores the task's records over the parent's. If the records conflict (the per-branch `_lym/fabric/origin.json`), `MergeTree` reports the merge as unclean. If they do not conflict, the clean tree holds the parent's records while the commit holds the task's. Either way the walk refuses. Today the same merge-in passes, because records never enter the code merge.
   - **Fix:** Have the walk's clean-merge comparison strip the records paths, the same way the held-work checks do.
   - **Evidence:** `internal/websterengine/gitwrap.go:132-139` and `:177-199`.

**Tool calls**
- `grep` for the `MergeActive:` wiring and the `MidMerge` definitions: the context lacked what the transition seam's mid-merge probe reads.
- `sed` of `internal/fabricengine/midmerge.go:1-60` and `internal/loomcli/wiring.go:210-232`: to check whether a merge on the code side already triggers the seam's skip today. It does, so I dropped the D4 mid-merge candidate.

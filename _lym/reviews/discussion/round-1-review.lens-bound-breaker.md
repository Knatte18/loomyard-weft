```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked each bound, guarantee and existing-code claim in the decision record and tried to build a concrete counterexample for it.

**Claims attacked**

1. D6 backstop "any run that takes the parent lock first writes a missing ledger landing": broken by findings 1 and 2.
2. D6 rationale "the parent is never ahead of its upstream by a commit that is not an owed landing tip": broken by finding 1.
3. D2 "an outcome outlives a driver restart so a resumed run still learns what happened to it", read together with D1's "writes its queue entry" at join: broken by finding 3.
4. D8 "`landed` with its landing commit on the parent: the run merges and gates nothing … ends Finalize": broken by finding 3.
5. D3 skips paused and moved-tip entries: broken by finding 4, because the check is made only when the batch is taken.
6. D5 "every task-head read on the batch path … resolves the task branch ref, never the worktree's HEAD": broken by finding 5.
7. D5 "Finalize's entry and resume restore a … detached [worktree]": broken by finding 6.
8. D7 "the empty prefix counts as passing": broken by finding 7.
9. D7 "after verifytree's one rerun": broken by finding 8.
10. D6 bound "a member's records commits from its join on are held back": broken by finding 9.
11. D9 "warned of as a retired key", and its D9 acceptance criterion: broken by finding 10.
12. D9 "two extra gates" and D7 "about log2(n) extra gates": broken by finding 11.
13. Constraint that a leader's gates and the gaps between bisection steps keep the lock: held. `finalizeKeepsLock` keeps the lock while the status says Finalize is running (`internal/loomcli/looplock.go:202`).
14. Constraint that `HoldsWork` reads a squash chain commit as holding the run's work: held (`internal/fabricengine/parenttips.go:225-246`). The merge base is the same as for today's squash landing.
15. D4 "merge-tree and commit-tree give today's landing shape": held for the tree. On a detached checkout, the code store's only hook (`post-checkout`) is warn-only, so it blocks nothing.
16. D10 bound "a GitHub rule the resolver's read misses ends in a direct push GitHub refuses": held as stated.

**Findings**

1. **MEDIUM**, design. Location: D6, backstop sentence.
   - The backstop runs only in "any run that takes the parent lock first". But two `lym fabric` verbs that are not runs take the same lock: `pull` (`internal/fabriccli/hubpull.go:33`) and `anchor` (`internal/fabriccli/anchordeps.go:254`, `acquireBranchLock` at `anchordeps.go:549-575`).
   - Counterexample: a leader moves the parent with A and B, then dies before it writes the ledger. Next, `lym fabric pull` takes the lock. It finds the parent ahead by commits that are not owed landing tips, and the backstop never runs.
   - Second counterexample: a host reboot kills every queued process. The D6 identification "whose `Lym-Landing` trailer names a queued entry" then depends on whether the next leader prunes dead entries without outcomes (D2) before it runs the backstop. The record sets no order.
   - Once A's entry is pruned, A's join-time records commit is gone. No ledger landing can be rebuilt, and `syncParent` stays stuck on "the local parent branch … is ahead of its upstream" (`internal/landingshed/finalize.go:749`).

2. **MEDIUM**, design. Location: D6 backstop with D8 "No outcome: the run leads a batch".
   - The backstop writes ledger landings but sets no outcome. A member that survives a leader's death after the move therefore takes the lock with no outcome and leads a batch over its own work, which the parent already holds.
   - D4's commit-tree always writes a new commit. Today, by contrast, `Fabric.Merge` classifies such a merge as already up to date and commits nothing (`internal/fabricengine/merge.go:444-449`).
   - The result is a second landing commit with an empty diff and a second trailer. Neither D3 nor D8 routes a parent that holds the work (`ParentHoldsTaskWork`) to the already-landed path before outcome dispatch.

3. **MEDIUM**, design. Location: D2 and D1 (join), D8.
   - The record never says whether a run rejoining at Finalize rewrites an existing entry or keeps its outcome, and both choices break a stated guarantee.
   - If a rewrite clears the outcome, a resumed run loses `failed`, `conflict` or `landed`. That breaks D2's rationale and D8's rule that a run keeps `failed` or `conflict`.
   - If the outcome is kept, the stale outcome applies to a new head. Counterexample: a run is marked `landed` and dies before reading the outcome. The operator runs `lym loom goto` to an earlier row and adds commits. On return to Finalize, D8's "`landed` with its landing commit on the parent" ends Finalize, and the new commits never land.
   - D8 compares only the landing commit, never the entry's task head against the branch. Today `ParentHoldsTaskWork` judges the current task head (`internal/fabricengine/replay.go:338-350`).

4. **MEDIUM**, design. Location: D3 skip rule, D6.
   - D3 checks pause, liveness and task tip only when the batch is taken. The lock is then held across up to three gates of 10 to 15 minutes each before the move.
   - Counterexample: after the take, `lym loom pause` covers member B, or a commit lands on B's task branch. D6 still moves the parent with B's old head and marks B `landed`. On waking, B ends Finalize under D8 without landing its newer commits, and a paused run has landed.
   - D6 re-reads only the parent's head before the move (D6 first paragraph). It does not re-read the members.

5. **MEDIUM**, consistency. Location: D5, the task-head sentence.
   - The rule's reach is undefined against existing readers of the worktree HEAD.
   - `ParentHoldsTaskWork` reads `pair.code.CurrentSHA()` (`internal/fabricengine/replay.go:338`). The pusher and `ownerChecks` call it.
   - `darnTreeScope` diffs from the merge base to the literal `"HEAD"` (`internal/loomcli/darnwiring.go:105`), and `verifytree` reads HEAD too.
   - Counterexample: a darn leader with a tree-scoped diff. If D5 is applied to the scope read, the batch gate covers only the leader's own trees and the other members' changes go untested. If it is not applied, D5's sentence is false as written.

6. **MEDIUM**, design. Location: D5, the restore rule.
   - The restore runs only at Finalize's entry and resume. Way forwards elsewhere in the code send the operator through `lym loom goto --to &lt;row&gt;`.
   - Counterexample: the leader dies while detached. The operator goes to an earlier row and resumes there, so no restore runs.
   - Fabric's merge-in and merge then refuse a detached HEAD (`internal/fabricengine/mergeguards.go:162-172`, called at `merge.go:162`). An agent session's commits on the detached HEAD are reachable from no ref.

7. **MEDIUM**, design. Location: D7, "the empty prefix counts as passing".
   - The parent's tip is not always a gated landing. Under the lock, `syncParent` fast-forwards the parent to its upstream (`internal/landingshed/finalize.go:752-772`). That brings in commits pushed to origin by the operator or another actor, which no gate here ever ran on.
   - Counterexample: a broken upstream commit arrives. Bisection blames entry 1. If entry 1 is the leader, it runs its fixer on a fault it did not make; otherwise that member gets `failed`. No bound is stated.

8. **MEDIUM**, design. Location: D7, "after verifytree's one rerun".
   - The rerun belongs to landingshed's `verifyGate.rerunTestFailure`, not verifytree. It reruns only test-run failures. A tier timeout, a build, vet or lint failure, and a log with no report are returned as they are (`internal/landingshed/verifygate.go:300-318`, `verifygate.go:307`).
   - A `TimedOut` result is a plain failure (`internal/verifytree/verify.go:463`). Only a runner crash or a memory stop is unjudged (`verify.go:120`).
   - Counterexample: a tier timeout caused by load or slot contention starts a bisection. It ends with `failed` on a run that did nothing wrong, with no rerun at all.

9. **LOW**, consistency. Location: D6 bound "records commits from its join on are held back".
   - Nothing holds a member's records between its join and the leader's ledger write. The hold comes only from a ledger `Landing` (`HeldRecordsCommit`, `internal/pushledger/ledger.go:239-261`, read by `recordsPushCeiling` in `internal/fabricengine/pushanchored.go`).
   - Counterexample: the member's pusher pushes the join-time records commit and later ones before the move. The leader then records a "held" commit that origin already holds, and `heldCeiling` takes its first parent (`internal/fabricengine/pusherreads.go:104-113`). The stated bound does not hold in that window.

10. **LOW**, consistency. Location: D9, and its D9 acceptance criterion.
    - In a default or worktree layer, a retired key is an unknown key. It gets the unknown-keys warning naming `lym config reconcile` (`internal/configengine/doc.go:74`, `config.go:325`), and the local and machine layers refuse it instead.
    - So the "retired-key warning" the criterion asserts does not exist.

11. **LOW**, consistency. Location: D9 rationale and D7's gate-count claim.
    - When the leader is the culprit, its chain commit for prefix 1 has the same tree as its own merged tree. Verifytree records passes only (`internal/verifytree/verify.go:375`), so "today's merge, gate and fixer alone" runs that known-failing tree again. That is two bisection gates plus one re-gate, three extra gates for a batch of four, against "two".
    - The support log's Q7 says three while D9 says two.

**Tool calls**

1. grep of production `parentlock` Acquire callers and `heldCeiling`: to find which processes besides Finalize take the parent lock, one hop past D6's backstop claim.
2. `anchordeps.go` `acquireBranchLock`, the `heldCeiling` definition and `mergeguards.go` detached guard: to confirm the lock taker that is not a run, the meaning of the records hold, and the guard a detached worktree meets.
3. Callers of `acquireBranchLock` and the `ParentHoldsTaskWork` wiring: to name the `lym fabric` verbs that take the lock and to see how holds-work is wired.
4. `ParentHoldsTaskWork` in `replay.go`: to see where it reads the task head, for D5's claim and D8's landed detection.
5. `parentHoldsTaskWorkAt` body: to confirm it reads `pair.code.CurrentSHA()`, the worktree HEAD.
6. The code store's `post-checkout` hook: to check whether D5's detached checkout trips a blocking hook. It is warn-only.

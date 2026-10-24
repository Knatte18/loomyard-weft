```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out (fix-proposal lens).

Proposals challenged: F1, F2, F3, F4, F5, F6, F7, F8, F9, F10, F11.
F2, F6, F7, F8 and F11: I could not build a case where the proposal is wrong.

**MEDIUM — F1, D11: "The start guard's refusal of a pair left from an earlier run … gives way for a project entry whose `beam` is true …"**
The exemption covers only the left-pair refusal, so the new D11 criterion ("passes batten's start guard and reaches Worktree-Create's skip") still fails whenever the prime has tracked dirt.
- In `internal/battencli/startguard.go:67-76`, the `SourceDirty` refusal runs before the `PairParts` refusal when `!found`, and the landing has no status file.
- A landing forks no pair, and D7 keeps the dirt refusal only "for a pair forked from the prime".
- So a beam whose parent is another beam is refused for dirt in the prime, which it never merges into.
- Its way forward ("commit or revert them in the source worktree") points at the prime, not at the parent Finalize actually merges into.
- Finalize already has its own Stuck for a dirty parent (D5).
- Suggested change: the exemption should cover the start guard's tracked-dirt refusal too.

**MEDIUM — F3, D4: "read as the prime only when that names the prime's worktree, and never derived by matching `ParentBranch` …"**
- `ParentWorktree` is the acting worktree's local directory name (`internal/fabricengine/add.go:441`, `ParentWorktree: l.WorktreeName`).
- `ResolveParent`'s prime test looks at the worktree on this machine (`isPrimeWorktree`, `internal/hubgeom/hubgeom.go:57`).
- Concrete case: a member is forked from the prime on machine A, then adopted onto machine B, where the prime's directory has a different name.
- On B its `ParentWorktree` names no worktree, so it is not read as the prime. It reads as a parent slug the board doesn't hold, and `Owners.Effective` answers "" for an unheld slug (`owners.go`).
- The entry then reads as owned by no machine instead of the prime's owner.
- Banning any use of `ParentBranch` throws away the one signal that doesn't depend on the machine: the prime's branch.
- Suggested change: read the parent as the prime when `ParentBranch` is the prime's branch or `ParentWorktree` names the prime's worktree.

**LOW — F5, D1: "an open project lists … under a layer of its own, `Project`, never Ready or a dependency letter"**
- Today a run-held entry gets the `Running` layer (`layer.go`, `runningLayer`), and D15 shows a member's "Running status".
- Under the proposal, a beam that is being landed (run-held, D11), or a sub-beam member that is being landed, lists as `Project`.
- That hides a live landing run from `lyx board list`, `get` and D15's standing.
- Suggested change: a run-held project keeps `Running`.

**LOW — F4, D8: "the guard refuses in a beam whose board entry is run-held by its landing, naming the landing"**
- The new refusal names the landing but not where to run the verb instead.
- An operator in the beam who wants to resume a halted landing (F6's "halts for the orch") or cancel it has no stated way forward.
- The guard's other refusal (D8, two lines up) names the nearest beam or the prime as where to run the verb.
- PATTERN-refusal-way-forward requires a way forward. This one should name the beam's parent worktree, or the prime.

**LOW — F9, D3 Bound: "elsewhere refuses it naming the owner and `lyx adopt <slug>`"**
- D9's refusal of a beam another machine owns names the owner, "`lyx beam create <slug>` run there, or `lyx adopt <slug>` … and then `lyx beam create <slug>` here".
- The rewritten bound, and its D3 criterion twin, list only the adopt route, so they disagree with D9's refusal text.
- Suggested wording: "elsewhere refuses it with D9's refusal of a beam another machine owns".

**LOW — F10, D13: "A beam whose worktree is absent on this machine sends the notice to the prime's orch"**
- The task's `parent_beam` is the beam, so D8's stencil filter ("the prime's orch those with an empty one") tells the prime's orch not to start it.
- Batten would refuse the start anyway, since the task reads as owned by the beam's owner.
- So the notice reaches an orch that can't act on it, while the stencil starts tasks a landing notice names (`orch-template-role.md:138-142`, cited in the review's F5).
- The two rules now give that orch conflicting instructions.
- Suggested change: send no notice in that case, or state that the prime's orch only reports it.

Tool calls:
- Grepped `isPrimeWorktree` in `internal/hubgeom` and the `ParentWorktree:` writes in `internal/fabricengine`, to learn how the prime and the recorded parent worktree are named. My context held only `decideParent`.

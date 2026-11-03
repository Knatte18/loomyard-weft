```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked each bound, guarantee and claim about existing code in the decision record. Five claims broke and one is a NIT.

**Claims attacked:**
- D1: adopt-only `Add` meets neither the non-empty-`Anchor` refusal nor `ErrSourceDirty`. Held (`add.go:239-246`, `add.go:397-401`).
- D1: "no parent is resolved". Held for `resolveAddParent`, but `Add` still reads the prime's HEAD and refuses a detached or unborn one (`add.go:351-361`). See F1.
- D1: the entry "is judged from its origin record and the Bolt". Broken, F4.
- D1 step 2: "prunes the slug's stale worktree registrations". Broken as worded, F6 (NIT).
- D1 bound: a pair whose records worktree directory still stands is refused. Held (`add.go:328-331`).
- D1 bound: a local weft left behind by a removal is re-adopted. Held (`remoteleftover.go:88-93`, `weftwiring.go:217-222`).
- D2: `ErrRemoteLeftover` under adopt-only is only the diverged-local-weft kind. Held (`remoteleftover.go:162-175`, sentinel before the warp probe).
- D2: "every other answer is transient" while the goal is that no pass fails forever. Broken, F1.
- D2 Land: pick the "newest" archive tag whose commit holds the child's `done` status. Broken, F5.
- D2: the re-run resumes from `failed` (`checkRunStatus`). Held (`arm.go:586`).
- D3: `childReachedParent` reads reached only at Worktree-Teardown or past the merge. Held (`cancel.go:691-708`).
- D3: a cancel "refuses as owned elsewhere", described as "this stop's adopter-first case". Broken, F7.
- D4: the branch push passes the same pause gate as the tag push. Held (`gitexec.go:226-238`, network gate on every remote subcommand).
- D4: the retry on "missing necessary objects" is a new retryable reason. Held as new (`pushretry.go:67` knows only "(failed)" and "cannot lock ref").
- Constraint: the archive-tag push takes no remote-refs lock. Held (`archive.go:127-131`).
- D6 bound: a branch the lease keeps "is another machine's live branch". Broken, F2.
- D7: only a detached push holds the teardown lock shared. Held, and that is the source of F2.
- D8 and the Constraint on the 2026-11-03 trigger: run A's tracking ref outlived its removal. Broken or unproven, F3.
- D8 bound: the deleted ref names a branch origin does not hold. Held, apart from a probe-to-delete race that does no harm.

**Findings:**

**F1 — MEDIUM, design, D2 "Every other answer is transient".**
Several refusals that never clear on their own sit in the transient bucket, so the pass fails every dobby pass with only the failing-pass notice. That is the board's bug 1 unchanged.
- **Counterexamples:**
  - The task branch is checked out in another worktree: `refuseAdoptLocalCheckedOut` runs under `AdoptLocal` (`add.go:341-345`).
  - The prime's weft has a move record: `moveRecordStop` (`add.go:259-266`). The record lists "the fork point's anchor-move stop" as transient by name.
  - The prime has an empty own marker (`add.go:256`), or is on a detached HEAD (`add.go:359-361`).
  - A stray directory sits at the task worktree path. `taskWorktreePresent` returns an error rather than "absent" (`wire.go:270-278`), so D1 never runs and the pre-run errors every pass. `Add` would refuse it too, with a plain error at `add.go:298-307`, which is outside the closed set.

**F2 — MEDIUM, design, D6 bound and `PATTERN-slug-remote-order` ("shared for a detached push").**
Only `DetachedPush` takes the teardown lock (`teardownlock.go:75-87`; nothing else calls `TryAcquireReadLock` on it per the grep). The synchronous `commit-records` push of the same pair (`weft_verbs.go`, the ordinary branch) takes none.
- **Counterexample:** an agent in the pair commits and pushes T2 between Remove's archive of local tip L and the leased delete.
  - The lease refuses, and the branch is kept on origin at T2.
  - The archive tag at L is an ancestor of T2, not a descendant, so T2 is uncovered (`remoteleftover.go:117-126`).
  - The next `Add` of the slug reads that leftover as live and adopts the old run's weft.
- The bound "under D7 it is no new pair of this slug on this machine; it is another machine's live branch" is therefore false for this machine's own synchronous push.

**F3 — MEDIUM, design, the 2026-11-03 Constraint and D8 rationale ("the tracking ref the reconcile judged was run A's, outliving run A's removal").**
- The cancel's remove ran with `Remote: true` (`cancel.go:636`). Its weft delete goes through `DeleteRemoteBranch` as `git push origin --delete <branch>` (`push.go:194`, called from `weftwiring.go:222-237`).
- A successful delete-push to a remote with a fetch refspec also deletes `refs/remotes/origin/<branch>`. A refspec must exist, because `originTargets` judges only branches that have tracking refs (`originreconcile.go:673-676`).
- So either the delete ran and dropped run A's ref, or it did not run and origin still held the branch. The latter contradicts the reconcile's "gone" verdict.
- The record gives no evidence, such as the ref's reflog, that the ref was run A's. D8's claim to cover the trigger rests on an unexplained link, which is a deferred research question.

**F4 — LOW, consistency, D1 ("its entry is judged from its origin record and the Bolt").**
The origin record lives in the absent records worktree. `ReadOriginFor` stats it and returns `found=false` (`origin.go:123-128`). With the pair absent, nothing is read from an origin record and the parent branch drops out of the first reconcile's scope (`originreconcile.go:627-637`).

**F5 — LOW, design, D2 Land way forward ("the newest of the slug's `archive/<slug>/*` tags").**
- Archive tags are lightweight (`git tag <tag> <tip>`, `archive.go:120`) and their names carry only a SHA, so "newest" has no defined order.
- D4 now pushes two tags per removal, and branch names carry no run id.
- When two machines drove the same run (the adopt case), both the local tip and the diverged origin tip can hold the child at `done`. The filter does not pick one.

**F6 — NIT, consistency, D1 step 2 ("prunes the slug's stale worktree registrations").**
`git worktree prune`, as `removeSibling` uses it (`remove.go:320`, `remove.go:516`), is repo-wide and cannot be scoped to one slug.

**F7 — LOW, consistency, D3 ("refuses as owned elsewhere when the local board shows another owner, which is this stop's adopter-first case").**
The adopter-first repair is reached only after `ownerChecks` let this machine land (`finalize_owner.go:41-75`, which halts on a run or parent owned elsewhere). So the local board shows this machine as owner, and the cancel would not refuse as owned elsewhere. D3's choice still stands on the owner-pause lift (`cancelcmd.go:475-496`), but the second reason is wrong.

**Tool calls:**
1. `ReadOriginFor` (`origin.go`) and `DeleteRemoteBranch`/`DeleteRemoteBranchLeased` (`push.go`). My context lacked how an absent pair's origin record is read and the exact delete-push command.
2. `NamedWorktree` (`lymcwd.go`), the lock text of the `ReconcileOrigin` body, and the `TeardownSlug:` setters. I needed the absent-pair anchor read and which pushes carry a teardown slug.
3. The `weft_verbs.go` push verb and every taker of the teardown lock. I needed to confirm that only `DetachedPush` takes the lock and the synchronous push does not.

```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case.

**Findings**

1. **BLOCKING · D8 with Finalize's pull-request path (D5, D7): a landing by pull request ends with a push the parent's rules refuse, and ancestry says the run never landed.**
   - Case: task `foo` gets PR head `lym-view/foo` = commit V. V's parent is the merge base M, and its tree is foo's code plus M's records.
   - GitHub merges V into `main`.
   - Finalize then runs its local `Merge(foo)`, as the exploration describes: it "runs against a parent that already holds the work".
   - `main` now holds V, but not foo's tip. foo's tip is not an ancestor of V's merge. So D5's rule ("never fast-forward; make a merge commit") makes a new local merge commit of foo into `main`.
   - `PushBranch` then pushes that commit to `main`. A parent whose GitHub rules require pull requests refuses the push, and those parents are the reason D8 exists (support log, Rejected alternatives).
   - If the local merge is skipped instead, foo's tip is never an ancestor of `main`. Every "parent holds the run's work" or `IsAncestor` reader then reads the run as not landed. Those readers include the owner-machine reconcile, `loomshed`'s verify gate, `landingdeps`' `RefSHA` check and teardown's landed classification.
   - Either way the purpose ("same as today from outside" for PR parents) is missed. The record says nothing about how Finalize's post-GitHub step works in one-repository mode.

2. **MEDIUM · D5: a merge into a receiver with uncommitted records is refused before the strip runs.**
   - Case: the prime has an uncommitted edit to `_lym/config/gate.yaml`, and task `foo` also changed that file.
   - `git merge --no-commit foo` refuses with "local changes would be overwritten" before D5's restore step ever runs.
   - Today the weft is outside the merge, so dirty receiver records never block a landing.
   - `parentRecordsCommitter` commits only the parent's runs root, and `syncSideBeforeMerge` (`merge.go` ~558) commits nothing.
   - D5 needs the receiver's records committed, or set aside, before the merge.

3. **MEDIUM · D4: records commits and agents' git now share one index, and the record does not say how they coexist.**
   - Case: an agent has staged `src/x.go` in a task worktree, or is in the middle of `git commit`, when a transition's records commit runs.
   - With one worktree and one index, the agent's operation and fabric's commit contend for `index.lock`, so one of them fails.
   - A records commit made by add-then-commit, rather than a path-limited commit, also sweeps the agent's staged code into a commit carrying the `Run-Transition` trailer.
   - Today the weft has its own index, so neither can happen.
   - Within one worktree, `Fabric.Commit`'s split also turns into two commits on one branch under two different locks, the weft write lock and the warp side's lock.

4. **MEDIUM · D8: a push that refreshes the view has no base to build it from.**
   - "Fabric's push of a branch that has a local view ref refreshes the view": the push verb is handed a branch and `SyncOptions`, not a base.
   - The local view ref records no base either, so the merge-base step has no input.
   - The record should name the source, for example `parent_branch` in `_lym/fabric/origin.json`, or store the base with the view.

5. **MEDIUM · D7's bound ("fabric never pushes ... an operator commit") breaks on a local landing.**
   - Case: the operator has an unpushed commit C on `main` in the prime, and a task lands locally into `main`.
   - Finalize's `PushBranch` pushes `main`, and C goes with it. D7's guard covers only the prime records push.
   - Either narrow the bound to the records push, or give the landing push the same guard.

6. **LOW · D7: an owed landing merge on the prime blocks pull for good.**
   - A landing merge that fabric made but has not pushed is neither records-only nor operator-made.
   - D7's replay "stops ... when any other local commit is unpushed", so once origin moves, a diverged prime whose landing push is owed never replays.
   - D7 also does not say how fabric recognises "a landing merge fabric made", for example by a trailer.

7. **LOW · D4: the `Warp-SHA` trailer's meaning is ambiguous in one repository.**
   - "Naming the prior code tip" can mean HEAD, which may itself be a records commit, or the last commit that changed code.
   - `OriginCodeTip` and the correspondence index (`index.go`) read that trailer.
   - D4's acceptance criterion inherits the same ambiguity.

8. **LOW · D1: the predicate reads "two repositories" during clone, before `_bolt` exists.**
   - In D2's fresh path, a `lymcwd` resolve that runs before Bolt is added falls back to the sibling-marker rule and finds no sibling.
   - D2 should order Bolt's creation before the first anchor resolve, or say that clone passes the layout in explicitly.

9. **LOW · D1 and the PATTERN changes: a new leaf package would change `PATTERN-cwd-resolution`.**
   - If the predicate's "leaf package both may import" is a new package, `PATTERN-cwd-resolution`'s closed import list for `lymcwd` changes. `## PATTERN changes` does not list that.
   - Naming `internal/dotgit` avoids the change.

**Decisions walked**

| Decision | Case | Purpose |
|---|---|---|
| D1 | `_bolt` gitfile common dir against the prime's `.git` | met, except during clone (finding 8) |
| D2 | `--single-repo` on an empty origin, then a second machine cloning the same URL | met |
| D3 | `Add foo` from `main`: the worktree checks out main's `_lym` and gets its own origin record and marker as a records commit | met; this matches today's fork from the parent's records |
| D4 | Transition commit while an agent has staged code | not met (findings 3 and 7) |
| D5 | Local merge where both sides edited `_lym/config` and the task added `_lym/shed/foo` | met in a clean tree, not met with dirty receiver records (finding 2) |
| D6 | Push, refused push, then teardown with archive tag and branch and view deletion | met |
| D7 | Operator commit C under records commit R in the prime | met for the records push, bound broken by the landing push (findings 5 and 6) |
| D8 | PR-Review opens on `lym-view/foo`, a fixer push, then the GitHub merge | not met at landing (findings 1 and 4) |
| D9 | Gate green on code tree T, then records commit R: the stripped tree is unchanged, so green stays valid | met |
| D10 | A write to `<root>/_lym/shed/x`, checked against the records paths first | met |
| D11 | Board write commits on `lym-bolt`, then a detached sync push | met |
| D12 | `hubforge` single-repo builder | met |

Tool calls: one read of `internal/fabricengine/merge.go` ~540-600 (`syncSideBeforeMerge`). My context held only the file's package comment, and I needed to know whether the merge commits the receiver's dirty side first; it does not.

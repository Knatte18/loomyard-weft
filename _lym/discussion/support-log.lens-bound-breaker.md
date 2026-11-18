```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: I tested the record's claims, bounds and ordering promises against concrete states and interleavings.

**Findings**

1. **BLOCKING — D5 / D9: the receiving worktree's live records make every fabric merge, and `FastForwardPair`, refuse.**
   - The merge verbs check for dirtiness before they merge: `pairDirtyReason` in `internal/fabricengine/merge.go`, around lines 83, 178 and 388.
   - `FastForwardPair` refuses a dirty code worktree with `FastForwardDirty`, in `parenttips.go`.
   - In single-repo mode the receiving worktree's `_lym` is tracked and is written all the time, by shed status, batten and config. Its dirt is ordinary code-tree dirt.
   - Counterexample: during Finalize, batten has written the parent's `_lym` and not committed it yet. Merging the task into the parent then refuses as dirty.
   - D9 covers only readers outside fabric. The record does not say whether fabric's own dirtiness guards ignore records paths, or whether the merge commits the receiver's records first.

2. **BLOCKING — D5: restoring "from its own HEAD" either destroys live records or leaves the task's records in the parent.**
   - Suppose the restore is a worktree checkout of the records paths from HEAD. It then overwrites the receiver's uncommitted records.
   - Suppose instead it touches only the index. Then any file that exists only in the task, such as `_lym/discussion/decision-record.md`, stays in the parent's working `_lym` as untracked content. The parent's tools would read it as the parent's own record.
   - `git merge` also refuses outright when an uncommitted receiver file overlaps a path the source changed ("local changes would be overwritten").
   - The record needs an order: commit or set aside the receiver's records, merge, then strip both the index and the worktree.

3. **BLOCKING — D8 / D9: "Ancestry checks stay as they are" does not hold for landing's "parent holds the work" checks.**
   - Finalize's resume logic depends on `ParentHoldsTaskWork`, `OriginParentHoldsWork` and `HoldsWork`, which ask whether merging the source into the base adds nothing (`internal/landingshed/deps.go` around 115–133 and 335, `finalize.go`, `finalize_owner.go`, `finalize_land.go`, `parentalign.go:393`).
   - In single-repo mode, merging the task into the parent always adds the task's records, so these checks never report "held".
   - After a pull request merges the D8 view, the task tip is not an ancestor of the parent.
   - So Finalize's local `Merge(TaskBranch)` after the GitHub merge re-merges the work, and the owner-machine rule "an ended entry whose parent already holds the run's work goes on" never fires.
   - Neither D8 nor D9 covers these checks.

4. **MEDIUM — D8: refreshing the view needs its base, and the record stores none.**
   - The record says fabric's push of a branch that has a local view ref refreshes it. But the view's merge base needs the base branch, and the push path has no base argument.
   - The record does not say where the base is kept, or what happens when the parent moves.

5. **MEDIUM — D8: GitHub may delete the head branch once the pull request merges.**
   - With the "delete head branch" setting on, GitHub removes `lym-view/<branch>` after the merge.
   - D6's teardown delete and the refresh-on-push must both tolerate a missing view ref on origin. The record does not say they do.

6. **MEDIUM — D9's bound "a missed reader … never passes changed code" is broken in the other direction.**
   - A missed reader that takes "HEAD moved since X" to mean "the implementer committed work", such as webster's card checks (`CurrentSHA`, `CommitParents` in `websterengine/gitwrap.go`), would accept a records-only commit from `webstercli`'s sync as code work.
   - That passes unchanged code as done. The stated bound covers only the safe direction.

7. **MEDIUM — D7: a "landing merge fabric made" cannot be told apart from an operator's merge, and landing already pushes operator commits.**
   - The record names no marker, trailer or ref that identifies fabric's landing merges.
   - Finalize's `PushBranch` on the parent already pushes the operator's unpushed commits on `main`, today and under D5. The guard D7 adds to the records push does not bind the landing push.
   - So the bound "fabric never pushes an operator commit" is false.

8. **MEDIUM — D2: the `--single-repo` refusal cannot recognise the warp of an existing two-repository hub.**
   - A warp origin has neither `lym-bolt` nor `.lym-warp`, which lives on the weft. So `--single-repo` creates `lym-bolt` in the code repo of a live two-repository hub.
   - After that, any machine that clones that URL alone gets the one-repository layout.

9. **MEDIUM — D1: the predicate misreads a hub during clone, or a one-repository hub whose `_bolt` is missing or broken.**
   - Before `_bolt` is added, or after it is lost, a one-repository hub reads as two. `lymcwd` then looks for a weft sibling that does not exist.
   - The anchor marker step in `CloneHub` runs after the prime is cloned, so the clone order matters, and the record does not fix it.

10. **LOW — D3: the weft-side exclude lines now apply to every code worktree.**
    - `seedWeftArtifactExcludes` (`recordsgit.go:99`) writes the weft lock dir, `.lym/` and the legacy lock-file names into the shared `info/exclude`.
    - `.lym/` unanchored is intended. But the lock-dir and legacy lock names would now hide matching paths in the code tree.
    - The record checks only the opposite direction: no line may hide a path the Bolt branch tracks.

11. **LOW — D6: the push locks.** Today's records-then-code push takes the weft and warp push locks in turn. In one repository both may resolve to the same path, so a non-reentrant lock would deadlock if the plan keeps the two-step order. The record does not say that the two locks become one.

12. **LOW — D4 / D7: replayed records commits keep a stale trailer.** A records commit replayed onto a new origin tip keeps a `Warp-SHA` trailer naming the old parent, not its new one.

**Claims I attacked**

- D1, the layout predicate: broken (#9).
- D2, the `--single-repo` refusal: broken (#8).
- D3, exclude sharing: one direction holds, the other is a gap (#10).
- D4, the bound that mixed agent commits are tolerated: holds for stripping, but feeds #6.
- D5, the strip resolves conflicts before code handling: broken (#1, #2).
- D5, a fast-forward to its own origin is unchanged: broken by records dirt (#1).
- D6, the push order holds trivially: holds. Lock note #11. Teardown delete: #5.
- D6, `OriginCodeTip`: holds.
- D7, fabric never pushes an operator commit: broken (#7).
- D7, the replay bound: holds, with a trailer note (#12).
- D8, a three-way merge of the view leaves the base's records unchanged: holds. Refresh and landing follow-up: broken (#3, #4, #5).
- D9, ancestry checks are unaffected: broken (#3).
- D9, the miss bound: broken (#6).
- D10, the classification order: holds.
- D11, the Bolt `.gitignore`: holds.

**Tool calls**

- Read `parenttips.go`'s `FastForwardPair` docs and `seedWeftArtifactExcludes`: my context did not show what `FastForwardPair` does or which lines the weft exclude adds.
- Grepped the landing "holds work" checks and the merge dirtiness guards: my context did not show how landing judges that the parent holds the task's work, or how merge treats dirt.

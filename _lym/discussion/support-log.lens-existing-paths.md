```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already write, read or reapply the state each decision changes, and checked whether each decision's result survives them.

**Findings**

1. **BLOCKING — D5, D9 (and D6, D8): `Fabric.HoldsWork` breaks once records are stripped at merge.**
   - `HoldsWork` (`internal/fabricengine/parenttips.go` ~224) merges the task into the base with `MergeTree` and asks whether the result's tree equals the base's tree.
   - After a D5 strip-merge, the task branch still carries task-only `_lym` files the parent lacks. So the merged tree never equals the parent's, and `HoldsWork` reads false forever.
   - Its readers then misread a landed run:
     - `ParentHoldsTaskWork` / `parentHoldsTaskWorkAt` (`replay.go` ~314-355).
     - `OriginParentHoldsTaskWork` (`originreads.go:24`), which the owner-machine catch-up uses ("an ended entry whose parent already holds the run's work goes on").
     - Finalize's `syncParent` resume guard (`landingshed/finalize.go` ~510-522).
     - `archiveHeldParentMerge` / push-rejected handling (`finalize.go` ~720-830).
     - battenshed's `innerrun.go` `ParentHoldsTaskWork`.
   - Effect: a Finalize resumed after its merge re-merges, and catch-up pauses an ended run whose work landed.
   - D9 covers only readers outside fabric and says "ancestry checks stay". `HoldsWork` is tree-based and inside fabric, and the record says nothing about it. It must compare code trees, with records paths stripped on both sides.

2. **BLOCKING — D8: after a GitHub merge of the view, the local landing merge pushes a new commit to the parent.**
   - Finalize's pull-request path merges on GitHub first (`finalize_pr.go` ~448), then runs the local `parentHandle.Merge(TaskBranch)` (`finalize.go` ~445) and `PushBranch`.
   - The task tip is not an ancestor of the view merge, and `HoldsWork` reads false (finding 1). So the local merge makes a new merge commit and pushes it straight to the parent branch.
   - A parent whose GitHub rules require pull requests rejects that push. Otherwise the parent gets an extra merge commit.
   - `finalize_pr.go` ~333 also compares `status.HeadSHA != head` and calls `CommitsMissing(status.HeadSHA, head)`. With the view as the pull request's head, `HeadSHA` is the view commit, which never equals the task tip and is no descendant of it. That would read as "commits missing" on every check.
   - D8 needs to say what local Finalize does after a view merge (adopt origin's parent, no local merge), and how the head-SHA check maps the view to the task tip.

3. **BLOCKING — D7 (and D4): `replayWeft` would rebase the task's code branch.**
   - `replayWeft` (`replay.go:448`) replays every local commit origin lacks onto upstream, and sets aside commits under an ended run's record directory. It is called from three places:
     - `originreconcilemoves.go:259`: the origin reconcile of a pair's records branch.
     - `pull.go:426`.
     - `pushclassified.go:770`: the replay after a rejected push.
   - In the one-repository layout the "records branch" is the code branch. These paths would rebase the task's code commits, and could tag-and-drop commits as ended-run records.
   - D7 limits replay to records-only commits for the prime alone. It says nothing about the reconcile, pull and push-rejection paths on task and beam branches. `PATTERN-landing` / warp never rewrite a code branch.
   - Each caller needs a stated one-repository behaviour, for example: replay only records-only commits, otherwise stop or hand over.

4. **MEDIUM — D5: restoring the receiving side's records "from its own HEAD" throws away live uncommitted records.**
   - The parent or task worktree holds uncommitted records during Finalize, such as shed status written continuously and the parent pair's runs root.
   - Two things go wrong:
     - `git merge --no-commit` refuses outright when an incoming records change touches a dirty records file ("local changes would be overwritten"). Shared per-branch files on both branches, like `_lym/fabric/origin.json` and `_lym/config/*`, make that likely on a merge-in from the parent.
     - Checking records out from HEAD overwrites dirty records.
   - Today a warp merge never touches the weft working tree.
   - D5 needs either "commit records before the merge" (as `parentRecordsCommitter` does for the parent only) or "restore from the pre-merge working state", plus a merge that excludes the records paths before git reads them.

5. **MEDIUM — D4: fabric and agents now share one index.**
   - `StageAndCommit` (`gitrepo.go` ~135-165) runs `git add -- files`, then `git commit -m msg -- files`. Those are `--only` semantics, so code an agent has staged is not swept in. That part survives.
   - But records commits (transition commits, the detached sync, webster sync) now run on the same worktree index as the agent's `git add` and `git commit`. Concurrent runs hit `index.lock` contention that today's separate weft index never sees.
   - The record says nothing about it. A transition commit failing on `index.lock` becomes a new failure class, so it should be retried or held under a lock.

6. **MEDIUM — D3: the reconcile and repair paths would act on real directories.**
   - `reconcile.go` (`CauseJunctionMissing`), `lym fabric reconcile`, and `WireJunctionsWith` / `adoptDotLymContent` (`junction.go`) treat a real `_lym` or `.lym` as a missing junction. They move real content into a weft target.
   - `seedGitExclude` writes `/<anchor>/_lym` lines, and `seedWeftArtifactExcludes` writes to the weft repo's `info/exclude`. In one repository that file is shared with the code side.
   - D3 says "no junction is made, repaired or unwired", but names neither the reconcile cause detection nor the weft-side exclude seeding. Both run on every `add`, `clone` and `reconcile`, and must branch on D1.

7. **LOW — D4, D6: weft-to-warp mapping goes vague in one repository.**
   - The `Warp-SHA` correspondence index (`index.go`), the anchor-change stop (`anchorchange.go` / `anchorstop.go`, which blocks pull and push across an anchor change) and `integrateweft.go` all assume two histories.
   - D5 says an anchor change is "a records commit" but not whether the anchor stop still applies.

8. **LOW — D11: the shared `info/exclude` and hub workspace files.**
   - `hubworkspace.go` may list `-weft` folders. The dobby orphan-pair sweep may key on siblings. Neither is addressed, but both are cosmetic or detectable.

**Paths traced, per decision**
- **D1:** `lymcwd` sibling read: survives with D3's marker rule.
- **D3:**
  - `Add`, junction wiring and exclude seeding: break (finding 6).
  - `.scratch` link through `fslink`: survives.
  - Heddle exclude lines: survive.
- **D4:**
  - `StageAndCommit` `--only`: survives.
  - Shared index: breaks (finding 5).
  - `RecordsWorktree` / `RecordsRepoRoot` readers (`weavercli`, `battencli` catch-up, `reportengine` tags): survive.
- **D5:**
  - `Fabric.Merge` with or without squash: survives if fixed (finding 4).
  - `HoldsWork` / `ParentHoldsTaskWork`: break (finding 1).
  - `FastForwardPair` (to its own upstream): survives.
- **D6:**
  - `PushPairAnchored` single push and the failed-push record: survive.
  - Rejected-push replay at `pushclassified.go:770`: breaks (finding 3).
  - Teardown archive tag: survives.
- **D7:**
  - Prime replay: covered.
  - Task and beam replay via the reconcile and pull: break (finding 3).
- **D8:**
  - Finalize's local merge after the GitHub merge: breaks (finding 2).
  - The `HeadSHA` check: breaks (finding 2).
- **D9:**
  - `verifytree` `headTree` / `DirtyPaths` and the gate green `headIdentity`: covered.
  - Fabric-internal `HoldsWork`: missed (finding 1).
- **D10, D11:** no breaking path found.

**Tool calls**
1. Grepped `recordsgit.go` / `commit.go` / `finalize*.go` / `replay.go` to find the commit primitive, the held-work checks and the replay entry. My context lacked these call sites.
2. Read `finalize.go` ~505-530 and `replay.go` ~440-470, plus the `replayWeft` callers, for the resume guard and the replay scope.
3. Grepped the `ParentHoldsTaskWork` / `HoldsWork` definitions to find the held-work predicate.
4. Read `replay.go` ~314-360, `parenttips.go` ~215-240 and `commit.go` ~90-110, to confirm `HoldsWork` is tree-based.
5. Grepped `commitWeftLocked` to see how records are staged.
6. Read `gitrepo.go` ~130-175 to confirm that `StageAndCommit` commits with an `--only` pathspec.

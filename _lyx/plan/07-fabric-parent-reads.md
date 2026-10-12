# Card 7 — fabric-parent-reads

**Create:**
- `plan:internal/fabricengine#ParentRelation` -> `type ParentRelation string`
- `plan:internal/fabricengine#ParentEqual` -> `const ParentEqual ParentRelation = "equal"`
- `plan:internal/fabricengine#ParentBehind` -> `const ParentBehind ParentRelation = "behind"`
- `plan:internal/fabricengine#ParentAhead` -> `const ParentAhead ParentRelation = "ahead"`
- `plan:internal/fabricengine#ParentDiverged` -> `const ParentDiverged ParentRelation = "diverged"`
- `plan:internal/fabricengine#ParentNoUpstream` -> `const ParentNoUpstream ParentRelation = "no-upstream"`
- `plan:internal/fabricengine#ParentTips` -> `type ParentTips struct`
- `plan:internal/fabricengine#Fabric.ReadParentTips` -> `func (f *Fabric) ReadParentTips(branch string, fetch bool) (ParentTips, error)`
- `plan:internal/fabricengine#Fabric.ChangedPaths` -> `func (f *Fabric) ChangedPaths(from, to string) ([]string, error)`
- `plan:internal/fabricengine#Fabric.HoldsWork` -> `func (f *Fabric) HoldsWork(base, source string) (bool, error)`
- `plan:internal/fabricengine#FastForwardRefusal` -> `type FastForwardRefusal string`
- `plan:internal/fabricengine#FastForwardDirty` -> `const FastForwardDirty FastForwardRefusal = "dirty"`
- `plan:internal/fabricengine#FastForwardMergeInProgress` -> `const FastForwardMergeInProgress FastForwardRefusal = "merge-in-progress"`
- `plan:internal/fabricengine#Fabric.FastForwardPair` -> `func (f *Fabric) FastForwardPair() (FastForwardRefusal, error)`
- `internal/fabricengine/parenttips_integration_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/gitrepo#Repo.Fetch`
- `internal/gitrepo#Repo.Upstream`
- `internal/gitrepo#Repo.IsAncestor`
- `internal/gitrepo#Repo.MergeTree`
- `internal/gitrepo#Repo.CommitTree`
- `internal/fabricengine#Fabric.Pull`

**Intent:**
Add the three code-side reads Merge-In and the moved-parent check need, in `fabricengine`, the one package that runs git on a pair (`PATTERN-fabric-git`, `PATTERN-gitexec-checked-call`).
Each reads the task pair's code repository, which shares its object store and remote-tracking refs with every worktree of the hub.

- `ReadParentTips(branch, fetch)` fetches the code repository's remotes first when `fetch` is true, then returns `ParentTips{Local, Upstream, Relation}`: `Local` is `refs/heads/<branch>`, `Upstream` the remote-tracking ref the branch is configured to track.
  With no upstream configured, or none fetched, `Upstream` is empty and `Relation` is `ParentNoUpstream`.
  Otherwise `Relation` is `ParentEqual`, `ParentBehind` (local is an ancestor of upstream, a fast-forward), `ParentAhead` (upstream is an ancestor of local) or `ParentDiverged`.
  A failed fetch is an error naming the reproducing command; a missing local branch is an error naming it.
- `ChangedPaths(from, to)` returns the sorted repo-relative paths `git diff --name-only from to` reports, the read `ReadConfigChanges` already makes with no pathspec.
- `HoldsWork(base, source)` reports whether merging `source` into `base` adds nothing: a clean `git merge-tree --write-tree` result whose tree equals `base`'s tree.
  A conflicted merge-tree holds nothing.
  It is the "parent already contains the task's work" test (D9) and holds after a squash landing, where ancestry does not.
- `FastForwardPair()` pulls the pair through `Pull` and maps its two refusals to neutral names a caller outside the fabric vocabulary's owner set may spell: a dirty code worktree returns `FastForwardDirty` and a fabric merge in progress returns `FastForwardMergeInProgress`, each with a nil error and nothing moved.
  Any other failure is returned as the error.
  Merge-In calls it on the parent pair only after it has seen the parent behind its upstream, so the pull is a fast-forward (D1).

`parenttips_integration_test.go` builds a hub with `hubforge` and asserts one table over `ReadParentTips`: equal, behind after a push to the upstream from another clone, ahead after a local commit, diverged after both, and no-upstream for a local-only branch, the push to the upstream seen only with `fetch` true.
It asserts `ChangedPaths` names exactly the files a commit changed, and `HoldsWork` true after the source's change was squash-committed onto the base and false before.
It asserts `FastForwardPair` moves a behind pair to its upstream tip, and returns `FastForwardDirty` with the pair unmoved when a tracked file is modified.

**ImpactSummary:** three new read-only `Fabric` methods and one pull wrapper; no existing fabric verb changes.

**Commit:** 7: fabricengine: parent tips, changed paths and the holds-work test

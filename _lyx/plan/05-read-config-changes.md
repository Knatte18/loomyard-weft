# Card 5 — read-config-changes

**Create:**
- `plan:internal/fabricengine#ConfigChanges` -> `type ConfigChanges struct`
- `plan:internal/fabricengine#ReadConfigChanges` -> `func ReadConfigChanges(l *lyxcwd.Location, taskBranch, parentBranch string, configRels []string) (ConfigChanges, error)`
- `internal/fabricengine/configchanges.go`
- `internal/fabricengine/configchanges_integration_test.go`

**Edit:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#WeftBranchName`
- `internal/fabricengine#WeftRepoRoot`
- `internal/fabricengine#ScopedPathspec`

**Intent:**
D4's computation half: the fabricengine function that reads a task's changes to its per-worktree config files, so git on the weft stays in fabricengine (`PATTERN-fabric-git`).

`ConfigChanges` (in the new `internal/fabricengine/configchanges.go`) carries `Files []string` (repository-relative, slash-separated paths, sorted), `Base string` (the merge-base SHA) and `Tip string` (the task side's tip SHA); every field documented.
`ReadConfigChanges` runs read-only git through `gitexec.Run` in the weft repo root:
1. resolve `WeftBranchName(taskBranch)` and `WeftBranchName(parentBranch)` to SHAs (`rev-parse --verify refs/heads/<b>`);
2. `merge-base` of the two;
3. `diff --name-only <base> <tip> -- <pathspec>`, where the pathspec is `ScopedPathspec(l.AnchorRel, configRels)` in slash form.
`configRels` is told by the caller (the per-worktree config files, anchor-relative); an empty `configRels` returns an empty `ConfigChanges` with no git call.
Every error is wrapped naming both branches it compares, so a caller that cannot read the diff can still tell the hub which two branches to diff by hand.
It writes nothing.
Landing never merges this side, so the merge-base is always the task's fork point and repeated calls return the same answer until the task side moves.

The function takes the file list rather than reading `configreg` because `configreg` imports `fabricengine`; the HubWide filter therefore lives with the caller (card 6).

Docs: `internal/fabricengine/doc.go` gains a short paragraph on `ReadConfigChanges`: what it diffs, that it is read-only, and that its caller supplies the file set.

Tests, `internal/fabricengine/configchanges_integration_test.go` (`integration` tag, `hubforge` hub with a pair forked from the prime): one table-driven test whose rows commit on the pair's weft branch, then call `ReadConfigChanges(l, slug, primeBranch, rels)`:
- a change to an anchored `loom.yaml` config file listed in `rels` is reported, with `Base` equal to the fork point and `Tip` equal to the branch tip;
- a change to a config file not in `rels` is not reported;
- no change reports empty `Files`;
- a commit on the parent's side after the fork does not appear;
- a parent branch whose weft branch does not exist returns an error naming both branches;
- a subpath-anchored hub (`hubforge.NewHub(t, "sub")`) scopes the pathspec under the anchor.

**ImpactSummary:** a new read-only fabricengine function and its doc paragraph; no existing caller changes.

**Commit:** 5: fabric: read a task's per-worktree config changes since its fork point

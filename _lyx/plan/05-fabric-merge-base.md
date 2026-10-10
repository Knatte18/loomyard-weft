# Card 5 — fabric-merge-base

**Create:**
- `plan:internal/gitrepo#Repo.MergeBase` -> `func (r *Repo) MergeBase(a, b string) (string, error)`
- `plan:internal/fabricengine#Fabric.MergeBase` -> `func (f *Fabric) MergeBase(ref string) (string, error)`

**Edit:**
- `internal/gitrepo/ancestry_test.go`
- `internal/gitrepo/merge_integration_test.go`
- `internal/fabricengine/mergesiblings_integration_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** The darn verify gate lints the comments from the task branch's merge base with the parent branch, and no fabric or gitrepo read answers that today.
`Repo.MergeBase` resolves `a` and `b` as commits and returns the SHA of their best common ancestor through go-git's merge-base over `readGoGit`, keeping `PATTERN-gitrepo-client-boundary` (a local read goes through go-git) and `PATTERN-gogit-read-helper` (only `readGoGit` calls `goGit()`);
it validates both arguments the way `IsAncestor` does, and two commits with no common ancestor are an error naming both.
`Fabric.MergeBase` returns the merge base of the code checkout's HEAD and `ref`, the one spelling loomcli is allowed under `PATTERN-fabric-git`.
Tests: `ancestry_test.go` gains the invalid-argument rows for `MergeBase`, the gitrepo merge integration test gains a step asserting the merge base of two diverged branches is their fork commit, and the fabric merge-siblings integration test gains a step asserting `Fabric.MergeBase` of the parent branch equals the pair's fork commit.

**ImpactSummary:** Two new read methods and their test rows; no existing symbol changes.

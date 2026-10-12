# Card 2 — fabric-commit-log

**Create:**
- `plan:internal/fabricengine#Fabric.CommitLog` -> `func (f *Fabric) CommitLog(from, to string) ([]string, error)`
- `plan:internal/fabricengine_test#TestCommitLog_ListsTheRangeNewestFirst` -> `func TestCommitLog_ListsTheRangeNewestFirst(t *testing.T)`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the read `lyx loom peers` needs for D6's parent-commit list: the commits reachable from `to` and not from `from` in the pair's code repository, each rendered as `<short sha> <subject>`, newest first.

`Fabric.CommitLog` sits in `parenttips.go` beside `Fabric.ChangedPaths`.
It resolves both refs through `f.code.ResolveSHA`, lists the range through `f.code.CommitsNotIn(to, from)` and reads each commit's subject, the first line of `f.code.CommitDetail(sha).Message`.
The short SHA is the first seven characters.
All of this goes through `internal/gitrepo`, in process, so no raw git runs (`PATTERN-fabric-git`, `PATTERN-gitrepo-client-boundary`).
An empty range returns an empty slice, and an unresolvable ref is an error naming it.

`TestCommitLog_ListsTheRangeNewestFirst` goes in `parenttips_integration_test.go` (package `fabricengine_test`, `integration` tag) on `buildReconcileFixture`, as `TestChangedPaths_NamesExactlyTheCommittedFiles` does.
It makes two commits after a base and asserts the two lines, newest first, each a seven-character SHA prefix and the subject, plus an empty result for an empty range and an error naming an unknown ref.
`test-budget.yaml` raises `internal/fabricengine` by one.

**ImpactSummary:** One read-only method added to `Fabric`; no existing member changes and no caller exists before card 3.

# Card 5 — fabric-origin-run-reads

**Create:**
- `plan:internal/fabricengine#Topology.TaskRecordsFile` -> `func (t *Topology) TaskRecordsFile(l *lymcwd.Location, slug, anchoredRel string) ([]byte, bool, error)`
- `plan:internal/fabricengine#Topology.OriginParentHoldsTaskWork` -> `func (t *Topology) OriginParentHoldsTaskWork(l *lymcwd.Location, slug string) (bool, error)`
- `plan:internal/fabricengine_test#TestTaskRecordsFile` -> `func TestTaskRecordsFile(t *testing.T)`
- `plan:internal/fabricengine_test#TestOriginParentHoldsTaskWork` -> `func TestOriginParentHoldsTaskWork(t *testing.T)`

**Uses:**
- `internal/fabricengine/doc.go`
- `internal/fabricengine#RecordsBranchFile`
- `internal/fabricengine#RecordsBranchName`
- `internal/fabricengine#Topology.TaskBranchOnOrigin`
- `internal/fabricengine#Fabric.HoldsWork`
- `internal/fabricengine#ParentHoldsTaskWork`
- `internal/fabricengine#ReadOriginFor`

**Intent:**
Add the two git reads D1's run reader needs on a machine with or without the run's pair, inside fabric as `PATTERN-fabric-boundary` requires.

`TaskRecordsFile` reads `anchoredRel` from the records branch of slug's task branch, through `RecordsBranchFile` over the records branch name of the task branch the topology resolves for slug.
It reads the local branch when one exists, else origin's copy, as `RecordsBranchFile` does; found is false when the branch or the file is absent.
It is how a machine with no pair for the run reads the child run record and the pair's origin record from origin's branch.

`OriginParentHoldsTaskWork` reports whether origin's parent branch, as this machine last fetched it, holds the work of origin's task branch.
It reads the parent branch from the pair's origin record, through `TaskRecordsFile` when the pair is absent, and the two remote-tracking refs from the prime's code repository,
then applies the holds-work test `Fabric.HoldsWork` applies; it makes no network call.
The local parent branch never decides it, since a merge there reaches origin only with the parent push.
A task or parent branch origin does not hold reads false; a missing origin record is an error naming the slug.

`TestTaskRecordsFile` reads a file from a local records branch and from origin's copy with the local branch gone, and reads found false for an absent file.
`TestOriginParentHoldsTaskWork` covers a fetched parent that holds the task's work, a local parent merge not yet pushed reading false, and a pair absent locally.
Both are tier 1 integration tests over hermetic git from `gitkit` or `hubforge`.

**Commit:** 5: fabric reads a run's task records and origin's parent-side merge with or without the pair

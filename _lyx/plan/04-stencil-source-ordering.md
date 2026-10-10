# Card 4 — stencil-source-ordering

**Edit:**
- `internal/fabricengine#StencilSource` -> `func StencilSource(worktreePath, sourceDir string, running buildvcs.Identity) stencilstore.Source`
- `internal/fabricengine/warpforward_integration_test.go`
- `cmd/lyx#seedStencilsAt`
- `internal/stencilcli#Command`

**Uses:**
- `internal/gitrepo#Repo.SHAExists`
- `internal/gitrepo#Repo.IsAncestor`
- `internal/stencilstore#Writer`
- `internal/stencilstore#Ordering`

**Intent:** D1's ordering read, told to the store through its `Source`.
`StencilSource` takes the running binary's whole identity: `Writer` is its revision and commit time whenever the revision is non-empty, and `Older` answers for a recorded writer by ancestry when the worktree at `worktreePath` holds both commits (`SHAExists` on each, then `IsAncestor(recorded, running)`, `RecordedOlder` only when the recorded revision is a strict ancestor), by commit time when it lacks either (`RecordedOlder` when the recorded time is strictly before the running time, both non-zero), and `OrderingUnknown` otherwise or on a read error, which is logged once.
An empty `sourceDir` no longer returns the zero `Source`: it returns one with no `Dir` and no `Build`, so the drift warning stays silent, but with `Writer` and `Older` set, which is how the specs pass gets the same ordering answer.
`Build` keeps its memoized `HeadContains` read for a non-empty `sourceDir` and revision.
Both callers pass `buildvcs.Running()` and the specs pass in `seedStencilsAt` and in `Command`'s sync switches from the zero `Source` to `StencilSource(worktree, "", running)`.
`TestStencilSource_BuildAncestryFollowsTheRevision` gains rows over a hubforge hub with two commits: recorded ancestor of running, recorded equal to running, recorded descendant, a recorded commit absent from the worktree with an earlier time, with a later time and with a zero time, and the empty-source-dir shape now carrying `Writer` and `Older`.

**ImpactSummary:** Re-signs one exported constructor with two production callers and one test, all listed; the ordering answer only changes a refresh decision for a stencil that already carries writer keys, which none does before card 3's production binaries run.

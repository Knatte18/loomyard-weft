# Card 4 — batten-child-ignores-prime-records

**Create:**
- `internal/battencli/recordleak_integration_test.go`

**Uses:**
- `internal/fabricengine#Topology.Add`
- `internal/battencli#wireForHub`
- `internal/battencli#seedEntryStatus`
- `internal/battencli#taskWorktreeLocation`

**Intent:**
Pin, end to end through batten's own rows, that a task pair no longer inherits prime's committed batten records — the reported failure `task worktree already seeded with a disagreeing seed: run self is already seeded with {Recipe:batten ...}; refusing to overwrite with {Recipe:loom ...}`.
Both tests fail on the code before card 2 and pass after it.

The file is `//go:build integration`, white-box `package battencli`, on a `hubforge` hub, for the reasons `lifecycle_integration_test.go`'s header gives.
Its fixture writes and commits, on prime's own pair through `fabricengine.CommitAnchoredPaths`, a batten seed (`shedrun.RecipeBatten`) and a status for slug `B` — the prime-side batten run — plus a batten seed for another slug `C`, using `shedrun`'s `SeedRel`/`StatusRel` for the paths.

- `TestBattenIntegration_SeedChild_IgnoresPrimesCommittedBattenRecords`: with a Board task `B` of type `loom`, drive `Worktree-Create` and `Seed-Child` through `battenrecipe.New(...).Step`; `Seed-Child` ends `Done`, the child's own seed (`shedrun.ReadSeed` at the child's location) names recipe `loom`, and the child has no `shedrun.RunDir(childLocation, "C")` on disk.
- `TestBattenIntegration_RealReadStatus_IgnoresPrimesCommittedStatusForTheSameSlug`: after `Worktree-Create` for `B`, the unstubbed `Env.InnerRun.ResolveStatus` and `Env.InnerRun.ReadStatus` report found == false with a nil error, although prime tracks a batten status for `B`.

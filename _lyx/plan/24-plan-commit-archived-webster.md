# Card 24 — plan-commit-archived-webster

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli/wiring_commitstatus_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/loomshed#ArchivedPlanWebsterDirs`
- `internal/websterengine#DirRel`

**Intent:** The Webster run record card 23's rotation moves lands in the plan commit, deletions and archive together (D9's commit half).
Webster's durable directory is tracked (`CommitWebster` commits `websterengine.DirRel()`), so moving the record out of it leaves deletions the plan-only `CommitPlan` pathspec never stages, while the Plan-Review and Plan-Bouncer rows run in between.
`CommitPlan` in `loomCLI.wire` commits `planCommitPathspec(location)` instead, a new unexported function beside `loomCLI.wire` in `wiring.go`, part of its change, that mirrors `reworkCommitPathspec`: the plan directory, plus `websterengine.DirRel()` when webster's directory holds a file or any directory `loomshed.ArchivedPlanWebsterDirs` returns holds one, so the moved-from deletions and the archive land in one commit; otherwise the plan directory alone, since git refuses a pathspec matching nothing.
Bound: the pathspec widens only to webster's own durable directory, and only when a record is there or was archived under the plan; the commit stages nothing else.
`TestPlanCommitPathspec` in `wiring_commitstatus_test.go`, beside the status pathspec test, is a table over a temp anchor: no webster dir (plan directory only), a webster dir holding a file, and a record moved into `archive-<stamp>/webster` with the webster dir empty (both paths); `test-budget.yaml` raises `internal/loomcli` by one for it.

**ImpactSummary:** Widens the CommitPlan pathspec in loomCLI.wire to webster's directory when a record is present or archived; every other commit seam is unchanged.

# Card 48 — fork-plan-write

**Edit:**
- `internal/websterengine/audit.go`
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/audit_test.go`
- `internal/websterengine/auditseverity_test.go`
- `internal/websterengine/recordbatch_test.go`

**Intent:**
Fix PR #321 round-3 finding B's first item: `CheckFork` raises no finding for a fork that writes into the plan directory, so a fork can rewrite a later card, or its own card's Verify, and nothing records it.
`ClassifyViolation` already classes a Master write under `PlanDir` as correctness;
a fork write there gets the same class.

In `audit.go`:

- A new class `ClassForkPlanWrite AuditViolationClass = "fork-plan-write"`, documented beside `ClassForkContractWrite`: a fork's own transcript wrote under the run's plan directory, which only webster itself rewrites.
- `CheckFork` gains a `planDirs []string` parameter after `workdir`: the told plan directory and its link-resolved form, so a write spelled through the `_lyx` link and one spelled through its target both match while `CheckFork` stays free of filesystem reads.
  Each fork write whose `resolveWritePath` result lies within one of `planDirs` (`pathWithin` over cleaned paths) yields one `ClassForkPlanWrite` finding with `Path` set to the write and `Key` from `forkKey` with a per-class ordinal.
  A write that is also a contract write keeps only its `ClassForkContractWrite` finding.
  The doc comment names the fourth banned write.
- A new unexported `planDirSpellings(geom Geometry) ([]string, error)` returns `geom.PlanDir` and its `canonicalPath`, deduplicated; the error is `canonicalPath`'s.
- `ClassifyViolation` classes `ClassForkPlanWrite` as correctness, beside `ClassForkContractWrite`.

`recordbatch.go` and `runlevel.go`'s `runExitAuditCrossCheck` compute `planDirSpellings` once, before any state mutation, and pass it to every `CheckFork` call.
A plan-write finding carries its `Path`, so record-batch's existing suspect-path collection names it in the failed batch's digest with no further change.

`doc.go`'s audit-findings paragraph adds a fork plan write to the correctness examples.

Tests:

- `audit_test.go`: update every `CheckFork` call to pass `planDirs`.
  `TestCheckFork` gains rows: an absolute write to `<planDir>/03-x.md`, a relative `_lyx/plan/03-x.md` resolved against the workdir, and a write through the second spelling each yield exactly one `fork-plan-write` with that `Path`;
  a write to a worktree source file yields none;
  two plan writes in one transcript get distinct Keys.
- `auditseverity_test.go`: `TestClassifyViolation` gains a `fork-plan-write` row, correctness.
- `recordbatch_test.go`: `TestRecordBatch_ForkPlanWriteFailsBatch`: an OK report whose fork transcript wrote the plan's card file → `ErrBatchFailed`, the report archived, the digest's reasons name the card file as a suspect path, and `RecoverSpawnOrAttach` then spawns.

**ImpactSummary:** CheckFork takes the plan directory's spellings and reports a fork plan write as a correctness finding, which fails its batch or holds run exit; every other CheckFork verdict is unchanged.

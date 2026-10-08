# Card 8 — computed-merriam-base

**Create:**
- `plan:internal/websterengine#MerriamBaseOf` -> `func MerriamBaseOf(texts ...string) batcher.StartBase`
- `internal/websterengine/merriambase.go`
- `internal/websterengine/merriambase_test.go`

**Edit:**
- `internal/websterengine#formBatches`
- `internal/websterengine#batchCards`
- `internal/websterengine#ExecutionBatches`
- `internal/websterengine#Run`
- `internal/websterengine#RebaselineDeps`
- `internal/websterengine#rebaselineBatches`
- `internal/websterengine/doc.go`
- `internal/websterengine/partition_test.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/runlevel_test.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/webstercli#websterCLI.executionBatches`
- `internal/webstercli#websterCLI.rebaselineCmd`

**Uses:**
- `plan:internal/batcher#StartBase`
- `internal/websterengine#MasterTemplate`
- `internal/websterengine#Geometry`

**Intent:** The batchifier prices Merriam's start from what Merriam loads instead of a fitted constant (D13's webster half).
`merriambase.go` holds:
- `MerriamBaseOf`, pure: the summed line count of the given texts (a text's lines counted as `batcher.DiskSizes` counts a file's) as `StartBase.Lines`, and the Go constant below as `StartBase.Fixed`.
- An unexported constant for the system prompt and tools: 20600, the measured Merriam session start, minus the computed size (lines × the template's `context_per_line` of 12) of this worktree's `CLAUDE.md` and the Master stencil with its inlined PATTERN overview as they stand when this card runs, so the computed base reproduces the measured start over that tree; its comment names that derivation, the tree it was taken at and the line counts it subtracted.
Cards 15 and 19 add Master ladder rungs after this card, so at landing the computed base sits those few stencil lines above the measured start; the constant is not re-derived for them, the decision record's auto-mode assumption leaving such drift to the `orientation` refit.
- `MerriamBase(geom Geometry) (batcher.StartBase, error)`: `MerriamBaseOf` over `CLAUDE.md` and, when present, `CLAUDE.local.md` at `geom.WorktreeRoot`, the Master template read with `MasterTemplate(geom.StencilsDir)` together with the orchestrator PATTERN directive `pattern.Directive(geom.RepoRoot, geom.StencilsDir, pattern.RoleOrchestrator)` renders (the stencil as rendered, minus the batch table and progress markers the partition has not formed yet), and `00-overview.md` at `geom.PlanDir`, the one plan file Merriam reads up front.
  An absent `CLAUDE.md`/`CLAUDE.local.md` adds nothing; any other read error is returned.
`formBatches`, `batchCards` and `ExecutionBatches` take the `batcher.StartBase` and pass it to `Batch`.
`Run` computes it with `MerriamBase(deps.Geom)` only where it forms a new partition (first init and `--fresh`), `RebaselineDeps` gains a `Base` field that `rebaselineBatches` passes when it batches the tail, the CLI's `rebaselineCmd` computes it from `c.geom`, and the CLI's `executionBatches` computes it only when `st` is nil (validate's first-init partition) and passes the zero `StartBase` otherwise, since `ExecutionBatches` maps a recorded partition's batches without reading the base; a computing failure refuses as transient naming the file.
A recorded partition keeps its recorded estimates and is never regrouped by the base (`PATTERN-batcher-registry`); the identity batchifier ignores it.
`merriambase_test.go` holds one top-level test, `TestMerriamBase`, for which `test-budget.yaml` raises `internal/websterengine` by one; its steps pin `MerriamBaseOf`'s line sum and constant and that `MerriamBase` grows with `00-overview.md` and with `CLAUDE.md` over a temp worktree; `partition_test.go` and `rebaseline_test.go` assert the base reaches `Batch` (the `beforeBatcher`-style fakes record it); `runlevel_test.go`'s fakes gain the parameter.
The package doc states where the base comes from and that it enters only at partition time.

**ImpactSummary:** Threads one value through webster's three batch-forming functions and their callers in Run, Rebaseline and the CLI; recorded partitions are untouched.

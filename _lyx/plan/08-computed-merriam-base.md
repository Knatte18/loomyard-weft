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
- `internal/webstercli#websterCLI.executionBatches`
- `internal/webstercli#websterCLI.rebaselineCmd`

**Uses:**
- `plan:internal/batcher#StartBase`
- `internal/websterengine#MasterTemplate`
- `internal/websterengine#Geometry`

**Intent:** The batchifier prices Merriam's start from what Merriam loads instead of a fitted constant (D13's webster half).
`merriambase.go` holds:
- `MerriamBaseOf`, pure: the summed line count of the given texts (a text's lines counted as `batcher.DiskSizes` counts a file's) as `StartBase.Lines`, and the Go constant below as `StartBase.Fixed`.
- An unexported constant for the system prompt and tools: 20600, the measured Merriam session start, minus the computed size (lines × the template's `context_per_line` of 12) of this worktree's `CLAUDE.md` and the Master stencil with its inlined PATTERN overview at this task's HEAD, so the computed base reproduces the measured start; its comment names that derivation and the line counts it subtracted.
- `MerriamBase(geom Geometry) (batcher.StartBase, error)`: `MerriamBaseOf` over `CLAUDE.md` and, when present, `CLAUDE.local.md` at `geom.WorktreeRoot`, the Master template read with `MasterTemplate(geom.StencilsDir)` together with the orchestrator PATTERN directive `pattern.Directive(geom.RepoRoot, geom.StencilsDir, pattern.RoleOrchestrator)` renders (the stencil as rendered, minus the batch table and progress markers the partition has not formed yet), and `00-overview.md` at `geom.PlanDir`, the one plan file Merriam reads up front.
  An absent `CLAUDE.md`/`CLAUDE.local.md` adds nothing; any other read error is returned.
`formBatches`, `batchCards` and `ExecutionBatches` take the `batcher.StartBase` and pass it to `Batch`.
`Run` computes it with `MerriamBase(deps.Geom)` only where it forms a new partition (first init and `--fresh`), `RebaselineDeps` gains a `Base` field that `rebaselineBatches` passes when it batches the tail, and the CLI's `executionBatches` and `rebaselineCmd` compute it from `c.geom`; a computing failure refuses as transient naming the file.
A recorded partition keeps its recorded estimates and is never regrouped by the base (`PATTERN-batcher-registry`); the identity batchifier ignores it.
`merriambase_test.go` pins `MerriamBaseOf`'s line sum and constant and that `MerriamBase` grows with `00-overview.md` and with `CLAUDE.md` over a temp worktree; `partition_test.go` and `rebaseline_test.go` assert the base reaches `Batch` (the `beforeBatcher`-style fakes record it); `runlevel_test.go`'s fakes gain the parameter.
The package doc states where the base comes from and that it enters only at partition time.

**ImpactSummary:** Threads one value through webster's three batch-forming functions and their callers in Run, Rebaseline and the CLI; recorded partitions are untouched.

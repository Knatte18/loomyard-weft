# Card 18 — recovery-uncommitted-paths

**Edit:**
- `internal/websterengine#RenderRecoveryPrompt` -> `func RenderRecoveryPrompt(batch batcher.Batch, cardGates, prevDigest, failureDigest, uncommittedPaths, reportPath, repoRoot, planDir, promptWorktreeRoot, stencilsDir, specsDir string, selfFixCap int, notePath, parentName string) ([]byte, error)`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine/runwrites.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/template_test.go`
- `internal/websterengine/runwrites_test.go`
- `internal/websterengine/recoverbatch_git_test.go`
- `contracts/stencils/webster/webster-prefix-recovery.md`
- `contracts/stencils/claims_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/websterengine#loadRunWrites`
- `plan:internal/websterengine#UncommittedPaths`

**Intent:** A recovery strand is told what uncommitted work it starts from and what to do with each path, instead of judging a dirty tree on its own (D5).
When `recoverSpawn` spawns a strand it lists the worktree's uncommitted paths, tracked changes and untracked files alike (`UncommittedPaths(deps.Geom)`, card 13, the list the reset reports), and classes each by the run's write evidence: a path some session of this run wrote successfully, by the transcripts `loadRunWrites` reads (a new helper in `runwrites.go` returns those writes as worktree-relative slash paths, untracked files included, unlike `ownTrackedPaths`), or a path no session of this run wrote.
`RenderRecoveryPrompt` takes the rendered block and fills a new optional marker, `{{.uncommitted_paths}}`, in `webster-prefix-recovery.md`, rendering `none` on a clean tree:
- under "written by this run": the earlier attempt's unfinished work; keep it as the starting point, check it against the card before building on it, and apply the suspect rule to any path the failure digest names;
- under "not written by this run": leave it untouched, and never stage or commit it.
The stencil gains a `## What the worktree holds` section carrying the marker and those two instructions, its comment lists the new optional marker, and the "You continue from the committed tree" sentence is rewritten to say the strand continues from the committed tree plus the uncommitted work listed there, so it no longer contradicts a dirty tree.
Bound: the classing reads the same transcript evidence the reset's own-path exemption reads; a path the evidence cannot attribute is "not written by this run", the side that touches nothing.
`template_test.go` adds the marker to the recovery prefix's optional set and a render row to the existing recovery render test asserting both classes and `none`; `TestLoadRunWrites` in `runwrites_test.go` gains a step pinning the helper; `recoverbatch_git_test.go` gains `TestRecoverBatch_UncommittedPathsPrompt`, which spawns over a real dirty worktree with one run-written and one foreign path and asserts the rendered prompt, and `test-budget.yaml` raises `internal/websterengine` by one for it; `claims_test.go` pins the rewritten sentence and the leave-untouched instruction.
The package doc's recovery section names the block.

**ImpactSummary:** Adds one parameter to RenderRecoveryPrompt, whose one production caller is recoverSpawn, and one optional marker to the recovery prefix stencil.

# Card 15 — batch-gate

**Create:**
- `internal/websterengine#renderBatchGate`

**Edit:**
- `internal/websterengine#RenderForkPrompt` -> `func RenderForkPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, reportPath, planDir, promptWorktreeRoot, stencilsDir, specsDir, outcomePath, summaryPath string, selfFixCap int, notePath string) ([]byte, error)`
- `internal/websterengine#RenderRecoveryPrompt` -> `func RenderRecoveryPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, failureDigest, uncommittedPaths, reportPath, repoRoot, planDir, promptWorktreeRoot, stencilsDir, specsDir string, selfFixCap int, notePath, parentName string) ([]byte, error)`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine/template_test.go`
- `internal/websterengine/cardgate_test.go`
- `internal/websterengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils/claims_test.go`
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `internal/websterengine#gateScopeOf`
- `internal/websterengine#gateScope.steps`
- `internal/gatecli#SlotBusyExit`

**Intent:**
Move tier 2 to one pass per batch (D2) and tell each fork beforehand what it may run (D8).
`renderBatchGate` renders the batch's package list (the union of its cards' surviving directories, per module) and the batch gate, `gateScopeOf(plan, cards, worktree).steps("integration")`, one step per line; with no surviving directory it renders a line saying the batch has no Go package and no batch gate.
`RenderForkPrompt` and `RenderRecoveryPrompt` take it and fill the implementer body's new required `batch_gate` marker; `BeginBatch` and `recoverSpawn` render it beside the card gates.
The implementer body states, in its card loop and a new step after the last card:

- Each card-gate step and each line of a card's `**Verify:**` runs as its own Bash call with `run_in_background`, whose exit the fork waits for, so neither a slot wait nor the run is killed by the Bash tool's timeout.
- After the last card's commit and that card's own Verify, before writing the report, the fork runs the batch gate once, each step the same way; a failure counts as a gate failure under the self-fix cap and makes the report `FAILED`.
- A slot-busy exit (`lyx gate test` exiting with the code `SlotBusyExit` defines, spelled as its number in the stencil, and a JSON error naming the holders) from any of these is not a failure: it costs no self-fix attempt, never makes the report `FAILED`, and the fork re-runs the same step.
- The fork may run raw `go test` on the batch's listed packages only; a module-wide or `tmux`-tier run goes through `lyx gate test`, never a raw `go` command.

Go never runs the batch gate, as with the card gate; Webster's plan-level verify gate catches a fork that skips it.
`template_test.go`'s fork and recovery cases list `batch_gate` as required and its render calls pass a placeholder batch gate.
`cardgate_test.go` gains a table-driven `renderBatchGate` test: a two-card batch's union across the root and a nested module, and a batch with no Go package; the `internal/websterengine` budget row rises by one for it.
`internal/websterengine/doc.go`'s sentence that there is no batch-wide verify distinct from the cards' own gates is rewritten to the batch gate: run once by the fork after the last card, over the batch's packages with `-tags integration`, never by Go.
Its fork-return sentence that a fork returns FAILED only on a non-zero build/unit gate or per-card `verify:` is restated: a fork returns FAILED on a failed card gate, card `**Verify:**` or batch gate, never on deviation alone and never on a slot-busy exit.
`claims_test.go`'s implementer claims follow the reworded card-gate step and pin the background-call, batch-gate and slot-busy sentences.
`loom-plan-spec.md`'s tier 2 states the batch gate.

**ImpactSummary:** Both implementer prompt renderers gain one parameter, with one production caller each plus the template test; every fork prompt gains the batch gate and the run rules.

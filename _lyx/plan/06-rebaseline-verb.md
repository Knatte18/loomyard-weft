# Card 6 — rebaseline-verb

**Edit:**
- `internal/webstercli/cli.go`
- `internal/webstercli/verbs_test.go`
- `cmd/lyx/helptree_test.go`

**Create:**
- `internal/webstercli/rebaseline.go`

**Uses:**
- `internal/websterengine/rebaseline.go`

**Intent:**
Expose D10 as `lyx webster rebaseline`, the way forward the fingerprint refusals now name.

`rebaseline.go` builds `rebaselineCmd()` on `websterCLI`, following `record-batch`'s verb shape:
`clihelp.ShouldAbort` first;
parse the plan from `c.geom.PlanDir`, sequence `c.batcher.Batch(plan.Cards)` through `websterengine.SequenceBatches`;
hold `websterengine.AcquireStateMutation` across load → `websterengine.Rebaseline` → `SaveState`, and release before the fabric sync;
a missing state.json refuses with the existing `no run in progress; run "lyx webster run" first` text;
fabric-sync `_lyx/webster` through `fabricSync` with the label `rebaseline`, reporting a sync failure the way `record-batch` does.
The success envelope carries `previous_fingerprint`, `plan_fingerprint` and `batches_kept`.
A refusal is one `output.Err` line carrying the engine's message, and nothing is saved.
`Short` is non-empty; `Long` describes D10 in one paragraph with an example.

`cli.go` registers the command and adds a `lyx webster rebaseline` line to the parent `Long`'s verb list.
`cmd/lyx/helptree_test.go`'s webster row gains `rebaseline`.

`verbs_test.go` (integration) adds `TestRebaselineCmd_AcceptsForeignEditAndKeepsRecords`: a state with batch 1 recorded done, a plan edit to a later card, the verb exits 0 with `batches_kept` 1, state.json's fingerprint equals the recomputed one and batch 1's record is intact;
and `TestRebaselineCmd_RefusesRemovedCard`: removing batch 1's card makes the verb exit non-zero with a message containing `--fresh`, leaving state.json byte-identical.

**ImpactSummary:** Adds one webster subcommand and a help-tree row; no existing verb's behavior changes.

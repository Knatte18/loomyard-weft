# Card 6 — webster-contract-file-stencils

**Edit:**
- `contracts/stencils/webster/webster-prefix-fork.md`
- `contracts/stencils/webster/webster-body-verify-fix.md`
- `contracts/stencils/webster/webster-template-master.md`
- `internal/websterengine#RenderForkPrompt`
- `internal/websterengine#RenderVerifyFixPrompt`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#Run`
- `internal/websterengine/template_test.go`
- `contracts/stencils/claims_test.go`

**Uses:**
- `internal/websterengine#OutcomePath`

**Intent:**
D4: forks never write Merriam's two contract files, and Master needs no script to read or write a file.

`webster-prefix-fork.md` (the fork-only half of the batch fork prompt; the recovery strand does not inherit Master's context and keeps its own prefix) gains a section naming `{{.outcome_path}}` and `{{.summary_path}}` as Merriam's own contract files, which the fork never writes, creates or deletes, with the reason: a fork writing either forges the run's terminal judgment and halts the run at its exit audit.
`webster-body-verify-fix.md` gains the same rule over the same two markers, beside its existing "never touch the plan directory or `_lyx`" bullet; the fixer inherits Master's "your final action is writing outcome and summary", which the existing bullet did not override by name.
Each stencil's banner comment states its new required markers.
`webster-template-master.md`'s "What you never do" section states that Master reads files with Read and Grep, writes its two contract files with Write, and never runs a script, an interpreter or a heredoc to read or write a file.

`RenderForkPrompt` and `RenderVerifyFixPrompt` each take `outcomePath` and `summaryPath` and fill the two markers.
`BeginBatch` passes `OutcomePath(deps.Geom.WebsterDir)` and `summaryparser.Path(deps.Geom.WebsterDir)`;
`Run` passes the `outcomePath` and `summaryPath` it already holds.
Stencils stay files read at call time through `stencilstore` (PATTERN-stencil-ownership).

Tests: `claims_test.go` rows assert the fork prefix and the fixer body each name both contract-file markers as never written, and the master template's no-script sentence;
`template_test.go`'s fill and render tests pass the new arguments, and a rendered fork prompt and fixer prompt each contain both rendered paths.
Grep the tests for every `RenderForkPrompt` and `RenderVerifyFixPrompt` call and every marker-set pin of these three stencils, and update each.

**ImpactSummary:** two render signatures gain two path arguments with one caller each; three stencils gain prohibitions and no step changes.

# Card 16 — Merriam fixes a gate failure through one fixer fork

**Create:**
- `contracts/stencils/webster/webster-body-verify-fix.md`

**Edit:**
- `internal/websterengine/runlevel.go`
- `internal/websterengine/render.go`
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/template_test.go`
- `contracts/stencils/stencils.go`
- `contracts/stencils/claims_test.go`
- `contracts/stencils/webster/webster-template-master.md`

**Uses:**
- `internal/websterengine/verifygatereport.go`

**Intent:**
D5: on a gate failure Merriam spawns one fixer fork, and the fork's writes are audited like a batch fork's.

**The fixer prompt.**
At run entry `Run` renders `webster-body-verify-fix.md` (registered in `stencils.go`) into the prompts dir in place of the integration prompt, through a new `RenderVerifyFixPrompt` pointing at `VerifyGateReportPath`.
The fixer reads the gate report, fixes the cause in source, never deletes, skips or weakens a test, never touches the plan directory or `_lyx`, commits each fix as `fix: <summary>`, runs the failing packages' tests with `-tags integration` before reporting, and does not run the plan-level verify.

**Merriam's stencil.**
The integration-fork section and its `integration_prompt_path`/`integration_report_path` markers go, and `RenderMasterPrompt` swaps them for `verify_fix_prompt_path`.
A gate-failure section says that on a `Gate findings recorded at …` message Merriam reads the findings, spawns one fixer fork in the background with the Go-rendered prompt at `{{.verify_fix_prompt_path}}`, and ends its turn;
on the fork's notification it rewrites `outcome.yaml` and `summary.md`, the summary gaining a "Verify gate fixes" section, and ends its turn, which re-arrives at the gate.
The waiting turn end in between is no arrival (card 2).

**The fork audit.**
Bound: fixer commits skip record-batch's done-checks, drift detection and glyph scope guard.
They are bounded by the gate's commit check (card 15), the run-exit fork audit, the gate's own verify and Webster-Review, which reviews the committed range.
The plan directory and `_lyx` sit outside every warp commit, so the run-exit fork audit is what catches a fixer write there:
`forkOwnReport` maps a fork transcript outside every batch bracket to `VerifyGateReportPath`, and `CheckFork` audits the fixer fork as it audits a batch fork.
record-batch's terminal-batch audit skips while `verify-gate.yaml` exists, where it skipped while the integration report existed, since an unseen transcript may then be the fixer's.
`doc.go` replaces the integration-fix section with the fixer section.

**Tests.**
`runlevel_test.go`: a fixer fork that writes the plan directory is flagged by the run-exit fork audit;
the rendered fixer prompt names the gate report.
`recordbatch_test.go` follows the skip condition.
`template_test.go` and `claims_test.go`: Merriam's stencil has no integration fork and has the gate-failure fixer section;
the fixer stencil carries its never-weaken-a-test, never-touch-the-plan and `fix: <summary>` rules.

**ImpactSummary:** confined to webster: Merriam's prompt gains the fixer section in place of the integration fork, and the run-exit audit attributes an out-of-batch fork to the gate report instead of the integration report; no other module reads either.

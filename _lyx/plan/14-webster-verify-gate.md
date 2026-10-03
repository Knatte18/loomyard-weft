# Card 14 — the webster verify gate on Merriam's strand

**Create:**
- `internal/websterengine/verifygate.go`
- `internal/websterengine/verifygate_test.go`
- `internal/websterengine/verifygate_integration_test.go`
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
- `internal/loomcli/wiring.go`

**Uses:**
- `internal/verifytree/verify.go`
- `internal/websterengine/gitwrap.go`

**Intent:**
D3, D4 and D5: the plan-level verify becomes a must-pass shuttle gate entry on Merriam's strand, and a failure goes back to Merriam to fix.
`Run` stops calling `runIntegrationStage` and stops rendering the integration prompt;
card 15 deletes the dead code.

**The gate (`verifygate.go`).**
`Run` hands `StartMaster` `deps.Gate` plus one entry named `verify`, must-pass, with `Attempts` from `Config.VerifyGateAttempts` (card 10).
The shipped Webster row supplies no gates of its own, so Merriam's spec is that one entry.
The closure, evaluated at each gated Done arrival, does in order:

1. When `outcome.yaml` names an outcome other than `done`, it passes without verifying, so a stuck or paused outcome ends the run as today.
2. `verifytree.DirtyPaths` on `Geom.WorktreeRoot`; a dirty tree fails with the paths as findings and runs no verify.
3. From the second arrival on, the fixer commit check:
   every commit from the pre-fix head to HEAD on the first-parent chain is a non-merge commit or a clean parent merge as `gitwrap.go`'s `parentMergeRejection` accepts it (no merge accepted in standalone mode, where `ParentBranch` is nil);
   anything else fails `Terminal`, naming the commit.
4. With no `## verify:` section it passes with a warning.
5. Otherwise `verifytree.Verify` with `verifytree.NewPaths(Geom.WorktreeRoot, Geom.VerifyDir)`, site label `webster gate` and the attempt number.
   A pass or skip passes, and the verify records the tree.
   A failure is parsed with `parseVerifyFailures` from the log, then rerun once;
   a rerun pass passes the gate with the existing flaky warning, summary section and friction note (`triageWarnings`, `AppendIntegrationTriage` and `writeTriageFrictionNote` over the flaky identities only), which `Run` applies after `Wait`.
6. A failure that survives the rerun records the HEAD as the pre-fix head on the first failure, and returns findings.

Attempt counts and the pre-fix head live in the closure's memory for one shuttle run, as the decision record's constraint states.

**Findings and the card hint (D4).**
The findings text names the attempt and the cap, each failing test identity with its output tail, the full log path, and a hint:
the cards, by `NN-slug`, whose recorded `CardSHAs` commits touched a file in a failing package's directory (the package import path mapped to a directory through `go.mod`'s module path).
Go writes the same content plus the fix commits (pre-fix head to HEAD) to `verify-gate.yaml` in `Geom.ReportsDir`, exported as `VerifyGateReportPath`, through a `VerifyGateReport` type, on every failed evaluation;
`Run` removes a stale one at entry.
No bisect, no baseline run and no detached checkout of the live worktree.

**Run's outcome mapping.**
After `Wait`, a done result whose `Gate` did not pass maps to outcome `stuck`, with a reason naming the failing identities read from `verify-gate.yaml` and the attempts spent, or the `Terminal` failure's `Gate.Reason`.

**The fixer (D5).**
At run entry `Run` renders `webster-body-verify-fix.md` (registered in `stencils.go`) into the prompts dir in place of the integration prompt, pointing at `verify-gate.yaml`.
The fixer reads the gate report, fixes the cause in source, never deletes, skips or weakens a test, never touches the plan directory or `_lyx`, commits each fix as `fix: <summary>`, runs the failing packages' tests with `-tags integration` before reporting, and does not run the plan-level verify.

Merriam's stencil:
the integration-fork section and its `integration_prompt_path`/`integration_report_path` markers go;
a gate-failure section says that on a `Gate findings recorded at …` message Merriam reads the findings, spawns one fixer fork in the background with the Go-rendered prompt at `{{.verify_fix_prompt_path}}`, and ends its turn;
on the fork's notification it rewrites `outcome.yaml` and `summary.md`, the summary gaining a "Verify gate fixes" section, and ends its turn, which re-arrives at the gate.
The waiting turn end in between is no arrival (card 2).

Bound: fixer commits skip record-batch's done-checks, drift detection and glyph scope guard.
They are bounded by the commit check, the run-exit fork audit, the gate's own verify and Webster-Review, which reviews the committed range.
The plan directory and `_lyx` sit outside every warp commit, so the run-exit fork audit is what catches a fixer write there:
`forkOwnReport` maps a fork transcript outside every batch bracket to `VerifyGateReportPath`, and `CheckFork` audits the fixer fork as it audits a batch fork.
record-batch's terminal-batch audit skips while `verify-gate.yaml` exists, where it skipped while the integration report existed, since an unseen transcript may then be the fixer's.

`loomcli/wiring.go` fills `RunDeps.ParentBranch` from the pair's fabric origin, as `webstercli` already does, so a clean parent merge during a fix passes the commit check under loom.
`doc.go` replaces the integration-stage and integration-fix sections with the gate, findings, hint and fixer sections.

**Tests.**
`verifygate_test.go` (closure seams over verify, dirty and commit checks):
a non-`done` outcome passes without verifying;
a flaky failure that passes on rerun passes with the flaky warning;
a failed evaluation writes `verify-gate.yaml` with the failing identities, the card hint and the fix commits.
`verifygate_integration_test.go` (real scratch repo):
a dirty tree at a done arrival fails with the dirty paths and runs no verify;
a merge commit above the pre-fix head other than a clean parent merge fails `Terminal`, and a clean parent merge there does not.
`runlevel_test.go` (fake starter driving the captured gate):
a fake Merriam whose first done arrival fails verify and whose second passes ends `done` with one re-prompt, and the findings name the failing identity and the card whose commit touched its package;
an exhausted gate ends `stuck` with a reason naming the failing identities;
a fixer fork that writes the plan directory is flagged by the run-exit fork audit.
`recordbatch_test.go` follows the skip condition.
`template_test.go` and `claims_test.go`: Merriam's stencil has no integration fork and has the gate-failure fixer section;
the fixer stencil carries its never-weaken-a-test, never-touch-the-plan and `fix: <summary>` rules.

**ImpactSummary:** webster's plan-level verify moves from a post-exit Go stage to a must-pass gate on Merriam's live session; Merriam's stencil, the run outcome mapping, the fork audit's out-of-batch mapping and loom's RunDeps wiring change with it.

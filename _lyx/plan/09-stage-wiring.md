# Card 9 — stage-wiring

**Edit:**
- `runlevel.go`
- `integration_test.go`
- `runlevel_test.go`

**Uses:**
- `internal/websterengine#IntegrationReport`
- `internal/websterengine#ParseIntegrationReport`
- `internal/websterengine#WriteIntegrationReport`
- `internal/websterengine#triageOutcome`
- `internal/websterengine#triageIntegrationFailure`
- `internal/websterengine#runVerifyCapture`
- `internal/websterengine#IntegrationLogPath`
- `internal/websterengine#AppendIntegrationTriage`
- `internal/websterengine#triageWarnings`
- `internal/websterengine#triageStuckReason`
- `internal/websterengine#writeTriageFrictionNote`
- `integration.go`
- `summary.go`

**Intent:** Run triage inside `runIntegrationStage` and make the integration verdict webster's: flaky or pre-existing keeps Master's outcome, a regression demotes `done` to `stuck` (D1, D2, D3, D7, D8, D10, D11).

`runIntegrationStage` parses the report with `ParseIntegrationReport`.
On `FAILED`, whenever every batch is terminal-done, it triages regardless of Master's outcome, keeping the three-phase lease split:

- **Phase 1, unleased:** load state; collect card SHAs and labels as today;
  the baseline is the `StartSHA` of the first batch in execution order (`batches[0]`'s record), or empty when that record or its `StartSHA` is absent.
  That value is the HEAD before batch 1 began, per D7, because `recover-batch` inherits it and card 12 makes a begin-batch re-begin keep it.
- **Phase 2, unleased:** open the bisector when `deps.OpenBisector` is non-nil;
  call `triageIntegrationFailure(runVerifyCapture, bisector, baseline, plan.Verify, deps.Geom.WorktreeRoot, deps.Geom.ScratchDir, IntegrationLogPath(deps.Geom.ScratchDir))`.
  On a regression, localize with `LocalizeIntegrationFailure` over the regressing ids when a bisector exists;
  with no bisector keep today's "could not be localized" warning and `unknown`/`unknown`.
- **Phase 3, leased:** reload the report, set its `Failures` and `Triage`, and write it back with `WriteIntegrationReport`.
  On a regression: write the `-1` record, and append the `## Integration suite failed` section with the regressing failures taken from the report's `Failures`.
  On a non-regression: append the triage section, write the friction note to `deps.FrictionDir`, and write no state record.
  Save state.

Outside the lease, emit one `logger.Warn` line naming the verdict and identities for a non-regression verdict.
The stage returns its warnings (triage's own, `triageWarnings`, and, on a regression only, the no-bisector notice) plus a stuck reason that is non-empty only for a regression, built with `triageStuckReason`.

Remove the loud "done over a FAILED suite" error;
keep the loud "done with no integration report" error.
In `Run`'s `OutcomeDone` branch, a non-empty stuck reason demotes a Master `done` to `RunResult{Outcome: stuck, StuckReason: <reason>}`, keeping `BatchesDone` and `SummaryTitle`, and returned as a result, not an error.
A Master `stuck` stays `stuck` with its own `StuckReason` on every verdict, since Master's reason may carry something webster cannot see.
Go never rewrites `outcome.yaml`.
Update the `RunResult.Warnings` doc comment and `runIntegrationStage`'s doc comment to describe triage in place of the fail-loud rule.
Update the `RunResult.Outcome` and `RunResult.StuckReason` doc comments, which say both are taken verbatim from `outcome.yaml`, to name the integration stage's demotion of a `done` as the one exception.
Rewrite the comment at `Run`'s `runIntegrationStage` call site, which explains the error path through the removed done-over-a-failed-suite error, to describe the error paths that remain.

Every test that spawns a shell or git here stays `integration`-tagged.
In `integration_test.go`, each fixture sets batch 1's `StartSHA` to the commit before its card SHAs, and scripts the fork log at `IntegrationLogPath` from `onWait` where a case needs a first-run set.
Rework `TestIntegrationStage_FailedSuite_DoneOutcomeFailsLoud` into the demotion case, and cover:

- Flaky: a verify that passes on rerun under Master `done` ends `done`, the report carries `verdict: flaky` with the first-run identities and tails, `summary.md` carries the triage section, `RunResult.Warnings` names the flaky identity, the friction note exists when `FrictionDir` is set and is absent when it is empty, no bisect checkout happens, and no `-1` record exists.
- Pre-existing: a verify emitting `go test`-shaped `--- FAIL:` output at both head and baseline ends `done` with `verdict: pre-existing`, a pre-existing warning, the summary section, and no bisect.
- Regression: a test failing at head and passing at baseline ends `stuck` with `verdict: regression`, a `StuckReason` naming the test and the localized card, the `-1` record, and `summary.md` naming the test with its tail.
- Master `stuck` over a FAILED report with a flaky verdict stays `stuck`, still gets the report's triage fields and the summary triage section.
- Batch 1 holding two commits where the first breaks a test: the baseline is batch 1's `StartSHA`, so the test is a regression, not pre-existing.
- The worktree is back on its branch after the baseline run, including when the baseline verify leaves the tree dirty with an untracked file and an uncommitted change to a tracked file identical at baseline and head.
  `RestoreBranch` is a plain checkout, so a change that conflicts with the branch surfaces as a restore error instead, per the Constraints; card 5's restore-failure case pins that path.
- A missing fork log does not error, and identities come from the rerun.

In `runlevel_test.go`, rework `TestRun_NilOpenBisectorRecordsUnlocalizedIntegrationFailure` so its verify fails on rerun, asserting `verdict: regression` with both the not-localized and the baseline-unavailable warnings, and add a nil-bisector case whose verify passes on rerun yielding flaky, with the flaky warning as its only warning.

**ImpactSummary:** Changes `runIntegrationStage`'s return shape and `Run`'s `OutcomeDone` mapping, both local to `internal/websterengine`; the Shed adapter already maps `stuck` with a reason and needs no change.

**Commit:** 9: stage-wiring

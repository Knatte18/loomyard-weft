# Card 5 — triage-core

**Create:**
- `triage.go`
- `triage_test.go`
- `plan:internal/websterengine#triageOutcome` -> `type triageOutcome struct`
- `plan:internal/websterengine#triageIntegrationFailure` -> `func triageIntegrationFailure(run verifyRunner, repo FabricBisector, baselineSHA, verifyCmd, worktree, scratchDir, firstLogPath string) (triageOutcome, error)`

**Uses:**
- `plan:internal/websterengine#IntegrationFailure`
- `plan:internal/websterengine#IntegrationTriage`
- `plan:internal/websterengine#parseVerifyFailures`
- `plan:internal/websterengine#opaqueFailureID`
- `plan:internal/websterengine#verifyRun`
- `plan:internal/websterengine#verifyRunner`
- `internal/websterengine#FabricBisector`

**Intent:** Classify a `FAILED` integration report as flaky, pre-existing, or regression, mechanically and in Go (D1, D6, D7, D8).
The function touches no state, report, or summary file, so the stage can call it with no state-mutation lease held.

`triageOutcome` holds `Failures []IntegrationFailure`, `Triage IntegrationTriage`, and `Warnings []string`.

`triageIntegrationFailure`:

1. Reads `firstLogPath`.
   A missing or unreadable log is not an error: the first run's failure set is then unknown.
   Otherwise the first set is `parseVerifyFailures(log, false)`.
2. Reruns the whole verify command once at the unchanged head through `run`, logging to the rerun scratch log.
   Green: verdict flaky, rerun `passed`;
   every first-set identity is recorded flaky, or the opaque identity (empty tail) when the first set is unknown.
   Red: rerun `failed`;
   the head set is the rerun's parsed set, and first-set identities absent from it are recorded flaky, keeping their first-run tails.
3. On a red rerun with a non-nil `repo` and a non-empty `baselineSHA`: captures the current branch, checks out `baselineSHA` detached, runs the verify command through `run` logging to the baseline scratch log, and restores the branch afterwards even when the run errored.
   A failed restore is surfaced (joined onto any run error), never dropped, matching `bisect`'s restore handling.
   Records `baseline_sha`.
   Each head identity is pre-existing when the same `id` fails at baseline and is not opaque;
   otherwise it is a regression (passes, absent, or never ran at baseline).
   Verdict regression if any head identity regressed, else pre-existing.
4. On a red rerun with no `repo` or an empty `baselineSHA`: every head identity is a regression, and a warning states the baseline comparison was unavailable.

`Failures` lists the head set with rerun tails, then the first-run-only identities with first-run tails;
an identity failing only in the rerun is therefore listed with the rerun's tail.
A spawn error from `run` or a checkout error is returned as an error.

The untagged `triage_test.go` (package `websterengine`) drives the function through a fake `verifyRunner` whose output depends on the fake bisector's current checkout, and a fake `FabricBisector` recording checkouts and restores.
It covers every triage acceptance case in the decision record:
flaky (red first, green rerun, no baseline run and no checkout);
pre-existing (red at head and baseline for the same test identity);
regression (red at head, green at baseline);
a test absent at baseline is a regression;
mixed pre-existing and regressing failures yield regression with both lists populated;
opaque red on rerun is a regression even when also red at baseline;
opaque green on rerun is flaky;
nil `repo` with a red rerun yields regression plus the baseline-unavailable warning, and with a green rerun yields flaky;
an empty `baselineSHA` behaves like nil `repo`;
a missing first log yields identities from the rerun without error;
a rerun-only failure appears in `Failures` with the rerun's tail;
the branch is restored after the baseline run, including when the baseline run returns an error;
and a restore failure is returned.

**Commit:** 5: triage-core

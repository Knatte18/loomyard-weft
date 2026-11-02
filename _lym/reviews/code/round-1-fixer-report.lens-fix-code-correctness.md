Task: fix-code-correctness lens over the code the fix changed.

No findings.

I ran each changed path by hand against the code and tests in my context.

- **`readDeployState(prime, readInstalled)`:**
  - The only production caller is `deps.go`, which passes `runningLymBuild`, so the behaviour is the same as before.
  - A build-read error comes back unchanged. A `HeadSHA` or ancestry failure still returns the empty state with the error.
  - The new rows catch a broken short-circuit. If the empty-command or branch-list check were dropped, `readInstalled` would be called with `installed == nil`, which fails the test.
  - The ancestor, head and missing-commit rows pin the `CommitExists` and `IsAncestor` wiring, and the first gate run proved the seed reaches the loader.
- **`deployFailure`:** I walked all four rows and the expected strings match. The start error wraps its cause, so `errors.Is` holds. The timeout row uses `DeployTimeout.String()`, the same text `%s` prints. A zero exit gives nil, and `runAdoptDeploy` still returns the log path alongside the error.
- **`AStopHoldingRootBackDeploysNothingAndWritesNoOwner`:** the row asserts kid1 is stopped, so the setup does hold root. If `!held[root]` were removed from `rootGoing`, `rootNamed` alone would run the deploy and `requireNoDeploy` would fail. The row guards what F1 named.
- **Failed-deploy row:**
  - The refusal path returns `finish()`, which fills `Mutations` and `Partial` from `rec`.
  - `rec` is seeded through `rec.Extend(report.Mutations)`. The test's `AppendRef` target and detail match the expected entry exactly.
  - A refusal that dropped them would fail the new assertion.
- **Renamed row:** `newWorld` sets `Prime: here`, so the row does cover the already-owned case. It asserts the deploy runs once and comes before the owner write.
- **`export_test.go` and `deploy_integration_test.go`:**
  - The aliases and wrapper compile, and the gate passed tier 0 and tier 1.
  - `errors.Is(err, nil)` is true only when `err` is nil, so the error-row logic is correct.

No further tool calls made.

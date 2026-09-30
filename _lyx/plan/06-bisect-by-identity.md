# Card 6 — bisect-by-identity

**Edit:**
- `integration.go`
- `runlevel.go`
- `integration_test.go`

**Create:**
- `bisect_test.go`

**Uses:**
- `plan:internal/websterengine#parseVerifyFailures`
- `plan:internal/websterengine#opaqueFailureID`
- `plan:internal/websterengine#verifyRun`
- `plan:internal/websterengine#verifyRunner`
- `plan:internal/websterengine#runVerifyCapture`

**Intent:** Localize a regression by the regressing identities rather than by the whole command's exit code, so a pre-existing failure present at every card SHA no longer makes bisect blame card 1 by arithmetic (D9).

`bisect` and `checkoutAndVerify` take a `verifyRunner` and a pass predicate over `verifyRun` instead of calling the runner directly and reading the exit code.
`LocalizeIntegrationFailure` and `BisectAndEscalate` gain a `regressions []string` parameter, append it after the existing ones, and delegate to an unexported variant that also takes the `verifyRunner`;
the exported pair passes `runVerifyCapture`, and bisect steps pass an empty log path.
The predicate at each SHA is "none of the regressing identities appears in that SHA's `parseVerifyFailures` set".
An identity that is absent, never ran, or surfaces only as its package identity counts as passing there.
When `regressions` is empty or contains `opaqueFailureID`, the predicate is the command's exit code, as today.
The last-SHA-passes → `unknown` rule and the defer-restore handling stay unchanged.

Update `runIntegrationStage`'s call to pass nil regressions, keeping today's behavior until card 9 wires triage in.
Update the `integration`-tagged `integration_test.go`'s direct `BisectAndEscalate` calls for the new parameter, passing nil so they keep exercising the exit-code predicate.

The untagged `bisect_test.go` (package `websterengine`) drives the unexported variant with a fake `FabricBisector` and a fake `verifyRunner` keyed by the checked-out SHA.
It covers:
a pre-existing failure present at every SHA plus a regression introduced at the second card localizes to the second card, not the first;
a regressing identity absent at early SHAs counts as passing there;
an opaque regressing set falls back to the exit-code predicate;
a last SHA on which no regressing identity fails yields `unknown`;
and the branch is restored after the search.

**ImpactSummary:** `LocalizeIntegrationFailure` has one production caller, `runIntegrationStage`, updated to pass nil regressions here and real ones in card 9; `BisectAndEscalate` has only test callers.

**Commit:** 6: bisect-by-identity

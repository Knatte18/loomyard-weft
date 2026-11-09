# Burler fixer-report, code review round 1

All six findings are fixed as proposed, except F6, whose option 1 was taken. The gate `lym gate test --wait --upto 1` over `internal/loomshed`, `shuttleengine`, `loomcli`, `gatecli` and `darnengine` passed.

## Changes

- **F1** `internal/shuttleengine/gate_test.go`: the wait-expiry, re-prompt-send-failure and session-end causes each assert `EndedEarly` in the table that already drives that path.
  New `TestGate_EndedEarlyStaysEmpty` pins it empty on a pass, a terminal failure and a failure whose entry spent its attempts.
- **F2** `internal/loomcli/wiring_darn_test.go` and `darntreescope_integration_test.go`: `VerifyCommand("darn-write")` ends with `--stop-on-fail`, and `code-burler-fix` and `finalize` do not.
  The wired `VerifyTreeScope` carries the flag for `darn-write` alone, and an empty scoped command stays empty.
- **F3** new `internal/loomshed/verifygate_budget_integration_test.go`: `NewVerifyGate` over a real failing verify arms the timer for the whole budget, cancels it on return, and sends one notice naming the attempt and the failed run.
  `wiring_darn_test.go` asserts `verify_budget_min: 45` gives a `VerifyBudget.Limit` of 45 minutes with a `Notify`.
- **F4** `internal/shuttleengine/wait.go`: `gateEndedEarly` and its comment sit above `finalizeAndRemoveDiedWindow`, whose own comment is back directly above its func line.
- **F5** rows added for the rerun's dirty tree, timed-out, crashed and no-named-test branches (`verifygate_rerun_integration_test.go`), the render-error fallback (`verifygate_test.go`), a build-blocked stop and a doubly crashed tier that does not stop (`gatecli/test_integration_test.go`), and the notice cut on a rune boundary plus the exit-code fallback (`verifybudget_test.go`).
- **F6** `internal/loomshed/verifybudget.go`: `budgetClock` keeps one entry per failed verify run, and the notice renders all of them, latest first, within `verifyNoticeBytes`.
  The wording in `verifygate.go`, `doc.go` and `darnengine/template.yaml` now says "failed runs so far".

A later gate run found three test comment lines of 120 characters or more, in `darntreescope_integration_test.go`, `verifygate_rerun_integration_test.go` and `verifygate_test.go`.
Each is now split at a sentence boundary or shortened, and committed as a repair of that gate finding.

The changes are not committed one per finding: F2 and F3 share `wiring_darn_test.go`, F5 and F6 share `verifybudget_test.go`, and one gate run covers them all.

## Fixed differently

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW, swallowed-input row pinned the upstream `shuttle: send text:` prefix: fixed; the row now asserts `pane swallowed input`.
- fix-proposal, LOW, the render-error test passes an empty list: disputed, see below.
- fix-proposal, LOW, the notice cut loses the earliest failures: disputed, see below.

## Disputed

- **fix-proposal LOW, render-error row:** ground `every fix worse`. `FailedTestsCall` returns an error only for an empty list or a `%` under windows quoting (`internal/verifyrun/run.go:78-86`), and the tests run on a non-windows host where `QuoteArg` never errors. The empty list is the one input that reaches the `err != nil` branch of `runFailedTestsAlone`, which F5 named.
- **fix-proposal LOW, notice cut:** ground `every fix worse`. The cap keeps the notice one short creel line (`verifyNoticeBytes`), and rendering latest first keeps the most actionable failure. Dropping the cap would break `TestVerifyBudget`'s cut row and the single-line contract.

## Deferred

None.

## Rulings

None.

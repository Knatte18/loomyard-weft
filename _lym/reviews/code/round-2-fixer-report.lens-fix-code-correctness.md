```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I ran each changed branch with a concrete input and found no BLOCKING or MEDIUM defects.

Branches traced:
- **F13 below the window, accept over an open scope BLOCKING entry:** `decisionRefusal` returns "". The verdict is CONTINUE, so `openBlockingKeys(round, ClassDecision)` returns ["hard"]. `softCapWindowOpen` is false, so `ensureFocus(round+1)` runs and the result is Stuck with `BudgetExempt` false.
  - That value is correct, because `budgetReached` is false whenever the window is closed.
  - The recorded accept stays pending and unsettled. A re-entry returns the same Stuck.
- **F13 in the window, same ledger:** `retireContinue` archives the escalation record, the parent notice and the decision file. `escalateWith` then finds no record and writes a fresh cause-budget record whose detail contains "leaves BLOCKING findings open (hard)".
  - The next Call sees `escalated` with cause budget. `routeDecisions` returns handled false and the normal `settleUndecided` path takes over, so there is no loop back to the decision cause.
  - A CONVERGED verdict skips the new check, since `refusalLeavingOut` already shows the open entry.
- **F13 tests:** Both new rows fail with the defect in place. Without the new check, `acceptRound` returns Done where the rows expect Stuck and Awaiting. The `settled` assertion also catches a settled accept. The tier0 gate passed.
- **F12:** `wantDetailAlso` asserts "the review also holds decision findings (D1)". It fails if the id assignment at `bouncerdecision.go:151` or the appended text at `:153` is dropped.
- **F2:** `yaml.Unmarshal(nil, &keys)` returns no error and an empty map, so absent input and header-only input are flagged. A file that sets false still returns `existing` with no flag.
  - The caller is `hubreconcile/reinterpret.go:64`. It skips absent files unless `RunsWithoutFile` is set, so the new flag on the loom entry makes the walk run the check with nil.
- **F11 tests:** Both `ProfileMapping` subtests (`HardenRoundAdmitsIt`, `OtherRowRefusesIt`) passed in the gate log. Setting `FocusFindings: hardenRound` at `entries_burler.go:183` to a constant false makes the `HardenRoundAdmitsIt` row fail.
  - The validate rows assert acceptance in `harden` and a refusal containing "does not list" in `code`. Forcing `paths.FocusFindings` false at `validate.go:137` fails the harden row.

Findings:

- **LOW — `internal/shedadapters/bouncerdecision.go`, `holdAcceptOverOpenEntries`:** the `readErr != nil` branch is untested. Its detail text and the retire-and-escalate path have no row. `openBlockingKeys` re-reads a ledger that `settleDecisionAccept` already parsed, so the branch is nearly unreachable and the gap is minor.
- **LOW — `internal/configreg/configreg.go`, loom `RunsWithoutFile: true`:** the fixer ran tier0 for `configreg` and `hubreconcile` callers, and tier1 only for `burlercli`. The walk now flags every hub with no `loom.yaml`. A higher-tier test that asserts a notice-free reconcile of such a hub would only show up at tier1 or above, and `hubreconcile` and other `configreg` callers were not run there.

Tool calls:
1. `grep RunsWithoutFile` in `internal`, to confirm the walk caller at `hubreconcile/reinterpret.go:64` passes nil bytes when the flag is set.
2. `grep` of `retireContinue` in `shedadapters`, because its body was not in my context.

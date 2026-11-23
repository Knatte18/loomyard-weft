```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the goal point of the decision record that asks for it.

**Decisions** (the record's five `## Goal` points; the record has no `Added after Discussion` entries)
- **D1 `fixed-blocking-escalates-parent`**, verify fixed BLOCKING findings before a spent budget asks the parent. Built by C1 to C7.
- **D2 `below-window-accept-blocking`**: no change in the diff. The behaviour is already present at the base: `holdAcceptOverOpenEntries` (`bouncerdecision.go`, reached from `settleDecisionAccept`) runs another round below the window and does not pass the accept. The entry's literal remedy is "the run should ask the parent instead", and that is not built. See F1.
- **D3 `budget-escalation-untested`**: no change in the diff. The base already asserts the detail in `bouncer_escalation_test.go:494-498` (`wantDetailAlso: "the review also holds decision findings (D1)"`). Satisfied.
- **D4 `code-auto-accept-flag-only`**: no change in the diff. The summary confirms that the flag is enough, which is one of the two outcomes the entry allows. Satisfied.
- **D5 `hardener-focus-wiring-untested`**: no change in the diff. Coverage is already there at the base: `internal/shedrecipe/entries_burlerhalves_test.go:329` "HardenRoundSetsTheFocusFindingsPermission" (producer, then round context, then `CheckReviewRound`), and `internal/burlercli/validate_integration_test.go:54` rows "FocusFindingAdmittedInTheHardenerRunDirectory" and "…RefusedInTheCodeRunDirectory". Satisfied.

**Changes**
- **C1**, the verifier pass and the ledger close (`bouncerverify.go` new; `bouncer.go` `blockingEscalationDetail` and the comments on `BouncerConfig.BlockingEscalates` and `settleUndecided`): traces to D1.
- **C2**, `rejudge` also archives `round-<N>-verification.md` (`bouncer.go`): forced by C1.
- **C3**, `renderLedger` and the `omitempty` tags on `ledgerEntryYAML` (`bouncerfiles.go`): forced by C1, since Go now rewrites the judge's ledger. `omitempty` changes only marshalling, not `parseLedger`.
- **C4**, the stencil `bouncer-template-verify.md`, its registry entry, the `roleOpeningStencils` row and the `markers_test.go` row: forced by C1.
- **C5**, the parity row `buildBouncerVerify` and the exported `RenderVerifyPrompt`: forced by C1, per `PATTERN-engine-parity` (a new producer takes a producer-table entry).
- **C6**, the comments in the three recipes, `shed-recipe-spec.md`, `docs/overview.md` and `doc.go`: forced by C1, because they describe `blocking_escalates`.
- **C7**, `bouncer_verify_test.go`: forced by C1.
- **C8**, commit e1118fa3b reflows comments in `bouncer.go`, `bouncerverify.go`, `doc.go` and `bouncer_verify_test.go`: forced, since it only touches lines C1 and C7 add or change.

No change traces to nothing, so the diff adds no unrequested scope.

**Findings**

- **F1**
  - Severity: LOW
  - Class: decision
  - Location: `internal/shedadapters/bouncerdecision.go:221` (`holdAcceptOverOpenEntries`), as the summary's handling of D2
  - Summary: The `below-window-accept-blocking` entry asks that a below-window decision accept over an open BLOCKING entry of another class make "the run … ask the parent instead". The code at the base runs another round below the window and asks the parent only inside it. The diff builds no change for this and records the reading as no deviation. The entry's aim, no landing past an open BLOCKING entry, holds; its literal remedy does not.
  - Fix: Leave the code in place. Defer to the run's parent, which confirms with `lym loom decision add` that another round below the window satisfies the entry, or orders the parent ask.
  - Evidence: `bouncerdecision.go:221-227` (below the window: `ensureFocus(round + 1)`, return Stuck), `_lym/discussion/decision-record.md` `## below-window-accept-blocking`, `_lym/landing/summary.md` "The other board entries folded into this task".

Tool calls:
1. `git show --stat e1118fa3b`, `git diff --stat` over the range, and the two hardener-wiring tests. My context did not show which files the second commit touched, or the bodies of the tests that D5's coverage claim cites.

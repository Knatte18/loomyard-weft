---
verdict: BLOCKING
findings:
  - id: F11
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/shedrecipe/entries_burler.go:183
    summary: The hardener-only focus permission has no test through recipe construction and producer forwarding or through the CLI validator's independent selection.
    fix: Extend TestBurlerReviewEntry_ProfileMapping in internal/shedrecipe/entries_burlerhalves_test.go with complete prior-round fixtures and harden_round true and false cases, exercising the constructed producer and checking that its resulting round context admits an outside-changes BLOCKING decision finding with origin focus only for the hardener case. Extend TestValidateReviewVerb_RoundChecks in internal/burlercli/validate_integration_test.go with equivalent hardener-directory and ordinary-directory cases that reach the new focus check instead of failing the prior-list check first. Reuse the existing tests rather than adding per-helper tests.
  - id: F12
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/bouncerdecision.go:151
    summary: The mixed budget-escalation test does not assert the decision finding IDs in the generated brief, allowing both reported detail mutations to survive.
    fix: Extend the mixed decision-and-BLOCKING row of TestBouncer_AutoAccept in internal/shedadapters/bouncer_escalation_test.go to assert that the budget escalation brief contains its decision finding ID D1 as well as the open BLOCKING key hard. Keep the existing cause and snapshot assertions. Use independent expected values and the existing row so deletion of either the ID assignment or the appended decision detail fails the test.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    carries: F2
    location: internal/loomengine/template.yaml:53
    summary: The configuration reinterpretation is present, but _lym/landing/summary.md still labels the configuration meaning change as a two-way merge door.
    fix: In _lym/landing/summary.md under Merge danger, change the Door classification to one-way because code_auto_accept true changes meaning. Retain the meaning-change explanation and registered reinterpretation; no template or implementation change is needed for this remaining part of F2.
prior:
  - id: F1
    status: resolved
    answer: A fresh undecided mixed CONTINUE or CIRCLING round in the soft-cap window now writes its decision snapshot and escalates with cause budget. CONVERGED retains the refusal that prevents acceptance over the non-decision BLOCKING entry.
  - id: F2
    status: open
    answer: The registry now carries FlagCodeAutoAccept and the summary names the meaning change, resolving the migration gap. The summary still declares a two-way door; only that remaining classification defect is carried, at MEDIUM.
  - id: F3
    status: resolved
    answer: All three consolidation stencils preserve origin focus for the applicable decision finding, and the review format states that preservation.
  - id: F4
    status: resolved
    answer: FocusFindings now gates the exemption and is selected by hardener wiring and CLI directory identification. TestCheckReviewRound refuses the exemption without that flag. F11 separately identifies missing coverage through the entry paths.
  - id: F5
    status: resolved
    answer: Both the review-step stencil and the review format explicitly exempt hardener focus decisions from the changed-hunk and changed-file restrictions.
  - id: F6
    status: resolved
    answer: The below-budget row retains alpha and requires the judge outputs to remain, so a dropped-key rejudge cannot satisfy it.
  - id: F7
    status: resolved
    answer: The replacement test directly exercises the unreadable-ledger settlement branch and checks Awaiting, cause budget and the unreadable detail. The full Call path parses the ledger before settlement and cannot supply this fixture unchanged.
  - id: F8
    status: resolved
    answer: The added hard-cap-only row uses an unspent bounce budget and asserts cause budget.
  - id: F9
    status: resolved
    answer: The overview and config field comment now describe the BLOCKING carve-out and parent escalation.
  - id: F10
    status: resolved
    answer: The recipe switch test now carries the requested testtiming keep marker.
---

### [BLOCKING] Focus permission is untested through its entry paths

**Location:** `internal/shedrecipe/entries_burler.go:183`

**Issue:** The new selection reaches the review context through `burlerreview.go:203` and `:276`.
The CLI independently selects the permission at `internal/burlercli/validate.go:137`.
The focus tests inject the boolean directly into the checker or context builder.
The profile-mapping test does not assert this selection,
and the CLI round-check test fails a missing-prior check rather than exercising an outside-changes focus finding.
Removing either selection or the producer forwarding would leave those assertions satisfied.
This is uncovered behavior under the test-economy rubric and `PATTERN-test-speed`'s public-surface rule.

### [BLOCKING] Mixed escalation does not assert its decision IDs

**Location:** `internal/shedadapters/bouncerdecision.go:151`

**Issue:** The mixed row at `bouncer_escalation_test.go:494` supplies decision finding `D1`.
Its brief assertion checks only `BLOCKING findings open (hard)`.
The current mutation report lists deletion of the ID assignment and appended decision detail as survivors.
Those lines produce data the parent needs, and the function's comment promises that detail.
The existing cause-decision brief assertions exercise a different rendering path.
The rubric requires a finding for this missing check; the output is generated data, not static stencil prose.

### [MEDIUM] Merge door remains inconsistent with the meaning change

**Location:** `internal/loomengine/template.yaml:53`; remaining fix at `_lym/landing/summary.md:77`.

**Issue:** The registered reinterpretation resolves the implementation part of F2.
The summary now acknowledges the changed meaning of `code_auto_accept: true`, but still declares `Door: two-way`.
The merge-danger rubric explicitly rejects that classification for a configuration meaning change.
This carries F2 at reduced severity because the migration obligation itself is now met.

## Focus assessment

- Mixed decision and non-decision BLOCKING findings now take the parent-only budget route in the soft-cap window.
  A CONVERGED verdict retains a refusal against the non-decision BLOCKING entry.
- The reinterpretation is registered and reached by the hub reconcile walker.
  The summary records the meaning change; its remaining door classification is F2.
- Cluster and assistant consolidation preserve `origin: focus`.
- The focus exemption requires a hardener context, and direct checker tests pin refusal elsewhere.
  Entry-path coverage remains F11.
- The incremental review step and format agree on both changed-hunk and changed-file exemptions.
- The BlockingEscalates rows retain the previous ledger key and assert the relevant outcomes.
  The below-budget row additionally detects judge-output archival.

## Verification

Reviewed the supplied diff against `30ed1ee588023fba05fb5eab3c24979eda94d479` through HEAD `0345151b68da3da0d9a3f91c709099d84d992a60`.
Read the decision record, board entry, commit messages, focus directive, relevant callers and repository rules.
All four lens reports are saved beside this report.

Attempted the focused tier-0 gate for shedadapters, burlerengine, loomengine and configreg.
It exited before running tests because the session cannot write the gate holder under `_bolt/.lym/gate`.
The supplied reach, redundancy and mutation reports informed this review; no fresh test pass is claimed.

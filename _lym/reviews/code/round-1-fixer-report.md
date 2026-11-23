---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "The proposed row would drive the unreadable ledger through a full Call, but the judge's verdict read already parses the ledger, so that branch cannot be reached through Call. A new test, TestBouncer_BlockingEscalatesAnUnreadableLedger, writes the unparsable ledger and calls settleUndecided directly, then asserts the budget escalation and the could-not-be-read detail."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
---

# Round 1 fixer-report

## What changed

- F1: `escalateUndecided` (`internal/shedadapters/bouncerdecision.go`) now hands a CONTINUE or CIRCLING round in the soft-cap window to the parent with cause `budget` when `BlockingEscalates` is set and the ledger holds an open BLOCKING entry of a class other than `decision`, or cannot be read.
  The detail names the open keys and the review's decision findings.
  `blockingEscalationDetail` and `openBlockingKeys` take a `leftOut` class, and `softCapWindowOpen` names the window test.
  A CONVERGED verdict keeps cause `decision`, since its refusal check already shows the open entry.
  The decision snapshot is written before either escalation.
  The `BouncerConfig.BlockingEscalates` comment, the `shed-recipe-spec` paragraphs and the escalation stencil's `budget` bullet state the route.
  `TestBouncer_AutoAccept` gains rows for the budget route over a decision finding, a decision-class BLOCKING entry that stays a decision escalation, and a CONVERGED verdict that stays one.
- F2: `loomengine.FlagCodeAutoAccept` flags a hub base file that sets `code_auto_accept: true` and leaves the value as it is; it is registered on the loom module under `CodeAutoAcceptBlockingReinterpretationID`.
  `_lym/landing/summary.md` drops the claim that the key keeps its meaning and names the change under `## Merge danger`.
  Tests: `TestFlagCodeAutoAccept` and `TestModules_LoomRegistersCodeAutoAcceptReinterpretation`.
- F3, F5: the cluster, assist cluster and assist consolidation stencils keep `origin: focus`; `burler-review-format.md` and `burler-step-review.md` exempt a hardener round's focus finding from the changed-hunk and file rules.
- F4: `RoundContext.FocusFindings` gates the focus exemption in `newFindings`.
  `RoundContextSources.FocusFindings`, `BurlerHalfDeps.FocusFindings` and the review producer carry it; the `harden_round` BurlerReview row and the `lym burler validate` check of the hardener's run directory set it.
  `TestCheckReviewRound` gains a refused row with the field unset, and `TestBuildRoundContext` a row for the field.
- F6: the below-budget row's ledger keeps `alpha` and the row asserts the judge outputs are still in place.
- F8: a `TestBouncer_AutoAccept` row at the hard cap with the budget unspent expects cause `budget`.
- F9: `docs/overview.md` and the `CodeAutoAccept` field comment name the BLOCKING carve-out.
- F10: `//testtiming:keep` above `TestDecisionSwitches_SetOnTheRowsThatAskTheParent`.

## Self-check

- fix-code-correctness, LOW: the budget escalation wrote no decision snapshot — fixed (snapshot written first, with a row assertion).
- fix-consistency, MEDIUM: `shed-recipe-spec.md` did not state the budget route — fixed.
- fix-consistency, LOW: the escalation stencil's `budget` bullet did not mention decision findings — fixed.
- fix-proposal, LOW: a CONVERGED verdict was caught by the budget route — fixed (verdict check and row).
- fix-proposal, LOW: garbled doc sentence of `escalateUndecided` — fixed.
- fix-proposal, LOW: the `code_auto_accept` flag overstated its reach — fixed (names the `blocking_escalates` condition).
- fix-affected-set: no findings.

## Gate repair

The gate refused the round before any test ran, over comment lines that wrapped mid-sentence or reached 120 characters.
Every flagged comment in `roundcheck.go`, `configreg_test.go`, `config.go`, `reinterpret.go`, `bouncer.go`, `bouncer_escalation_test.go` and `bouncerdecision.go` now holds one sentence per line under the limit.
No behaviour changed.

## Disputed

None.

## Deferred

None.

## Rulings

None.

## Notes

`lym burler validate` is not covered by a test for the hardener's run directory; the gate over the touched packages passed.

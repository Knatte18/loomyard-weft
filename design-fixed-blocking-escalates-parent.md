# Bouncer escalation fixes: fixed BLOCKING findings, soft-cap accept, auto-accept flag, untested wiring

`fixed-blocking-escalates-parent` · Task · bug, loom, config · running · Drift-Verify

## fixed-blocking-escalates-parent

### Seen

Since `code-bouncer-asks-parent` landed (8cf7f4775), four darn runs on 2026-11-23 stopped at Code-Bouncer with cause `budget` after round 1:
`amend-plan-loses-webster-batches`, `pr-findings-open-fabric-nil`, `beam-pr-describes-itself` and `godocreflow-worsens-findings`.
In each, the fixer had fixed every finding, BLOCKING included, committed it and passed its gates;
the ledger still held the BLOCKING entry `open`, because no later round re-reviewed the fix, and a darn's budget is spent after one round.
The orch then read the fixer report and the commit and accepted each time, which cost each run a parent turn and a wait.

### Idea

A BLOCKING finding the fixer reports `fixed`, with a commit and a green gate, is verified without a full review round before the Bouncer judges the budget:
for example, a short verification of only those findings against the fix commit, which closes the entry or keeps it open.
The parent is asked only when a BLOCKING finding stays open after that check, or the fixer did not fix it.

## below-window-accept-blocking

Left open when `code-bouncer-asks-parent` landed (8cf7f4775), as finding `below-window-decision-accept-passes-open-blocking`.

A decision accept recorded below the soft-cap window passes over an open BLOCKING ledger entry of another class, in `internal/shedadapters/bouncerdecision.go`.
That lets a run land past an open BLOCKING finding, which the landed task was meant to stop: the run should ask the parent instead.

## budget-escalation-untested

Left open when `code-bouncer-asks-parent` landed (8cf7f4775), as finding `budget-escalation-decision-detail-unasserted`.

The decision-findings detail appended to a budget escalation has no test asserting it, in `internal/shedadapters/bouncer_escalation_test.go`.

## code-auto-accept-flag-only

Left open when `code-bouncer-asks-parent` landed (8cf7f4775), as finding `code-auto-accept-meaning-change-no-reinterpretation`.

The narrowed meaning of `code_auto_accept` reaches hubs only through a flagging reinterpretation in `internal/loomengine/reinterpret.go`, with no rewrite.
Confirm a flag is enough, or rewrite the hub layers that set the key.

## hardener-focus-wiring-untested

Left open when `code-bouncer-asks-parent` landed (8cf7f4775), as finding `hardener-focus-permission-wiring-untested`.

The wiring that lets a hardener round raise a focus finding (`FocusFindings` into the round context) is not covered end to end, in `internal/burlercli` and `internal/shedadapters/roundcontext.go`.


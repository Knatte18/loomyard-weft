## Goal

```markdown
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

```

## Scope

## Decisions

### Added after Discussion (arbiter, 2026-11-23): below-window-accept-blocking is settled by another round below the soft-cap window

Decision: The below-window-accept-blocking entry needs no code change. A decision accept recorded below the soft-cap window over a CONTINUE or CIRCLING verdict whose ledger holds an open BLOCKING entry of another class keeps the base behaviour of holdAcceptOverOpenEntries in internal/shedadapters/bouncerdecision.go: it passes nothing and starts another round so the fixer fixes the entry; inside the soft-cap window the accept is retired and the round escalates to the parent with cause budget. The run does not add a parent ask below the window. Review finding F11 (below-window-accept-blocking-unrecorded-reading) is answered by this decision, and the fixer changes nothing for it.

Rationale: The entry names its harm as a run landing past an open BLOCKING finding without the parent. At the base the below-window accept no longer passes such an entry: another round runs, and once the budget is spent the soft-cap window's BLOCKING check escalates with cause budget, which only the parent accepts. So the harm cannot occur, and the entry's premise (the accept passes over the entry) does not hold at the base. Asking the parent below the window, while budget is left for the fixer, would also go against this task's main entry, fixed-blocking-escalates-parent, whose aim is to ask the parent only when a BLOCKING finding stays open after the fixer and the check.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria

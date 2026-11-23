# Decision-findings detail of a budget escalation has no test

`budget-escalation-untested` · Note · bug, loom

Left open when `code-bouncer-asks-parent` landed (8cf7f4775), as finding `budget-escalation-decision-detail-unasserted`.

The decision-findings detail appended to a budget escalation has no test asserting it, in `internal/shedadapters/bouncer_escalation_test.go`.

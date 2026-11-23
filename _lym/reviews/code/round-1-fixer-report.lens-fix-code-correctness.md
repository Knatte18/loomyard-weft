```yaml
lens: fix-code-correctness
outcome: reported
```

Task restated: fix-code-correctness lens on the round-1 change set.

No BLOCKING or MEDIUM findings. One LOW.

**LOW. The budget escalation over decision findings writes no decision snapshot.**
- Changed text: `escalateUndecided` in `internal/shedadapters/bouncerdecision.go`. It returns through `escalateWith(round, EscalationBudget, …)` before `writeDecisionSnapshot`.
- Evidence: `bouncercarryover.go:117` calls `addedSinceSnapshot`, which returns nil when the round has no snapshot, as its doc comment says. When the parent accepts such a round, `acceptRound` runs `writeCarryOver`, and the carry-over then lists no decision-record entries added since the snapshot.
- Impact: every other path to an accept of a decision-bearing round writes the snapshot first.
- Fix: call `writeDecisionSnapshot(b.cfg.RunDir, round, b.cfg.DecisionRecordPath)` before the budget escalation, or accept that the carry-over omits the added-entry titles. This is only a hazard if `bouncercarryover.go` uses `addedSince` to name the entries that answered the held decision findings. I did not read the rest of that file to check.

Traced and found correct:
- **Budget escalation:** `routeDecisions` returns handled false on a later call, because `escalated` is true and the cause is not decision. A parent accept or continue then reaches `judgmentRefusal` and `settle`. `softCapWindowOpen` and the `leftOut` filter give the keys `hard` for a scope entry and none for a decision entry.
- **Callers:** `escalateUndecided` has one caller, `routeDecisions`, and no test calls it. `blockingEscalationDetail` and `openBlockingKeys` are called only from `settleUndecided` (with `""`) and `escalateUndecided`; `go vet` and the tier0 run were clean.
- **New rows with the defect in place:** the decision-finding row would escalate with cause decision and fail `wantCause: budget`. The `wantJudgeKept` row fails under a rejudge, which archives the files.
- **Hard-cap row (F8):** it passes with or without the defect. The old code already escalated with budget there, so it only pins the cause and is not a regression guard.
- **`TestBouncer_BlockingEscalatesAnUnreadableLedger`:** `settleUndecided` runs with a spent budget and a ledger that does not parse, so it takes the unreadable branch and carries "could not be read".
- **`FlagCodeAutoAccept`:** a null or absent value is not flagged, `maybe` is an error, and `true` is flagged with the file returned unchanged.
- **`FocusFindings` wiring:** it is set only when `BuildRoundContext` builds a context, and a nil context refuses the exemption. The `entries_burler.go` literal passes `hardenRound`. `validate.go` compiles because `:=` introduces `hardenSubdir`.

Tool calls:
1. `grep -n "addedSinceSnapshot\|decisionSnapshotPath" internal/shedadapters/*.go`: to find who reads the snapshot, which my context lacked.

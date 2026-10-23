# Card 8 — driver-precondition-row

**Create:**
- `plan:internal/sheddriver#preconditionUnmet` -> `func preconditionUnmet(stop Stop) bool`

**Edit:**
- `internal/sheddriver#decideDrive`
- `internal/sheddriver#TestDecideDrive`
- `internal/sheddriver/doc.go`

**Uses:**
- `internal/sheddriver#budgetExhausted`
- `plan:internal/shedengine#ReasonPreconditionUnmet`

**Intent:**
Add D11's rule row: a blocked stop whose reason, or whose status error for a stop built from a status, starts with `shedengine.ReasonPreconditionUnmet` hands back, ahead of every hand-over, arbiter and tackler row.
`preconditionUnmet` mirrors `budgetExhausted`: it reads `stop.Key().State` as blocked and `strings.HasPrefix` on `stop.Reason` or `stop.Error`, the same stop and status reading the budget row uses (PATTERN-sole-parsers).
`decideDrive` gains the case right after the `budgetExhausted` case, returning `driveActionHandBack` with the rule text `a precondition-unmet halt: hand back, no tackler settles it`.
Bound: the row only hands back and never resumes; only a producer that writes the prefix reaches it.
`TestDecideDrive` gains rows: a precondition halt from the envelope hands back, one from the status error hands back, one on a changed build still hands back, and a blocked stop whose reason only contains the prefix past its start still reaches the tackler row.
`doc.go`'s rule-table list gains the row beside the bounce-budget row.

**ImpactSummary:** One new row early in `decideDrive`; a stop that did not start with the new prefix meets the same later rows as before.

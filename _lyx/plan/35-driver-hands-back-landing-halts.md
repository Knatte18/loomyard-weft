# Card 35 — driver-hands-back-landing-halts

**Edit:**
- `internal/sheddriver#Stop`
- `internal/sheddriver#parseDriveStop`
- `internal/sheddriver#stopFromStatus`
- `internal/sheddriver#decideDrive`
- `internal/sheddriver#Deps`
- `internal/sheddriver#goDriver.recheck`
- `internal/sheddriver#TestDecideDrive`
- `internal/sheddriver#TestParseDriveStop`
- `internal/sheddriver#TestGoDriver_Settle`
- `internal/loomcli#loomCLI.goDriverDepsFor`

**Uses:**
- `internal/landingshed#HandsBack`

**Extends:**
- `internal/sheddriver/doc.go`

**Intent:** The Go driver hands back the landing halts ahead of every hand-over, arbiter and tackler row (D11, `PATTERN-shed-driver`).
`Stop` gains `HandBack`, which `parseDriveStop` reads from the envelope's `loop.hand_back`; a stop built from a status reads it through the new told `Deps.HandBack` check over the status's current producer and error, the same predicate the loop marks with, which `goDriverDepsFor` fills with `landingshed.HandsBack`.
`decideDrive` gains a row right after the paused row and before the budget, precondition, arbiter, hand-over, tackler and re-step rows: a blocked stop marked `HandBack` hands back with the rule "a landing halt: only the orch's resume lifts it".
So a fixer-spent, escalation, fixer-timeout, budget-0 Merge-In over-cap or Finalize reroute-cap halt spawns no tackler and no arbiter, and is never handed over to a fresh driver on a build change; a Webster reroute-cap halt is unmarked and is decided as today.

`TestDecideDrive` gains rows for one marked blocked stop, a marked stop with a changed build and an escalation pending (still hand-back), and a Webster reroute-cap stop (the tackler row as today);
one marked row covers every kind of landing halt, since `decideDrive` reads only the `HandBack` flag and card 34's `TestHandsBack` pins the predicate, the `LandingStopPrefix` every fixer-spent, escalation, fixer-timeout and over-cap reason opens with and the reroute-cap reason; `TestParseDriveStop` pins the envelope key, absent reading as false.
`TestGoDriver_Settle` gains rows for a stop the driver builds from a status in `goDriver.recheck`: a blocked Finalize reroute-cap status hands back through the told `Deps.HandBack`, and with a nil check the same status is decided as today.

`doc.go`'s rule table lists the new row first among the blocked rows.

**ImpactSummary:** One rule-table row and one told check added; the Go driver's only owner wiring is loom's, and batten's driver deps leave the check nil, marking nothing from a status.

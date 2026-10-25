# Card 10 — driver-yield-exit

**Edit:**
- `internal/sheddriver#Stop`
- `internal/sheddriver#parseDriveStop`
- `internal/sheddriver#goDriver.run`
- `internal/sheddriver#TestParseDriveStop`
- `internal/sheddriver#TestGoDriver_Settle`
- `internal/battencli#battenRunEnvelope`
- `internal/battencli#TestRunCmd_ResumeDispositions`
- `internal/sheddriver/doc.go`

**Create:**
- `plan:internal/sheddriver#ExitYield` -> `const ExitYield Exit = "yield"`

**Uses:**
- `plan:internal/shedengine#Yield`

**Intent:**
Make the Go driver read a yield as no stop (D4): no rule-table row, no stop report, no repair, bounce or tackler budget, and the pass exits 0.
`Stop` gains `Outcome`, decoded by `parseDriveStop` from the envelope's top-level `outcome`.
`goDriver.run`, after it saves the envelope and before `settleStop`, ends with `d.exit = ExitYield` and a nil error when the stop is OK, its loop state is `running` and its outcome is `yield`; it appends no settle record and writes no report, park or notice, and logs the yield at Debug (`PATTERN-shed-driver`'s constraint for this task); a stop built from a status or a resumed attempt carries no outcome, so only a stepped envelope yields.
`battenRunEnvelope` answers `ExitYield` with the success envelope `outcome: yielded`, the run's current producer and `reason`, the text of the Run-Shed row's stuck-reason file (`readStuckReason`), so a pass prints the hold or wait reason it yielded on (D4); `history_length` is reported as for any other exit.
The package doc's loop section states that a yield ends the driver with `ExitYield` and nothing decided.
Tests: `TestParseDriveStop` gains a row decoding `outcome: yield`; `TestGoDriver_Settle` gains a row whose step returns a yield envelope: the run ends with `ExitYield`, no settle record and no hand-back call; `TestRunCmd_ResumeDispositions` gains a yield row printing `outcome: yielded` with the reason file's text.

**ImpactSummary:** A new exit and one early return in the driver loop; the rule table and every other exit path are unchanged, and batten's envelope gains one case.

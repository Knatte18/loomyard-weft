# Card 16 — driver-drops-verify-row

**Edit:**
- `internal/sheddriver#driveFacts`
- `internal/sheddriver#driveAction`
- `internal/sheddriver#decideDrive`
- `internal/sheddriver#Deps`
- `internal/sheddriver#goDriver.settleStop`
- `internal/sheddriver#goDriver.facts`
- `internal/sheddriver#TestDecideDrive`
- `internal/sheddriver#TestGoDriver_Settle`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/battencli#TestBattenDriverDeps`

**Delete:**
- `internal/sheddriver#VerifySeam`
- `internal/sheddriver#verifyTestRow`
- `internal/sheddriver#verifyHaltStatus`
- `internal/loomcli#publishHaltMatches`

**Extends:**
- `internal/sheddriver/doc.go`

**Intent:** Remove the Go driver's verify-halt row and its `Verify` seam (D11): a failed landing gate is the fixer's to fix, and no driver sends a run to the failure record's target.
`driveFacts` drops `VerifyMatches`, `VerifyTarget` and `VerifyRetrySpent`, `driveActionVerifyGoto` goes, and `decideDrive` loses its verify-halt case, so a blocked stop falls through to the binary hand-over, tackler and hand-back rows as any other blocked stop does until card 35's landing hand-back row stands ahead of them.
`Deps` drops `Verify`, `goDriver.facts` stops reading it, and `goDriver.settleStop` loses its verify-goto branch.
`goDriverDepsFor` stops building the seam and `publishHaltMatches` goes; `TestBattenDriverDeps` drops its `Verify` row.
`TestDecideDrive` and `TestGoDriver_Settle` lose their verify-halt rows and their two helpers.
`shedrun.SettleVerifyRetry` and `GoVerifyRetrySpent` stay for now: older settle records still carry the action, and batten's retry reads them until card 18 removes it.

`doc.go`'s rule table drops the verify-halt row and the `Verify` seam.

**ImpactSummary:** One rule-table row and one driver seam go; the seam's only filler is loom's driver wiring, and batten's driver deps leave it nil.

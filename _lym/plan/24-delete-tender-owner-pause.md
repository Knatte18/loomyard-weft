# Card 24 — delete-tender-owner-pause

**Delete:**
- `internal/tendercli#PauseOwnedElsewhere`
- `internal/tendercli#endOwnedElsewhere`
- `internal/tendercli#EndOwnedElsewhere`
- `internal/tendercli#ownedElsewhereEnder`
- `internal/tendercli#ownedElsewhereEnder.run`
- `internal/tendercli#ownedElsewhereEnder.removeFollowUp`
- `internal/tendercli#TestPauseOwnedElsewhere_PausesTheSlugOnceUnderTheOwnerCaller`
- `internal/tendercli#TestOwnedElsewhereEnder`

**Edit:**
- `internal/tendercli/doc.go`

**Intent:** Delete `tendercli.PauseOwnedElsewhere` and the process end beside it, whose last callers card 23 removed (D5: "The owner caller retires with every writer: ... tendercli.PauseOwnedElsewhere and the process end beside it").
`pairteardown.EndSubjectProcesses` is the one end of a subject another machine owns.
The tests that covered them go with them, and `doc.go` drops both.

**ImpactSummary:** Removes two exported functions, their helper type and their two tests; no production caller remains after card 23.

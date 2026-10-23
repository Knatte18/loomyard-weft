# Card 15 — Plan-Bouncer's commit removes a carried amendment

**Edit:**
- `internal/loomcli#planCommitPathspecWith`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestPlanCommitPathspec`

**Create:**
- `internal/loomcli#removeSettledAmendment`
- `internal/loomcli#TestRemoveSettledAmendment`

**Uses:**
- `internal/loomcli#amendmentPathspec`
- `internal/loomshed#AmendmentSettledBy`
- `internal/loomshed#ReadAmendment`
- `internal/loomshed#RemoveAmendment`
- `internal/shedadapters#SettleRoundStart`
- `internal/loomrecipe#BouncerRunSubdir`

**Intent:**
D7: the amendment record is removed by Plan-Bouncer's approved settle, inside its commit seam, only when the settle's recorded non-zero round start is after the record's latest write time.

`removeSettledAmendment(amendmentPath, planRunDir)` reads the record; with none it does nothing.
It reads `shedadapters.SettleRoundStart(planRunDir)` and removes the record when `loomshed.AmendmentSettledBy(record, start)` holds; an unknown start keeps it.
A read or removal error is returned, so the commit seam fails as Bouncer's seam failures do and the settle is retried; an undecodable record carries card 11's way forward, and a removal error is a propagated raw I/O error, exempt from a reaching row.
The approved settle writes its settle record before Commit runs (`beginSettle`), so the record read is this settle's.

In `loomCLI.wire`, the `CommitPlan` seam runs `removeSettledAmendment` over `loomengine.LoomAmendmentRecord(location)` and the Plan-Review run directory (the reviews directory joined with `loomrecipe.BouncerRunSubdir(loomshed.NamePlanBouncer)`) before its fabric commit.
`planCommitPathspecWith` appends `amendmentPathspec(location, tracked)`, so the removal, a deletion of a tracked file, is staged in that commit.

What this keeps: an amendment appended while a round runs is after that round's start and stays for one more round; a round-0 settle (a skip-seam approval) records no start and never removes the record; the next approved settle with a later start removes it.

`TestRemoveSettledAmendment` rows: no record; a record written before a settle's recorded start is removed; one written after it is kept; a settle with no recorded start keeps it; an unreadable record is an error.
`TestPlanCommitPathspec` gains rows: a present record adds its path, and an absent untracked record adds none.

**ImpactSummary:** `CommitPlan` serves only Plan-Bouncer's approved settles and skip approvals; with no amendment record it commits exactly what it did, and `planCommitPathspecWith`'s callers get the extra path only when the record exists or is tracked.

# Card 14 — settled-segment guard admits an amendment

**Edit:**
- `internal/loomcli#settledSegmentGuard` -> `func settledSegmentGuard(producers []shedengine.ProducerDef, reviewsDir string, publishFailure func() (verifytree.PublishFailure, bool, error), amendment func() (loomshed.AmendmentRecord, bool, error)) shedengine.GotoGuard`
- `internal/loomcli#loomCLI.settledGuard`
- `internal/loomcli#TestSettledSegmentGuard`

**Uses:**
- `internal/shedadapters#SettleRoundStart`
- `internal/loomshed#ReadAmendment`
- `internal/loomengine#LoomAmendmentRecord`
- `internal/loomshed#NamePlanBurlerReview`

**Intent:**
D7: the settled-segment goto guard gains one admission: a goto onto `Plan-Burler-Review` over a live Plan-Review settle record when the amendment record exists and its latest write time is after that settle's round start, or that settle has no known round start.

`settledSegmentGuard` takes an `amendment` reader; `settledGuard` passes one over `loomshed.ReadAmendment(loomengine.LoomAmendmentRecord(location))`.
After the existing live-record and Publish-failure checks, and before the refusal, when `target` is `loomshed.NamePlanBurlerReview` it reads the record; when one exists it reads `shedadapters.SettleRoundStart` of the segment's run directory and admits when the start is unknown or `record.WrittenAt` is after it, comparing instants.
A read error of either is returned, which Shed turns into a refusal, as the guard's other reads are.
Every other target, and this target with no record or a record written before the judged round started, is refused as today, with today's text and way forward.

What it lets through, and the bound: it re-opens only Plan-Review, only for an amendment written after its last judged round started or over a settle with no known round start; the fix still passes the plan gate (`done-card-edited` included) and a Plan-Bouncer judgment before Webster sees it.
An amendment appended while a round runs is after that round's start, so it is admitted again after the round's settle.
`WrittenAt` keeps full precision against a round start floored to the second (card 12), so a write in the start's own second reads as after it and the guard admits: the floor costs at most an extra round and never loses an amendment.

`TestSettledSegmentGuard` gains rows, each with a live Plan-Review settle record carrying a whole-second round start: an amendment written after the start admits `Plan-Burler-Review`; one written before it is refused; an amendment over a settle with no round start (a round-0 settle) admits; no amendment record is refused as today; an amendment record does not admit `Plan-Burler-Fix` or another segment's review row; an unreadable amendment record is a returned error.
Existing rows pass a reader answering no record.

**ImpactSummary:** `settledSegmentGuard` has one production caller, `settledGuard`, and one test, both covered here; the guard serves loom and darn, and the darn recipe has no `Plan-Burler-Review` row, so its behavior is unchanged.

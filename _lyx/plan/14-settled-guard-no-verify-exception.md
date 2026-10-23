# Card 14 — settled-guard-no-verify-exception

**Edit:**
- `internal/loomcli#loomCLI.settledGuard`
- `internal/loomcli#settledSegmentGuard` -> `func settledSegmentGuard(producers []shedengine.ProducerDef, reviewsDir string, amendment func() (loomshed.AmendmentRecord, bool, error)) shedengine.GotoGuard`
- `internal/loomcli#TestSettledSegmentGuard`

**Extends:**
- `internal/loomcli/doc.go`

**Intent:** Remove the settled-segment guard's exception for a verify failure (D11): no record target routes a failure to a settled review row any more, so a goto onto a settled segment's review or fix row is refused whatever failure record exists.
`settledSegmentGuard` drops its failure-record reader argument and the admission it fed; the Plan-Burler-Review amendment admission and every other rule stay.
`settledGuard` stops building the reader.
`TestSettledSegmentGuard` loses its failure-record admission rows; its existing settled-refusal row pins the refusal that now meets every goto onto a settled segment's review or fix row, since the guard takes no failure-record input any more.

`doc.go` drops the exception from the goto guard's description.

**ImpactSummary:** One guard argument and its admission go; the guard's sole caller is `settledGuard`, and the refusal text is unchanged.

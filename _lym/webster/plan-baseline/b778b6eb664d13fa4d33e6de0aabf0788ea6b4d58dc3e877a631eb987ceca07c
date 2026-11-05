# Card 26 — report-raddle

**Create:**
- `plan:internal/boardengine#ParseRaddleStoreEntry` -> `func ParseRaddleStoreEntry(data []byte) (RaddleStoreEntry, error)`
- `plan:internal/reportengine#RaddleRow` -> `type RaddleRow struct`
- `plan:internal/reportengine#RaddleReport` -> `type RaddleReport struct`
- `plan:internal/reportengine#ComputeRaddle` -> `func ComputeRaddle(src Sources) (RaddleReport, error)`
- `plan:internal/reportengine#TestComputeRaddle` -> `func TestComputeRaddle(t *testing.T)`
- `plan:internal/reportcli#raddleCommand` -> `func raddleCommand() *cobra.Command`

**Edit:**
- `plan:internal/boardengine#ReadRaddleStore`
- `internal/reportengine#allowedImports`
- `internal/reportcli#Command`
- `internal/reportcli#TestReportVerbs_RefuseBadFlags`
- `internal/reportcli/doc.go`
- `internal/reportengine/doc.go`
- `//docs/overview.md`

**Uses:**
- `plan:internal/raddleparser#Parse`
- `internal/shedrun#ParseRunUsage`
- `internal/shedrun#UsageDirRel`
- `internal/reportengine#Sources`
- `internal/reportengine#world`
- `plan:internal/reportengine#usageFiles`
- `plan:internal/reportcli#costCommand`
- `internal/boardengine/doc.go`

**Intent:** D14, `lym report raddle`, the record the raddle stencil is tuned from.
`ParseRaddleStoreEntry` decodes one `store.json`, and `ReadRaddleStore` reads through it, so the store's record has one decoder.
`ComputeRaddle` lists every store directory on the board branch at its tip (`Sources.Board`, `SubdirsAtRevision` of the store root) and gives per Sketch a `RaddleRow`: the Sketch slug, the predicted cost (the approved proposal's `cost.expected_usd`, parsed by `raddleparser.Parse`, null when it is), the actual cost (the summed session costs of the filed parts' runs, each joined by its slug to an archive tag and read with `shedrun.ParseRunUsage`; a part's run with an unknown-cost session makes the actual cost partial and says so), the filed parts not yet landed (no archive tag), and whether the approved proposal differs from the cold one (their hashes differ).
A store file that does not decode is a finding naming the Sketch, and the row is skipped.
`raddleCommand` is `lym report raddle [--format json|markdown]`, prime-only like the other verbs; `Command` adds it and the group's `Long` text names it.
`allowedImports` gains `internal/raddleparser`.
`TestComputeRaddle` (tier0, the `world` fake with a fixture store on the board branch and usage files through `usageFiles`) has rows: a proposal whose parts all landed (predicted against actual); one with an unlanded part (listed); one the operator changed (the cold and approved differ) and one unchanged; a part run with an unknown-cost session (partial); an undecodable `store.json` (a finding).
`TestReportVerbs_RefuseBadFlags` gains the `raddle` `--format` row.
`docs/overview.md`'s report lines and the two `doc.go` files name the verb.

**ImpactSummary:** A new report verb over the board branch; the store read gains one shared decoder, and no write path changes.

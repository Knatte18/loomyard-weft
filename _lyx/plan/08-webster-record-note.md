# Card 8 — webster-record-note

**Create:**
- `plan:internal/loomshed#WebsterRecordNote` -> `func WebsterRecordNote(anchorPath string) string`

**Edit:**
- `internal/loomshed/publishfailure_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/websterengine#DoneCards`
- `internal/websterengine#State`

**Intent:** Render webster's run record for the Plan-Review rubric the way `PublishFailureNote` renders the Publish failure record (D2).
`WebsterRecordNote` loads the state under `anchorPath` through `websterengine.LoadState`; with no record it returns `none`.
With one it renders markdown naming the begun batches with their status and card ids, the done cards, and the rule that a done card is frozen and a finding against it lands as a follow-up card after the last begun batch with its Card Index line in `00-overview.md`.
A record that cannot be read is logged at Warn and renders `none`, as an unreadable Publish failure record does, so a rubric read never fails on it.
A new `TestWebsterRecordNote` in `publishfailure_test.go` pins `none` without a record and the begun batches, done cards and follow-up rule with one; the loomshed row of `test-budget.yaml` rises by one.

**ImpactSummary:** One new exported function with no production caller until card 9; the only edits are a test file and a budget row.

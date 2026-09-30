# Card 2 — audit-ledger

**Edit:**
- `internal/websterengine/state.go`
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/beginbatch_test.go`

**Create:**
- `internal/websterengine/auditledger.go`
- `internal/websterengine/auditledger_test.go`

**Uses:**
- `internal/websterengine/audit.go`

**Intent:**
Add the durable ledger D5 and D8 disposition findings into, so a finding refuses or warns at most once per run.

`auditledger.go` declares:

- `AuditWarning` (`Identity`, `Class`, `Detail`, with camelCase JSON tags), one recorded warning.
- The disposition values `dispositionWarned` ("warned") and `dispositionFailed` ("failed").
- `findingIdentity(sessionID string, v AuditViolation) string`: a parent finding's identity is `<sessionID>/<v.Key>`, because parent ordinals restart in every Master session;
  a fork finding's identity is `v.Key`, since the transcript path is already unique.
- `isDispositioned(st *State, id string) bool`.
- `recordBatchWarning(st *State, bs *BatchState, id, class, detail string) (text string, added bool)`: records the disposition and appends one `AuditWarning` to `bs.AuditWarnings`, returning `added == false` and no append when `id` is already dispositioned.
  It takes the class and detail as strings rather than an `AuditViolation`, so card 10's later-card drift warning (D9, class `later-card-drift`) records through the same path as an audit finding's `string(v.Class)` and `v.Detail`.
- `recordRunWarning(st *State, id, class, detail string) (text string, added bool)`: the same, appending to the run-level list.
- `recordFailedFinding(st *State, id string)`: records `dispositionFailed`.
- `auditWarningText(w AuditWarning) string`: the envelope line, `audit warning (<class>): <detail>`.
- `RecordedAuditWarnings(st *State, batches []batcher.Batch) []string`: every recorded warning's text, batch warnings in `batches` order first, run-level warnings last.

`state.go` gains `State.AuditDispositions map[string]string` (`auditDispositions,omitempty`), `State.AuditWarnings []AuditWarning` (`auditWarnings,omitempty`, the run-level list) and `BatchState.AuditWarnings []AuditWarning` (`auditWarnings,omitempty`), each with a doc comment naming D5/D8's once-per-identity rule.
A state.json written before these fields decodes with them empty.

A recorded batch warning must outlive the record it was written on, because its identity stays dispositioned and no later call records it again.
`BeginBatch` and `recoverSpawn` both replace the batch's record with a fresh `BatchState`: a re-begin after a no-report call (Master's re-fork rung), and a recovery after a failed batch.
Each new record inherits the prior record's `AuditWarnings`, beside the `StartSHA` inheritance both already do, with a comment naming the once-per-identity rule as the reason.

`beginbatch_test.go` (integration, the existing begin fixture) adds `TestBeginBatch_ReBeginKeepsAuditWarnings`: a begun record carrying one `AuditWarning`, re-begun by `BeginBatch`, still carries it.

`auditledger_test.go` (untagged, `package websterengine`) covers: a repeated record of one identity adds nothing; the same parent `Key` under two session IDs is two identities; `recordFailedFinding` makes `isDispositioned` true without adding a warning; `RecordedAuditWarnings` orders by the given batches then run-level; and a `SaveState`/`LoadState` round trip keeps all three new fields, while a legacy state.json without them loads with nil maps and slices.

**ImpactSummary:** Additive omitempty fields on State/BatchState; existing state.json files decode unchanged and no current reader branches on them; begin-batch and recover-batch copy one more field from the prior record, empty on every record written before this card.

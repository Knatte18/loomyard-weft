# Card 22 — Publish failure record trace fields

**Create:**
- `plan:internal/verifytree#CommitRange` -> `type CommitRange struct`
- `plan:internal/verifytree#TracedCard` -> `type TracedCard struct`
- `plan:internal/verifytree#FailureTrace` -> `type FailureTrace struct`

**Edit:**
- `internal/verifytree#PublishFailure`
- `internal/verifytree#TestPublishFailureRecord`
- `internal/verifytree/doc.go`

**Intent:**
D9: the Publish failure record gains the trace's fields, all optional (PATTERN-tolerant-records).

`CommitRange` holds `From` and `To`, full commit names (`from`, `to`); its zero value is the empty range.
`TracedCard` holds `Number int` (`number`), `Glyphs []string` (`glyphs`) and `Locations []string` (`locations`, each `path:line`) matched to it, `Batch int` (`batch`, 0 when Webster holds no record of the card) and `Range CommitRange` (`range`, omitted when empty).
`FailureTrace` holds `Cards []TracedCard`, `LaterRange CommitRange` and `Untraced []string`: what a trace returns and the record stores.
`PublishFailure` gains `Cards []TracedCard` (`cards,omitempty`), `LaterRange CommitRange` (`later_range,omitempty`) and `Untraced []string` (`untraced,omitempty`); no slug or intent text is stored, since the note reads them from the live plan.
`verifytree/doc.go` describes the fields and that an older record without them reads as before.

`TestPublishFailureRecord` gains rows: a record with cards, a later range and untraced locations round-trips through `WritePublishFailure` and `ReadPublishFailure`; a record without them writes no such keys; an older record's YAML without the keys reads with them empty.

**ImpactSummary:** `PublishFailure` gains three optional fields; every writer and reader that does not set them writes and reads the same YAML as before.

# Card 2 — takeover-entry

**Create:**
- `internal/shedengine#Takeover`
- `internal/lymdirs#HandOverDirName`
- `internal/testkit/takeoverkit#WriteTakeover`
- `internal/testkit/takeoverkit/doc.go`

**Edit:**
- `internal/shedengine#Status`
- `internal/shedengine#TestStatus_JSONRoundTrip`

**Extends:**
- `internal/shedengine/doc.go`
- `internal/lymdirs/doc.go`

**Uses:**
- `internal/testkit/doc.go`

**Intent:**
D2's takeover entry, written into a run's batten record and child record, is a field of the status file.
`Takeover` holds `Machine` (the new owner machine, json `machine`), `Subject` (the adopt subject that moved the run, json `subject`) and `At` (RFC 3339, json `at`).
`Status` gains `Takeover *Takeover` with json `takeover,omitempty`, outside-owned like `Cancel` and `PauseIntent`: Shed never writes it, and a record written before the field existed decodes as nil.
Shed's persist rewrites the status through `internal/state`, so the key survives a rewrite by an older binary (`PATTERN-tolerant-records`).
The production writer comes with adopt in a later part; this part reads it.

`HandOverDirName` names the directory under the hub's board-level `.lym` that holds this machine's hand-over marks, one per slug (D2).
Its doc comment says a pair removal does not take it, since it sits under the hub's `.lym` and never in a pair.
The lymdirs doc gains it beside the other hub-level names.

`internal/testkit/takeoverkit` is the takeover entry's test writer, which the decision record's exports name.
`WriteTakeover` reads the status JSON at statusPath as a generic object, sets its `takeover` key to entry and writes it back, keeping every other key.
It imports `encoding/json`, `os` and `internal/shedengine` only, so it carries none of the kit-denied imports; the caller commits and pushes the file.
Its package doc (`doc.go`) says it serves the tests of fabricengine and pairteardown that plant another machine's takeover on origin.
The shedengine doc's status-field section gains the takeover entry.

`TestStatus_JSONRoundTrip` (tier0) gains a row carrying a `Takeover` that round-trips and a row asserting a status without the key decodes `Takeover` nil.

**ImpactSummary:** Adds one optional status field, one directory constant and a new test kit; every existing status reader decodes the field as nil, and no production code reads it until card 5.

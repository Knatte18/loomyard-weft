# Card 6 — integration-fix-state

**Create:**
- `internal/websterengine#IntegrationFixState`

**Edit:**
- `internal/websterengine/state.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/runlevel_test.go`

**Intent:**
Record the run's one integration-fix attempt in state.json and make its strand reclaimable on resume (#332b, decisions D5 and D6).

In `state.go`, add `IntegrationFixState` with:
- `PreFixHead string`, the HEAD the attempt started from;
- `StrandGUID string`, empty until the strand has started;
- `SpawnedAt string`, RFC3339 UTC;
- `Result string`, empty while the attempt is in flight and the attempt's result once it ends.

`State` gains `IntegrationFix *IntegrationFixState` (`json:"integrationFix,omitempty"`).
A non-nil value means the run's one attempt is spent.
`run --fresh` already builds a new `State`, which resets it; say so on the field's doc comment.

`reclaimEntryTimeStrands` also stops `st.IntegrationFix.StrandGUID` through `removeStrandIfLive` when the record is non-nil and the GUID non-empty, so a run that crashed during the fix never leaves a strand committing behind a fresh Master.
Update its doc comment, which today names only Master's strand and recovery strands.

Tests in `runlevel_test.go`: a state recording a live fix strand has it removed at `Run` entry;
a dead or empty-GUID record removes nothing;
a state without the record behaves as before.

**ImpactSummary:** state.json gains one optional field; run's entry-time reclaim gains one more strand to stop.

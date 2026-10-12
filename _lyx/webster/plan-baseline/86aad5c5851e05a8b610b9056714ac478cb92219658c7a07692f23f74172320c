# Card 24 — status-paused-since

**Create:**
- `plan:internal/battencli#TestBattenStatus_CarriesPausedSinceForACoveredSlug` -> `func TestBattenStatus_CarriesPausedSinceForACoveredSlug(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.loomStatusExtras`
- `internal/loomcli#loomVerbTexts`
- `internal/loomcli#TestStatusCmd_EnvelopeKeySet`
- `internal/battencli#battenCLI.battenStatusExtras`
- `internal/battencli#battenVerbTexts`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Carry `paused_since` from the pause record in `lyx loom status` and `lyx batten status` for a covered run (D5), since no status file changes state for a pause and a covered run still reads `running` at its current producer.
`loomStatusExtras` and `battenStatusExtras` each read `pauserecord.ReadCoverage` for their run's slug, at `hubgeom.PauseGeometry` of the receiver's location (loom's task worktree, batten's prime), and add `paused_since` (the covering entry's `PausedAt`, RFC 3339) and `paused_by` (its scope) when covered; an uncovered run carries neither key, and a receiver with no location reads none.
An unreadable record adds neither and logs one Warn, so a status read never fails on the record.
Both verbs' `Long` help in `loomVerbTexts` and `battenVerbTexts` names the two keys.

Tests: `TestStatusCmd_EnvelopeKeySet` gains a `covered` field and a row whose receiver carries a location over a temp hub whose record holds a live entry for the run's slug: the envelope carries `paused_since` equal to the entry's time and `paused_by`, while the existing rows assert both keys absent.
`TestBattenStatus_CarriesPausedSinceForACoveredSlug` runs batten's status verb on a hand-populated receiver for a covered slug and an uncovered one and asserts the keys are present for the first only; the battencli budget row grows by one.

**ImpactSummary:** Two status-extras hooks gain one optional pair of envelope keys; no status file and no other verb changes.

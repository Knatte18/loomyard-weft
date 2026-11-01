# Card 1 — The pause record's hold caller

**Create:**
- `internal/pauserecord#HoldCaller`

**Edit:**
- `internal/pauserecord#Coverage`
- `internal/pauserecord#Record.CoverageOf`
- `internal/pauserecord#TestCoverageOf`

**Extends:**
- `internal/pauserecord/doc.go`

**Intent:**
Add `HoldCaller`, the fixed `Caller` of the pause entry `lym adopt` writes to hold a run it takes over until its subject's adopt check is green (D4), beside `OwnerCaller` in `record.go`.
Its doc comment says the entry holds a run, or a released task, adopt took over, and that only the green adopt check of the subject lifts it by itself.
An older binary reads the new value as any other caller's entry (PATTERN-tolerant-records): it covers its slug like every entry.

`Coverage` gains `Held bool`: set when a live entry of the slug's own scope carries `HoldCaller`, whichever entry is the oldest covering one, as `OwnerHeld` is for `OwnerCaller`.
`Record.CoverageOf` sets it in the same loop that sets `OwnerHeld`, for a non-empty slug only.

`TestCoverageOf` gains rows: a live `HoldCaller` entry of the slug's own scope sets `Held` and `Covered` and leaves `OwnerHeld` false; a hub entry, an entry of another slug's scope and a resumed `HoldCaller` entry leave `Held` false; an empty slug never reads `Held`.

`doc.go` gains the `HoldCaller` paragraph beside the `OwnerCaller` one: the hold-caller entry, `Coverage.Held`, and that a dobby's owner-pause lift and the hibernate wake, which judge only `OwnerCaller`, never treat it as theirs.

**ImpactSummary:** one new constant and one new Coverage field set by CoverageOf; no existing reader changes behaviour.

**Commit:** 1: pauserecord: a hold caller for the runs an adopt takes over

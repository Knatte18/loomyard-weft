# Card 11 — the plan amendment record

**Create:**
- `plan:internal/loomengine#LoomAmendmentRecordRel` -> `func LoomAmendmentRecordRel() string`
- `plan:internal/loomengine#LoomAmendmentRecord` -> `func LoomAmendmentRecord(l *lyxcwd.Location) string`
- `plan:internal/loomshed#AmendmentRecord` -> `type AmendmentRecord struct`
- `plan:internal/loomshed#ReadAmendment` -> `func ReadAmendment(path string) (AmendmentRecord, bool, error)`
- `plan:internal/loomshed#AppendAmendment` -> `func AppendAmendment(path, title string, at time.Time) error`
- `plan:internal/loomshed#RemoveAmendment` -> `func RemoveAmendment(path string) error`
- `plan:internal/loomshed#AmendmentSettledBy` -> `func AmendmentSettledBy(record AmendmentRecord, roundStart time.Time) bool`
- `plan:internal/loomshed#TestAmendmentRecord` -> `func TestAmendmentRecord(t *testing.T)`

**Uses:**
- `internal/loomengine#LoomDurableDirRel`
- `internal/loomengine#LoomDurableDir`

**Intent:**
D7's amendment record: tracked content at `_lyx/loom/amendment.yaml` (PATTERN-durable-vs-ephemeral-state), since the run's routing reads it and it must survive adoption on another machine.

`loomengine.LoomAmendmentRecordRel()` is `LoomDurableDirRel()` joined with `amendment.yaml`, and `LoomAmendmentRecord(l)` joins it onto `l.AnchorPath()`, beside `LoomDurableDir`; `internal/loomengine` stays the declarer of the path, and no caller names the segment.

`internal/loomshed` owns the record over told paths (PATTERN-told-geometry), in a new `amendment.go`:
`AmendmentRecord` holds `Titles []string` (`titles`), the title of every pending amending decision entry in append order, and `WrittenAt time.Time` (`written_at`), the latest write in UTC, which YAML renders in RFC 3339 at nanosecond precision.
`ReadAmendment(path)` decodes it, reporting false for an absent file; unknown keys are read and dropped (PATTERN-tolerant-records); a file that does not decode is an error naming the path and its way forward: restore the file from `HEAD` or remove it, then run `lyx fabric commit`, which a session lyx refuses the verb from reports as status: FAILED for the orch to run.
Every reader below, the guard, the commit seam and `decision add`, returns that error unchanged, so each refusal over a broken record carries the fix.
`AppendAmendment(path, title, at)` reads the record, appends `title`, sets `WrittenAt` to `at` in UTC, and writes the whole file, creating it and its directory when absent.
`RemoveAmendment(path)` removes the file; an absent file is no error.
`AmendmentSettledBy(record, roundStart)` reports whether an approved settle whose judged round started at `roundStart` carried the amendment: true exactly when `roundStart` is non-zero and after `record.WrittenAt`, comparing instants; a zero `roundStart`, a settle with no known round start, keeps the record.

`TestAmendmentRecord`, table-driven in package `loomshed`, asserts: an absent record reads not found; a first append creates the file with one title and the write time at nanosecond precision; a second append keeps the first title, adds the second and moves the time; a record with an unknown key reads; an undecodable file is an error naming the path and `lyx fabric commit`; a removal of an absent file is no error; and `AmendmentSettledBy` is true for a round start after the write, false for one before, for an equal instant and for a zero round start.

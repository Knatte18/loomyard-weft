# Card 11 — the plan amendment record

**Create:**
- `internal/loomengine#LoomAmendmentRecordRel`
- `internal/loomengine#LoomAmendmentRecord`
- `internal/loomshed#AmendmentRecord`
- `internal/loomshed#ReadAmendment`
- `internal/loomshed#AppendAmendment`
- `internal/loomshed#RemoveAmendment`
- `internal/loomshed#AmendmentSettledBy`
- `internal/loomshed#TestAmendmentRecord`

**Edit:**
- `internal/loomshed#loomshedAllowedImports`

**Uses:**
- `internal/loomengine#LoomDurableDirRel`
- `internal/loomengine#LoomDurableDir`

**Intent:**
D7's amendment record: tracked content at `_lyx/loom/amendment.yaml` (PATTERN-durable-vs-ephemeral-state), since the run's routing reads it and it must survive adoption on another machine.

`loomengine.LoomAmendmentRecordRel()` is `LoomDurableDirRel()` joined with `amendment.yaml`, and `LoomAmendmentRecord(l)` joins it onto `l.AnchorPath()`, beside `LoomDurableDir`; `internal/loomengine` stays the declarer of the path, and no caller names the segment.

`internal/loomshed` owns the record over told paths (PATTERN-told-geometry), in a new `amendment.go`; `loomshedAllowedImports` gains `gopkg.in/yaml.v3`, with a comment that the amendment record is YAML and the package still resolves no geometry:
`AmendmentRecord` holds `Titles []string` (`titles`), the title of every pending amending decision entry in append order, and `WrittenAt time.Time` (`written_at`), the latest write in UTC, which YAML renders in RFC 3339 at nanosecond precision.
`ReadAmendment(path)` decodes it, reporting false for an absent file; unknown keys are read and dropped (PATTERN-tolerant-records); a file that does not decode is an error naming the path and its way forward: the operator rewrites the file as a valid record, `titles` listing the pending amending decision entries' titles and `written_at` the current UTC time, or removes it when no amendment is pending, runs `lyx fabric commit`, then re-runs the refused step, which is the goto, `lyx loom resume` for a failed Plan-Bouncer commit seam, or the `decision add` call; a session lyx refuses the verb from reports status: FAILED.
A removed record admits no goto (card 14), so a goto that the broken record refused needs the rewrite.
Every reader below, the guard, the commit seam and `decision add`, returns that error unchanged, so each refusal over a broken record carries the fix.
`AppendAmendment(path, title, at)` reads the record, appends `title`, sets `WrittenAt` to `at` in UTC, and writes the whole file, creating it and its directory when absent.
`RemoveAmendment(path)` removes the file; an absent file is no error.
`AmendmentSettledBy(record, roundStart)` reports whether an approved settle whose judged round started at `roundStart` carried the amendment: true exactly when `roundStart` is non-zero and after `record.WrittenAt`, comparing instants; a zero `roundStart`, a settle with no known round start, keeps the record.
`WrittenAt` keeps full precision while the round start is floored to the second (card 12), so an amendment written in the second its round started reads as after the start and the record is kept: the floor costs at most an extra round and never loses an amendment.

`TestAmendmentRecord`, table-driven in package `loomshed`, asserts: an absent record reads not found; a first append creates the file with one title and the write time at nanosecond precision; a second append keeps the first title, adds the second and moves the time; a record with an unknown key reads; an undecodable file is an error naming the path, its rewrite or removal, `lyx fabric commit` and the re-run of the refused step; a removal of an absent file is no error; and `AmendmentSettledBy`, its round starts written as whole seconds, is true for a round start after the write, false for one before, for an equal instant, for a write later in the start's own second and for a zero round start.

**ImpactSummary:** one test-only allowlist gains `gopkg.in/yaml.v3`; the rest is new code with no caller until card 13.

# Card 2 — sampler-record

**Create:**
- `internal/samplerengine/record_test.go`
- `plan:internal/samplerengine#Record` -> `type Record struct`
- `plan:internal/samplerengine#ReadRecord` -> `func ReadRecord(path, lockPath string) (Record, bool, error)`
- `plan:internal/samplerengine#UpdateRecord` -> `func UpdateRecord(path, lockPath string, mutate func(*Record)) error`
- `plan:internal/samplerengine#EntryStamp` -> `func EntryStamp(history []shedengine.HistoryEntry, row string, predecessors []string) int`
- `plan:internal/samplerengine#TestRecord` -> `func TestRecord(t *testing.T)`
- `plan:internal/samplerengine#TestEntryStamp` -> `func TestEntryStamp(t *testing.T)`
- `plan:internal/shedrun#SamplerRecord` -> `func SamplerRecord(l *lymcwd.Location, runID string) string`
- `plan:internal/shedrun#SamplerRecordLock` -> `func SamplerRecordLock(l *lymcwd.Location, runID string) string`

**Edit:**
- `internal/shedrun#TestPathConstructors`

**Extends:**
- `internal/samplerengine/doc.go`

**Uses:**
- `internal/shedrun/doc.go`
- `internal/shedrun#PullRequestRecord`
- `internal/shedrun#PullRequestRecordLock`

**Intent:**
D5 and D6: the record the Sampler-Session row keeps its session in and the done flag `lym loom sampler-done` writes.
`shedrun.SamplerRecord` returns `sampler.json` under `ScratchDir(l, runID)` and `SamplerRecordLock` returns `sampler.json.lock` beside it, as `PullRequestRecord` and its lock do, so the record is never tracked and is named through `lymdirs` by shedrun alone (`PATTERN-durable-vs-ephemeral-state`).
`TestPathConstructors` gains a row for each.
`samplerengine.Record` holds `StrandGUID`, `StrandName`, `SessionID` and `Engine` (the session the row recorded) and `DoneStamp int`, the entry stamp a done flag carries, 0 meaning no flag; JSON keys snake_case.
`ReadRecord` reads it through `state.ReadJSON` under the lock, `found` false for an absent file, and tolerates an unknown key (`PATTERN-tolerant-records`).
`UpdateRecord` applies `mutate` to the record under the lock and writes it through `state`, starting from the zero `Record` when the file is absent, and creates the directory the file and lock live in.
`EntryStamp` counts the times `history` shows the run entering `row` from another row, scanning in order and tracking whether the run stands at `row` after each entry.
An entry with outcome `Done` or `skipped` whose producer is in `predecessors` (the rows whose `on_done` names `row`) is an entry, counted, and leaves the run at `row`.
An entry with outcome `goto` whose producer is `row` is counted only when the run did not already stand at `row`, and leaves it there; a goto onto another row leaves it elsewhere.
An entry of `row` with outcome `Done`, or a `Stuck` whose `RoutedTo` names another row, leaves it elsewhere; any other entry of `row` (a halt) keeps it at `row`; an entry of any other producer leaves it elsewhere.
So a goto onto the row while the run stood there, halted, or failed with no history entry (a producer error appends none, `internal/shedengine/run.go`'s `appendHistory`), is no entry.
A folded predecessor `Done` counts once per call it folds, `1 + Repeats`.
A halt, a failure and a resume write no entry from another row, so they leave the stamp unchanged; every return by a goto from another row raises it, whether or not the row ended Done before.
`record_test.go` carries `//go:build tier0`.
`TestRecord`: an absent file reads not found; `UpdateRecord` on an absent file writes the mutated record, which reads back equal; a second update keeps the fields it does not touch; a file with an unknown key reads without error.
`TestEntryStamp` is a table over histories with `row` `Sampler-Session` and predecessor `Record-Write`: empty history 0; `Record-Write` Done 1; that plus a `Sampler-Session` Stuck (a halt) still 1; plus a goto onto `Sampler-Session` right after its own Stuck still 1; a `Sampler-Session` Done, a `Describe` Stuck and a goto onto `Sampler-Session` 2; a `Sampler-Session` Stuck then a goto to `Describe` then a goto back onto `Sampler-Session` with no Done between 2; a goto right after a `Sampler-Session` Done (the run had left the row) 2; a `Record-Write` Done then a goto onto `Sampler-Session` with nothing between (the row failed with no history entry) still 1; a `Record-Write` Done entry folding two calls 2.
`samplerengine/doc.go` gains a paragraph on the record, its done flag and the entry stamp.

**ImpactSummary:** adds two shedrun path accessors beside the pull-request ones and a test row for each; the samplerengine additions have no caller yet.

**Commit:** 2: samplerengine: the Sampler-Session record, its done flag and entry stamp

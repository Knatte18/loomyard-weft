# Card 1 — halt-note-leaf

**Create:**
- `internal/haltnote#NoteID`
- `internal/haltnote#Note`
- `internal/haltnote#Row`
- `internal/haltnote#Render`
- `internal/haltnote#Parse`
- `internal/haltnote#IsNoteName`
- `//internal/haltnote/doc.go`
- `//internal/haltnote/haltnote_test.go`
- `//internal/haltnote/leaf_enforcement_test.go`
- `//internal/haltnote/testmain_test.go`

**Edit:**
- `internal/loomcli#writeHaltNote`
- `//PATTERN.md`
- `//pattern/PATTERN-sole-parsers.md`
- `//pattern/PATTERN-leaf-packages.md`
- `//cmd/lyx/testdata/test-budget.yaml`
- `//internal/friction/doc.go`

**Intent:**
D7 moves the halt-note format out of `internal/loomcli` into a new leaf package, `internal/haltnote`, so `reportengine` (card 11) can parse notes without importing a cli package.

`internal/haltnote` imports the standard library only; its production code lives in `haltnote.go` and its package doc in `doc.go`.
`Note` carries `Producer`, `State`, `Reason`, `Anomaly`, `HistoryEntries` (the `history_entries:` count), `TraceFile`, `History []Row` and `ParentStatus`, every field a string or int, so the leaf needs no `shedengine` type.
`Row` is one history row: `Producer`, `Outcome`, `At`.
`Render` produces exactly the bytes `writeHaltNote` writes today: the `loom halted <state> at <producer>` title, the fenced reason, the `producer:`, `state:`, optional `anomaly:` (omitted when `Anomaly` is empty), `history_entries:` and `trace_file:` lines, the `history:` section (`- <producer> / <outcome> / <at>` rows or `no history entries`), and the `parent fabric status:` section only when `ParentStatus` is non-empty.
`Parse` is the tolerant reader the report uses (D5): it reads the field lines and the history rows back into a `Note`, skips any line it does not know, and returns an error only when the `producer:` or `state:` line is missing.
`IsNoteName` reports whether a friction-dir file name is a halt note: `loom-halt.md` or `loom-halt-<N>.md`, the names `friction.NotePath(dir, NoteID)` hands out.

`writeHaltNote` keeps its signature and its no-op on an empty directory; it maps its `haltNote` onto a `haltnote.Note`, setting `Anomaly` from `loomengine.ClassifyHalt` and `HistoryEntries` from `len(n.History)`, takes its path from `friction.NotePath(frictionDir, haltnote.NoteID)` and writes `haltnote.Render`'s bytes.
The bytes on disk do not change, so `TestWriteHaltNote_NamesAnomalyProducerReasonAndHistory`, `TestLoomPostRun_HaltNotes` and the `cli_test.go` halt assertions keep passing unchanged.

Tests in `haltnote_test.go`, one table-driven `TestRenderParse` whose rows each render a `Note` and parse the bytes back:
- a blocked note with an anomaly and two history rows round-trips every field, and its rendered bytes equal a literal golden string, which pins the format `writeHaltNote` writes;
- a note with no history renders `no history entries` and parses back an empty `History` with `HistoryEntries` 0;
- a note with a parent status round-trips it;
- bytes with an extra unknown `foo: bar` line parse with the known fields intact;
- bytes missing the `state:` line, and bytes missing the `producer:` line, are each an error.
A second test, `TestIsNoteName`, is a table over `loom-halt.md` and `loom-halt-2.md` (true) and `reflection-report.md`, `loom-halt.txt`, `01-plan-write.md` and `loom-halt-x.md` (false).
`TestLeafInvariant_AllowlistOnly` in `leaf_enforcement_test.go` asserts the empty non-stdlib import allowlist through `scankit.AssertImportAllowlist`, as `internal/discussionparser` does.
The same file holds `TestSoleParser_NoOtherProductionFileSpellsTheNoteTitle`: a `scankit` walk over `internal/` that fails when a non-test `.go` file outside `internal/haltnote` holds the string literal `"loom halted "`, the title only the halt note spells; its field names are not scanned, since `loomcli`'s crash-resume note in `entryobservation.go` shares `history_entries:` and `trace_file:`.
`loomcli`'s `writeHistoryRows` stays, for the crash-resume note alone; `haltnote.Render` writes the same `- <producer> / <outcome> / <at>` row shape for the halt note, and the golden row above pins it.
`testmain_test.go` runs the package under `tmuxkit.Main`.

PATTERN changes in this commit (D7 and the record's Constraints):
`pattern/PATTERN-leaf-packages.md` gains an `internal/haltnote` section: imports only the standard library.
`pattern/PATTERN-sole-parsers.md` gains a `## Halt-note format (internal/haltnote)` section: `internal/haltnote` is the sole renderer and parser of the Go-authored halt note `loomcli` writes into the loom friction directory, and it imports the standard library only.
The `PATTERN-sole-parsers` line in `PATTERN.md` becomes:

```markdown
- `PATTERN-sole-parsers` — Parsing plan, discussion, summary, recipe or halt note: only `planparser`, `discussionparser`, `summaryparser`, `shedbuild`, `haltnote`. (test) — [background](pattern/PATTERN-sole-parsers.md)
```

`cmd/lyx/testdata/test-budget.yaml` gains the row `internal/haltnote: 4`.

`internal/friction/doc.go`'s "Go-authored notes" paragraph adds that the loom halt note's format is owned by `internal/haltnote`.

**ImpactSummary:** `writeHaltNote` keeps its signature and its on-disk bytes, so its one production caller, `reflectHalt`, and every halt-note test are unaffected; the new package has no importer yet.

**Commit:** `1: halt-note leaf package`

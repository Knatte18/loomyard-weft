# Card 3 — planparser-restore

**Create:**
- `internal/planparser#RestoreResult`
- `internal/planparser#RestoreRenumber`
- `internal/planparser#ErrRenumberNoJournal`
- `internal/planparser#ErrRenumberJournalInvalid`
- `internal/planparser#ErrRenumberRestoreFailed`
- `internal/planparser#readRenumberJournal`

**Edit:**
- `internal/planparser/renumber_test.go`
- `internal/planparser/doc.go`
- `pattern/PATTERN-sole-parsers.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#RenumberIO`
- `internal/planparser#renumberJournal`
- `internal/planparser#reverseRenumber`
- `internal/planparser#ErrRenumberIOFailed`
- `internal/planparser#cardFileName`

**Intent:** Add `RestoreRenumber`, the journal's only reader, which card 4's `restore-renumber` verb calls after an interrupted `InsertCard` or `RemoveCard` left the journal on disk.

`readRenumberJournal` reads `journalPath` through `ops.ReadFile`: a missing file is found false (a `.tmp` file alone is ignored), a file that does not decode as `renumberJournal` is `ErrRenumberJournalInvalid`, and any other read error is `ErrRenumberIOFailed` wrapping the cause, with nothing written, so it joins the loom raw I/O row.
`RestoreRenumber` refuses with `ErrRenumberNoJournal` when none is found, writing nothing.
It then holds every name the journal lists, in `Files` and in `Created`, to the plan directory's top level: a name is a base name with no `/` or `\` and is neither `.` nor `..`, and is either `00-overview.md` or a card file name, one or more digits, a hyphen, a non-empty slug of non-whitespace characters and `.md`, the shape `cardFileName` renders for any slug the Card Index parser accepts, with no `cardSlugRe` check, since the plan format imposes no slug rule on existing cards; any other name, `../x.md`, `archive-1/01-a.md` or `round-1/prior-generation/01-a.md` included, is `ErrRenumberJournalInvalid` with nothing written, so a corrupt journal never reaches an archive or escapes the directory.

A valid journal is applied through `reverseRenumber`, every `Created` name removed and every `Files` entry written back, and the journal is removed last; `RestoreResult` holds `Removed` and `Restored`, the names in each set.
A raw I/O failure partway is `ErrRenumberRestoreFailed` wrapping the cause, with the journal kept; the function is idempotent, since `reverseRenumber` skips a `Created` name already gone and rewrites every `Files` entry whole, so a re-run completes the restore.
The restore discards any edit made to the journaled files since the interrupted call, which the doc comment states as the bound's cost.

Tests join `renumber_test.go` as `TestRestoreRenumber`, a table over card 2's fixtures, `archive-1/` and `round-1/prior-generation/` subtrees included: a journal left by a failing reversal of an insert and of a remove, each restoring the directory byte-identical to before the interrupted call, the removed card's bytes, the moved cards' headings and `Commit:` prefixes and the unchanged archive and prior-generation subtrees included, and removing the journal; a journal left over a plan whose moved card's slug is outside `cardSlugRe`, restored the same way; no journal, an undecodable journal, a journal whose read fails with an error other than `fs.ErrNotExist`, and a journal naming each outside path above, each refusing with its error and writing nothing; a temporary journal file alone, read as no journal; and a failure injected into a write partway, which keeps the journal, after which a re-run leaves the directory byte-identical.
`internal/planparser` is at its test budget after card 2, so raise its row by one.

`doc.go`'s write-path list and scratch paragraph gain `RestoreRenumber` and the journal's restore, and `pattern/PATTERN-sole-parsers.md`'s write-path line names `RestoreRenumber` beside the two card 2 adds.

**ImpactSummary:** One new read-and-write path over the journal card 2 introduced; no existing caller changes, and the sole-parsers background names one more writer.

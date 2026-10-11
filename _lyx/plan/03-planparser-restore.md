# Card 3 — planparser-restore

**Create:**
- `plan:internal/planparser#RestoreResult` -> `type RestoreResult struct`
- `plan:internal/planparser#RestoreRenumber` -> `func RestoreRenumber(ops RenumberIO, planDir, journalPath string) (*RestoreResult, error)`
- `plan:internal/planparser#ErrRenumberNoJournal` -> `var ErrRenumberNoJournal error`
- `plan:internal/planparser#ErrRenumberJournalInvalid` -> `var ErrRenumberJournalInvalid error`
- `plan:internal/planparser#ErrRenumberRestoreFailed` -> `var ErrRenumberRestoreFailed error`
- `plan:internal/planparser#readRenumberJournal` -> `func readRenumberJournal(ops RenumberIO, journalPath string) (renumberJournal, bool, error)`

**Edit:**
- `internal/planparser/renumber_test.go`
- `internal/planparser/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/planparser#RenumberIO`
- `plan:internal/planparser#renumberJournal`
- `plan:internal/planparser#reverseRenumber`
- `internal/planparser#cardFileName`

**Intent:** Add `RestoreRenumber`, the journal's only reader, which card 4's `restore-renumber` verb calls after an interrupted `InsertCard` or `RemoveCard` left the journal on disk.

`readRenumberJournal` reads `journalPath` through `ops.ReadFile`: a missing file is found false (a `.tmp` file alone is ignored), and a file that does not decode as `renumberJournal` is `ErrRenumberJournalInvalid`.
`RestoreRenumber` refuses with `ErrRenumberNoJournal` when none is found, writing nothing.
It then holds every name the journal lists, in `Files` and in `Created`, to the plan directory's top level: a name is a base name with no separator, and is either `00-overview.md` or a card file name (`NN-<slug>.md`, `cardSlugRe` on the slug); any other name, `../x.md`, `archive-1/01-a.md` or `round-1/prior-generation/01-a.md` included, is `ErrRenumberJournalInvalid` with nothing written, so a corrupt journal never reaches an archive or escapes the directory.

A valid journal is applied through `reverseRenumber`, every `Created` name removed and every `Files` entry written back, and the journal is removed last; `RestoreResult` holds `Removed` and `Restored`, the names in each set.
A raw I/O failure partway is `ErrRenumberRestoreFailed` wrapping the cause, with the journal kept; the function is idempotent, so a re-run completes the restore.
The restore discards any edit made to the journaled files since the interrupted call, which the doc comment states as the bound's cost.

Tests join `renumber_test.go` as `TestRestoreRenumber`, a table over: a journal left by a failing reversal of an insert and of a remove, each restoring the directory byte-identical to before the interrupted call, the removed card's bytes and the moved cards' headings and `Commit:` prefixes included, and removing the journal; no journal, an undecodable journal, and a journal naming each outside path above, each refusing with its error and writing nothing; a temporary journal file alone, read as no journal; and a failure injected into a write partway, which keeps the journal, after which a re-run leaves the directory byte-identical.
`internal/planparser` is at its test budget after card 2, so raise its row by one.

`doc.go`'s write-path list and scratch paragraph gain `RestoreRenumber` and the journal's restore.

**ImpactSummary:** One new read-and-write path over the journal card 2 introduced; no existing caller changes.

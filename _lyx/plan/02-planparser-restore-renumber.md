# Card 2 — planparser-restore-renumber

**Create:**
- `plan:internal/planparser#ErrRenumberNoJournal` -> `var ErrRenumberNoJournal error`
- `plan:internal/planparser#ErrRenumberJournalCorrupt` -> `var ErrRenumberJournalCorrupt error`
- `plan:internal/planparser#ErrRenumberJournalPathOutside` -> `var ErrRenumberJournalPathOutside error`
- `plan:internal/planparser#ErrRenumberRestorePartial` -> `var ErrRenumberRestorePartial error`
- `plan:internal/planparser#RestoreRenumber` -> `func RestoreRenumber(planDir, journalPath string, ops RenumberOps) error`
- `plan:internal/planparser_test#TestRestoreRenumber` -> `func TestRestoreRenumber(t *testing.T)`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/planparser#RenumberOps`
- `plan:internal/planparser#InsertCard`
- `plan:internal/planparser#RemoveCard`
- `plan:internal/planparser#ErrRenumberReversalFailed`

**Intent:**
Add `RestoreRenumber`, the journal's only reader on disk (D2), in `internal/planparser/renumber.go`, reusing card 1's reversal helper.
It restores the plan directory to its state before an interrupted `InsertCard` or `RemoveCard`, discarding any edit made since to the files the journal names; that is the bound's cost (D2).

It refuses, writing nothing:
- `ErrRenumberNoJournal` when no file exists at `journalPath` (inputs: a clean plan directory, and one with only a stale `journalPath + ".tmp"` beside it, which is ignored).
- `ErrRenumberJournalCorrupt` when the journal does not decode as the journal's JSON (input: a journal file holding `not json`); since card 1's write is atomic, only an outside edit reaches it.
- `ErrRenumberJournalPathOutside` when any journaled file or created name is not a top-level plan file name: it contains `/` or `\`, is `.` or `..`, or is neither `00-overview.md` nor a name matching `^\d+-.+\.md$` (inputs: a decodable journal naming `../escape.md`, and one naming `archive-1/01-a.md`).

The restore removes every journaled created name (a missing one is not an error), writes back every journaled file's pre-call bytes under its base name inside `planDir`, then removes the journal.
A raw failure partway through returns an error wrapping `ErrRenumberRestorePartial` and keeps the journal; the steps are idempotent, so a re-run completes the restore.
Where the journal holds the name of a file now under `archive-*/` or `round-<N>/`, those subdirectories are still never written, since every write joins a validated base name onto `planDir`.

`TestRestoreRenumber`, in `internal/planparser/renumber_test.go`, covers in rows:
- after a reversal failure injected into `InsertCard` and into `RemoveCard` (card 1's `RenumberOps` seam), `RestoreRenumber` leaves the plan directory byte-identical to its pre-call snapshot, a removed card's bytes and the moved cards' headings and `Commit:` prefixes included, with the journal gone;
- the three refusals above, each asserting `errors.Is` and the plan directory and journal byte-identical; the path-outside journals are written by hand in card 1's fixed JSON shape (`files` and `created` keys);
- a failure injected at a write-back asserting `ErrRenumberRestorePartial` and the journal kept, then a re-run with the zero `RenumberOps` leaving the directory byte-identical to the pre-call snapshot;
- a file under `archive-1/` and under `round-1/prior-generation/` byte-identical after a restore.

`cmd/lyx/testdata/test-budget.yaml` raises `internal/planparser` by one.

**ImpactSummary:** additive: one exported function and four sentinels in planparser beside card 1's, no existing caller changed.

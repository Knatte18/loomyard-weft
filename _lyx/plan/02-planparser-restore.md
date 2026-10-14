# Card 2 — planparser-restore

**Create:**
- `plan:internal/planparser#RestoreResult` -> `type RestoreResult struct`
- `plan:internal/planparser#ErrNoRenumberJournal` -> `var ErrNoRenumberJournal error`
- `plan:internal/planparser#ErrRenumberJournalUndecodable` -> `var ErrRenumberJournalUndecodable error`
- `plan:internal/planparser#ErrRenumberJournalOutside` -> `var ErrRenumberJournalOutside error`
- `plan:internal/planparser#ErrRestoreIncomplete` -> `var ErrRestoreIncomplete error`
- `plan:internal/planparser#Renumber.Restore` -> `func (r Renumber) Restore() (RestoreResult, error)`
- `plan:internal/planparser_test#TestRenumberRestore` -> `func TestRenumberRestore(t *testing.T)`

**Edit:**
- `internal/planparser/doc.go`
- `pattern/PATTERN-sole-parsers.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/testkit/plankit#WriteTree`
- `plan:internal/planparser#Renumber`
- `plan:internal/planparser#RenumberFS`
- `plan:internal/planparser#Renumber.InsertCard`
- `plan:internal/planparser#Renumber.RemoveCard`
- `plan:internal/planparser#ErrRenumberHalted`
- `plan:internal/planparser#ErrRenumberJournalPresent`
- `internal/testkit/plankit#Write`

**Intent:**
Add `Renumber.Restore` to `internal/planparser/renumber.go`: the only reader of the renumber journal Card 1's verbs leave when their reversal fails or the process is killed mid-apply (decision record D2, `restore-renumber`).
It shares Card 1's reversal routine, so the in-call reversal and the restore apply the same steps.

Refusals, each before any write:

- `ErrNoRenumberJournal`: no file at `JournalPath`; reached by a row on a clean plan directory.
- `ErrRenumberJournalUndecodable`: the journal is not valid JSON of the journal's shape, or lacks its files map; the atomic temp-and-rename write rules out a partial journal, so only an outside edit reaches it, and a row reaches it by writing `{` to `JournalPath`.
- `ErrRenumberJournalOutside`: a journaled file name or `created` name is not a base name (it holds a path separator, or is `.` or `..`) or is neither `00-overview.md` nor a card file name matching `^[0-9]+-[^/\\]+\.md$`; reached by rows naming `../x.md`, `archive-1/01-a.md` and `notes.txt`, each decoded before any write, so a corrupt journal never reaches `archive-*/`, `round-<N>/prior-generation/` or outside `PlanDir`.

`Restore` then removes every `created` name (an absent one is not an error), writes back every journaled file's pre-call bytes, and removes the journal, discarding any edit made to those files since the interrupted call, which is the only state that matches the run record the begun-card bound was checked against.
`RestoreResult` carries `Removed` and `Restored`, the base names it removed and wrote back, sorted.
A raw I/O failure at any of those writes, the journal's removal included, returns an error wrapping `ErrRestoreIncomplete` and keeps the journal: some files may already be rewritten, so it is not the grouped "nothing is mutated" case, and a re-run converges because every step is idempotent.
A raw failure reading the journal, before any write, is returned unwrapped by a sentinel, the grouped raw-I/O case.
The new edge is bounded by the name check above: it writes and removes only base names the journal lists, inside `PlanDir`.

`TestRenumberRestore`, a new table-driven test in `renumber_test.go` reusing Card 1's failing `RenumberFS` fake, has rows:

- after an `InsertCard` and after a `RemoveCard` whose move and then reversal write were failed (`ErrRenumberHalted`), `Restore` with a nil `FS` leaves the plan directory byte-identical to the snapshot taken before the interrupted call, including the deleted card's bytes and the moved cards' headings and `Commit:` prefixes, and no journal remains;
- each refusal above, asserting `errors.Is` on its sentinel and that the plan directory and the journal are byte-identical; the outside-path rows put the bad name once in the files map and once in `created` beside a valid files map, the `created` row naming `../outside.md` next to a real file at that path, which must still exist afterwards, since `created` names are the ones `Restore` removes;
- a restore whose first write-back is failed returns `ErrRestoreIncomplete` with the journal still present, and a second `Restore` with a nil `FS` leaves the directory byte-identical to the pre-call snapshot and removes the journal;
- an `archive-x/` and `round-1/prior-generation/` subtree stays byte-identical through the restore.

One new top-level test takes `internal/planparser` past its row, so `test-budget.yaml` raises it by one.
`doc.go`'s `# Card renumbering` section gains a sentence naming `Restore` as the journal's only reader, and the package comment's write-path sentence and `PATTERN-sole-parsers.md`'s write-path bullet each add `Renumber.Restore` (journal restore) beside Card 1's two methods.

**ImpactSummary:** additive to Card 1's file: one method, its result type and four sentinels; `doc.go`'s and `PATTERN-sole-parsers.md`'s write-path lists and the budget row are the only other edits.

**Commit:** 2: planparser: restore a plan directory from the renumber journal

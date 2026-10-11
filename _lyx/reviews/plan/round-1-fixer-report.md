# Plan-Review round 1 — fixer report

Every finding was checked against the code and the decision record, and every one held.
All ten are fixed; `lyx loom validate-plan` reports only the informational `create-new-unit` findings after the edits.

## Fixed

- **F1** (journal directory never created): card 2's `RenumberIO` gains a `MkdirAll` field, and `writeRenumberJournal` creates the journal's parent directory before the temporary write, a failure joining `ErrRenumberIOFailed` with the plan directory untouched.
  Card 2's fixtures pass a `journalPath` whose parent does not exist yet, `TestRenumber_IOFailureReversed` gains the directory-creation step, and card 4's fixtures use `newRenumberDeps`' real path shape under a temp anchor with no scratch directories.
  The overview's `raw-io-seam` decision now says five function fields.
  Verified: `websterengine.AcquireStateMutation` creates only `.lyx/webster` (`internal/websterengine/state.go:104`), and nothing creates `.lyx/plan`.
- **F2** (restore's slug rule stricter than the format): card 3 holds a journaled card name to digits, a hyphen, a non-empty non-whitespace slug and `.md`, the shape `cardFileName` renders for any slug `cardIndexLineRe` accepts, with no `cardSlugRe` check.
  `TestRestoreRenumber` gains a row restoring a journal over a slug outside `cardSlugRe`.
- **F3** (plural and list prose mentions missed): card 2's scan matches `[Cc]ards?` followed by one number or a list or range, reporting the line when any number, a range's interior included, is an old number of a moved card.
  The success test's fixture carries an `Applies to: cards …` line.
- **F4** (PATTERN-sole-parsers omits `RestoreRenumber`): card 3 lists `pattern/PATTERN-sole-parsers.md` under `**Edit:**`, and its Intent names `RestoreRenumber` on the write-path line; its ImpactSummary says so.
- **F5** (card 4 misstates restore's bound): card 4's bound paragraph now covers `insert-card` and `remove-card` alone, and a second paragraph bounds `restore-renumber`: same lease, only top-level journal names, refusal of an absent, undecodable or outside-path journal, and what it lets through, the discarding of edits since the interrupted call.
  Card 4's `doc.go` section states that cost.
- **F6** (archive-untouched criterion untested): card 2's fixtures hold `archive-1/` and `round-1/prior-generation/` with card-named files, and the byte-for-byte comparison covers them; card 3's restore rows use the same fixtures.
- **F7** (card 4 duplicates planparser tests; per-verb success tests): card 2 folds `TestInsertCard` and `TestRemoveCard` into one `TestRenumber` table, and its budget raise drops from four to three.
  Card 4 has two tests, `TestRenumberVerbs` (a success table over the three verbs' envelopes and the lease) and `TestRenumberVerbs_Refusals` (exit code and way forward per row; only the begun-bound rows assert the directory untouched), and its budget raise drops from four to two.
- **F8** (non-integer `N` read as a run-id): verified against `internal/loomcli/arm.go:89`, where `resolveRunID` takes `args[0]` as the run-id for every verb but `reject`.
  Card 4 edits `loomCLI.resolveRunID` to exempt `insert-card` and `remove-card` beside `reject`, with `TestResolveRunID` rows in `arm_seed_test.go`.
  `insertCardVerb` and `removeCardVerb` now take the raw position string and parse it, so `TestRenumberVerbs_Refusals` reaches the non-integer position refusal.
- **F9** (racy lease test): the goroutine probe is gone; `acquireLease` wraps the real `AcquireStateMutation` to record acquisition and release, and the injected `ops` fail the test at any apply step reached without the lease held.
- **F10** (precheck keyed on a readable journal): card 2's precheck refuses with `ErrRenumberJournalExists` on a successful read, and treats any other read error than `fs.ErrNotExist` as `ErrRenumberIOFailed` with nothing written; `TestRenumber_Refusals` gains the unreadable-journal row.

## Disputed

None.

## Deferred

None.

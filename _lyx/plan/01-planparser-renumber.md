# Card 1 — planparser-renumber

**Create:**
- `plan:internal/planparser#Renumber` -> `type Renumber struct`
- `plan:internal/planparser#RenumberFS` -> `type RenumberFS interface`
- `plan:internal/planparser#RenumberResult` -> `type RenumberResult struct`
- `plan:internal/planparser#MovedCard` -> `type MovedCard struct`
- `plan:internal/planparser#ProseMention` -> `type ProseMention struct`
- `plan:internal/planparser#RenumberJournalPath` -> `func RenumberJournalPath(anchorPath string) string`
- `plan:internal/planparser#ErrRenumberJournalPresent` -> `var ErrRenumberJournalPresent error`
- `plan:internal/planparser#ErrPlanUnparsed` -> `var ErrPlanUnparsed error`
- `plan:internal/planparser#ErrPlanInconsistent` -> `var ErrPlanInconsistent error`
- `plan:internal/planparser#ErrPositionOutOfRange` -> `var ErrPositionOutOfRange error`
- `plan:internal/planparser#ErrPositionFrozen` -> `var ErrPositionFrozen error`
- `plan:internal/planparser#ErrOnlyCard` -> `var ErrOnlyCard error`
- `plan:internal/planparser#ErrSlugInvalid` -> `var ErrSlugInvalid error`
- `plan:internal/planparser#ErrIntentInvalid` -> `var ErrIntentInvalid error`
- `plan:internal/planparser#ErrRenumberHalted` -> `var ErrRenumberHalted error`
- `plan:internal/planparser#Renumber.InsertCard` -> `func (r Renumber) InsertCard(n int, slug, intent string) (RenumberResult, error)`
- `plan:internal/planparser#Renumber.RemoveCard` -> `func (r Renumber) RemoveCard(n int) (RenumberResult, error)`
- `internal/planparser/renumber_test.go`

**Edit:**
- `internal/planparser/doc.go`
- `pattern/PATTERN-sole-parsers.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#OverviewWithoutCardIndex`
- `internal/testkit/plankit#WriteTree`
- `internal/planparser#ParsePlan`
- `internal/planparser#checkIndexFileConsistency`
- `internal/planparser#checkCardNumbering`
- `internal/planparser#cardFileName`
- `internal/planparser#cardIndexBounds`
- `internal/planparser#cardIndexLineRe`
- `internal/planparser#cardHeadingRe`
- `internal/planparser#PlanDir`
- `internal/planparser#ValidateFormat`
- `internal/lyxdirs#DotLyxDirName`
- `internal/testkit/plankit#Write`

**Intent:**
Add the plan's fourth write path, `internal/planparser/renumber.go`, beside `SetApproved`, `RewriteRefs` and `AppendAmendment`: inserting or removing one card and renumbering the cards after it (decision record D1, D2, D3).
`planparser` owns it because it is the plan's sole parser and the declarer of the plan directory's paths (`PATTERN-sole-parsers`, `PATTERN-lyxdirs-single-declarer`).

`Renumber` carries `PlanDir` (the directory `PlanDir` returns), `JournalPath`, `FrozenThrough` (the highest card number of the last begun webster batch, `0` when none; the caller computes it) and `FS`, a `RenumberFS` holding `MkdirAll`, `WriteFile`, `Rename` and `Remove`, with a nil `FS` meaning the `os` functions.
`FS` is the seam the tests inject raw I/O failures through; every read stays `os.ReadFile` or `os.ReadDir`.
`RenumberJournalPath(anchorPath)` returns `<anchorPath>/.lyx/plan/renumber-journal.json`, joined from `lyxdirs.DotLyxDirName` and `PlanDirName`: the journal is never tracked, so it sits under `.lyx` at the plan directory's mirrored subpath (`PATTERN-durable-vs-ephemeral-state`), and no new file spells `.lyx`.

`InsertCard(n, slug, intent)` makes card `n` a new card: every card numbered `n` or higher moves up by one, then it writes `NN-<slug>.md` holding only `# Card n — <slug>` and a trailing newline, and adds the Card Index line `n — <slug> — <intent>` at index position `n`, copying the neighbouring index line's leading `- ` bullet marker when it has one.
`n` one past the last card appends with no move.
`RemoveCard(n)` deletes card `n`'s file and Card Index line, and every later card moves down by one.
A move from `M` to `M±1` rewrites exactly four things: the file name's number prefix, the `# Card M` heading number, the `M: ` prefix of an explicit `**Commit:**` value (backtick-wrapped or not; a value not starting `M: ` is left alone), and the number on its Card Index line.
No other byte of a card file, and no overview byte outside the Card Index section `cardIndexBounds` bounds, changes; the frontmatter, `amendments.md`, `archive-*/` and `round-<N>/prior-generation/` are never opened for writing.
A stub's missing fields are left for the plan gate to report.

`RenumberResult` carries `Moved` (one `MovedCard` per moved card: old and new number, old and new file name), `Created` (the new file, insert) or `Removed` (the deleted file, remove), and `Prose`: one `ProseMention` (file name, 1-based line, line text) per line of the post-call overview and card files matching `[Cc]ard <M>` as a whole word, where `M` is a moved card's old number.
Each card file's title heading line is skipped, since the verb rewrites it.
Mentions are reported and never rewritten (D1), and the scan runs over the post-call contents computed in memory, so it adds no I/O after the apply.

Refusals, each checked before any write, in this order, each returned as an error wrapping its sentinel with the detail the caller needs (`fmt.Errorf("%w: ...")`):

- `ErrRenumberJournalPresent`: a file exists at `JournalPath`; checked first, since a plan mid-renumber need not parse.
  Reached by a test row that plants a journal file.
- `ErrPlanUnparsed`: `ParsePlan` fails; reached by a row whose overview has no Card Index heading.
- `ErrPlanInconsistent`: `checkIndexFileConsistency` or `checkCardNumbering` reports a finding, which a parsed plan can still carry (an orphan `.md` file, a gap, a heading number that differs from the index); reached by a row with an orphan `07-stray.md` and a row with a heading number differing from its index number.
- `ErrPositionOutOfRange`: `n` outside `first_card`..last (remove) or `first_card`..last+1 (insert), the detail naming the allowed range; reached by rows below `first_card` and past the bound for each verb.
- `ErrPositionFrozen`: `n <= FrozenThrough`, the detail naming `FrozenThrough`; reached by a row with `FrozenThrough` set to the target position.
- `ErrOnlyCard`: `RemoveCard` on a one-card plan, reached after the range check passes because `n` equals the only card's number.
- `ErrSlugInvalid`: a slug not matching `^[a-z0-9]+(-[a-z0-9]+)*$`; reached by rows `a/b`, `..`, `Foo` and `a--b`.
  This rule keeps the new file name inside `PlanDir`.
- `ErrIntentInvalid`: an `--intent` that is empty, whitespace only or holds `\n` or `\r`; reached by rows for each.

Before the first apply step the call computes every new name and content in memory, reads the pre-call bytes of every file it will rewrite, rename or delete, and writes the journal: JSON holding those bytes keyed by base name, and `created`, every base name the call creates that did not exist before it (each move destination and the new card file).
The journal is written with `MkdirAll` of its directory, `WriteFile` to the fixed name `<JournalPath>.tmp`, then `Rename` onto `JournalPath`, so a journal on disk is always complete; a failure in those steps removes the temporary file, ignoring a not-exist error from that removal, and returns a raw I/O error with the plan directory untouched.
A temporary file a kill leaves behind is ignored by the presence check and overwritten by the next write.
Apply order keeps any two files from claiming one number: `RemoveCard` removes card `n`'s file, then moves later cards in ascending order; `InsertCard` moves cards from the last down to `n` in descending order, then writes the new file.
A move is `Rename` of the old name onto the new, then `WriteFile` of the rewritten bytes at the new name.
Both then write the overview with its rewritten Card Index, and `Remove` the journal as the final step.

A raw I/O failure at any apply step, the journal's removal included, is reversed from the journal: every `created` name is removed (an absent one is not an error), every journaled file is written back under its pre-call name with its pre-call bytes, then the journal is removed, and the original error is returned unwrapped by any sentinel, so the caller reports it as the grouped transient raw-I/O case.
A failure during that reversal returns an error wrapping `ErrRenumberHalted` and leaves the journal on disk, so the next `InsertCard` or `RemoveCard` returns `ErrRenumberJournalPresent`.
These bounds are what the new edge can let through: it renames, rewrites and deletes only the current generation's top-level card files and the overview's Card Index, and only on a plan that parses consistently, outside the frozen begun cards.

`renumber_test.go` (package `planparser_test`, plans written with `plankit.Write`, and the bytes `plankit` cannot render, a bare `Commit:` value, a mismatched heading, an orphan file, the archive subtrees and a planted journal, with `plankit.WriteTree`) adds two table-driven tests, `TestRenumberInsertAndRemove` and `TestRenumberRawIOFailure`:

- `TestRenumberInsertAndRemove` rows: insert at a middle position of a four-card plan, asserting each moved card's new file name, heading number, explicit `Commit:` prefix (one backtick-wrapped, one bare) and index number, the stub's exact bytes and index line, and that the old names are gone; insert at last+1 with no move; remove at a middle position; a plan with `first_card: 3` renumbered within 3..; a card whose body says `see card 3` reported in `Prose` with its file and line and left byte-identical; and an insert and a remove over a plan beside a planted `<JournalPath>.tmp`, which proceed and leave no temporary file.
  Every row's fixture carries an `archive-x/` and a `round-1/prior-generation/` subtree, and every row asserts both stay byte-identical, so each verb is checked against them.
  Each success row re-parses the result and asserts `ValidateFormat` returns no `index-file-mismatch` or `card-numbering` finding, that `OverviewWithoutCardIndex` of the overview is byte-equal before and after (frontmatter, `approved:` and every other section untouched), and, for an insert, that `ValidateFormat` reports a blocking `card-type-missing` finding on the stub, so a forgotten stub cannot pass.
  Every refusal row above asserts `errors.Is` on its sentinel and that a recursive snapshot of the plan directory is byte-identical.
- `TestRenumberRawIOFailure` uses a test-local `RenumberFS` fake that delegates to `os` and fails the first call matching an operation and a base name, optionally also a second call to fail the reversal.
  Rows, for each verb where the step exists: the journal's temporary write, the journal's rename, the deletion, a move's rename, a move's write, the new file's write, the overview write and the journal's removal; each asserts the plan directory is byte-identical, no new-number file sits beside its old one, and neither the journal nor its temporary file exists.
  Reversal rows fail a move and then a reversal write, asserting `ErrRenumberHalted`, the journal kept, and that a following `InsertCard` and `RemoveCard` each return `ErrRenumberJournalPresent`.

Two new top-level tests take `internal/planparser` past its row, so `test-budget.yaml` raises it by two.
`doc.go` gains a `# Card renumbering` section naming `InsertCard` and `RemoveCard` as the write path for inserting or removing a card, and the journal they write.
Its package comment's sentence naming "the package's three write paths" and "three named writers" is rewritten to add `Renumber.InsertCard` and `Renumber.RemoveCard` (card renumbering, renumber.go) as the fourth.
`PATTERN-sole-parsers.md`'s write-path bullet, which closes the list with "and no others", likewise adds `Renumber.InsertCard` and `Renumber.RemoveCard` (card insertion and removal), in this commit as `CLAUDE.md` requires for an invariant's text.

**ImpactSummary:** additive to `internal/planparser`: new file and API with no existing caller changed; `doc.go`'s and `PATTERN-sole-parsers.md`'s write-path lists and the budget row are the only edits.

**Commit:** 1: planparser: insert and remove a card with a renumber journal

# Card 1 — planparser-insert-remove-card

**Create:**
- `plan:internal/planparser#PlanScratchDir` -> `func PlanScratchDir(anchorPath string) string`
- `plan:internal/planparser#RenumberJournal` -> `func RenumberJournal(anchorPath string) string`
- `plan:internal/planparser#RenumberOps` -> `type RenumberOps struct`
- `plan:internal/planparser#CardMove` -> `type CardMove struct`
- `plan:internal/planparser#ProseMention` -> `type ProseMention struct`
- `plan:internal/planparser#RenumberResult` -> `type RenumberResult struct`
- `plan:internal/planparser#ErrRenumberJournalPresent` -> `var ErrRenumberJournalPresent error`
- `plan:internal/planparser#ErrPlanUnparseable` -> `var ErrPlanUnparseable error`
- `plan:internal/planparser#ErrPlanInconsistent` -> `var ErrPlanInconsistent error`
- `plan:internal/planparser#ErrCardPositionOutOfRange` -> `var ErrCardPositionOutOfRange error`
- `plan:internal/planparser#ErrOnlyCard` -> `var ErrOnlyCard error`
- `plan:internal/planparser#ErrInvalidCardSlug` -> `var ErrInvalidCardSlug error`
- `plan:internal/planparser#ErrInvalidCardIntent` -> `var ErrInvalidCardIntent error`
- `plan:internal/planparser#ErrRenumberReversalFailed` -> `var ErrRenumberReversalFailed error`
- `plan:internal/planparser#InsertCard` -> `func InsertCard(planDir, journalPath string, n int, slug, intent string, ops RenumberOps) (RenumberResult, error)`
- `plan:internal/planparser#RemoveCard` -> `func RemoveCard(planDir, journalPath string, n int, ops RenumberOps) (RenumberResult, error)`
- `plan:internal/planparser_test#TestInsertCard` -> `func TestInsertCard(t *testing.T)`
- `plan:internal/planparser_test#TestRemoveCard` -> `func TestRemoveCard(t *testing.T)`
- `plan:internal/planparser_test#TestRenumber_ApplyFailureLeavesPlanUnchanged` -> `func TestRenumber_ApplyFailureLeavesPlanUnchanged(t *testing.T)`

**Edit:**
- `internal/planparser/doc.go`
- `pattern/PATTERN-sole-parsers.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#ParsePlan`
- `internal/planparser#checkIndexFileConsistency`
- `internal/planparser#checkCardNumbering`
- `internal/planparser#cardFileName`
- `internal/planparser#cardHeadingRe`
- `internal/planparser#cardIndexBounds`
- `internal/planparser#cardIndexLineRe`
- `internal/planparser#splitFrontmatter`
- `internal/planparser#OverviewWithoutCardIndex`

**Intent:**
Add the plan's fourth write path, card insertion and removal with renumbering (D1, D2, D3), in a new file `internal/planparser/renumber.go`, beside `SetApproved`.

Paths: `PlanScratchDir(anchorPath)` is `filepath.Join(anchorPath, lyxdirs.DotLyxDirName, PlanDirName)`, the `.lyx` sibling of `PlanDir` (`PATTERN-durable-vs-ephemeral-state`: each module exposes a scratch accessor beside its durable one), and `RenumberJournal(anchorPath)` is `renumber-journal.json` inside it.
No new file spells `_lyx` or `.lyx`; both come from `lyxdirs` (`PATTERN-lyxdirs-single-declarer`).

`RenumberOps` is the file-operation seam the apply and reversal steps go through: `WriteFile`, `Rename` and `Remove` function fields, each nil field defaulting to its `os` function, so production passes the zero value and a test injects a failure at one named step.
`CardMove` is one moved card: old and new number, old and new file name.
`ProseMention` is one line naming a moved card's old number in prose: file name, 1-based line number and the line's text.
`RenumberResult` carries `Moved []CardMove`, `Created` (insert's new file name, else empty), `Removed` (remove's deleted file name, else empty) and `ProseMentions []ProseMention`.

`InsertCard(planDir, journalPath, n, slug, intent, ops)` and `RemoveCard(planDir, journalPath, n, ops)` refuse, before writing anything, in this order, each a wrapped sentinel:

1. `ErrRenumberJournalPresent`: a file exists at `journalPath` (input: a journal left on disk over a valid plan).
2. `ErrInvalidCardSlug` (insert only): `slug` does not match `^[a-z0-9]+(-[a-z0-9]+)*$` (inputs: `Bad_Slug`, `a/b`, `..`, `a--b`, empty).
3. `ErrInvalidCardIntent` (insert only): `intent` is empty or whitespace only, or holds `\n` or `\r` (inputs: ``, `   `, `one\ntwo`, `one\rtwo`).
4. `ErrPlanUnparseable`: `ParsePlan(planDir)` fails, its error wrapped (input: an overview with no `## Card Index` heading).
5. `ErrPlanInconsistent`: `checkIndexFileConsistency` or `checkCardNumbering` reports a finding, the findings joined into the message (inputs: an orphan `09-stray.md` beside a valid plan; a card whose heading number differs from its index number).
6. `ErrCardPositionOutOfRange`: `n` is below `first_card`, or above the last card number for remove, or above the last card number plus one for insert (inputs: `n = first_card - 1`, `n = last + 1` for remove, `n = last + 2` for insert).
7. `ErrOnlyCard` (remove only): the plan has exactly one card and `n` names it.

Every input above passes every earlier check, so each refusal is reachable, and each is a row of `TestInsertCard` or `TestRemoveCard` asserting `errors.Is` on its sentinel and a plan directory byte-identical to before the call (read with `os.ReadDir` and `os.ReadFile` over the whole tree, subdirectories included).

Rewrite, computed in memory first from the pre-call bytes: moving card `M` to `M±1` rewrites exactly four things, its file name (`cardFileName`), the number in its `# Card M` heading line (the first line, located with `cardHeadingRe`), the `M: ` prefix of an explicit `**Commit:**` value (the backticks kept), and the number of its Card Index line; no other byte of the card changes.
Insert writes the new card's file `cardFileName(n, slug)` holding only `# Card n — <slug>` and a trailing newline, and inserts the index line `n — <slug> — <intent>` before the index line of the card previously numbered `n`, or after the last index line when `n` is one past the last card.
Remove deletes card `n`'s file and its index line.
The overview's frontmatter, framing and every section other than the Card Index stay byte-identical; `amendments.md`, `archive-*/`, `round-<N>/` and every other subdirectory are never read for writing, renamed or removed, since only top-level card files the Card Index names and `00-overview.md` are touched.

Journal: before the first apply step the call writes a JSON journal holding the pre-call bytes of every file it will rewrite, rename or delete, keyed by base name (the overview and every moved or removed card), and every base name it will create that did not exist before (each move destination and insert's new file).
Its JSON shape is fixed, so a test can write one by hand: an object with key `files`, an object mapping each base name to its pre-call bytes (a `[]byte`, so base64 in JSON), and key `created`, an array of base names.
It is written to `journalPath + ".tmp"` (parent directory created) and renamed to `journalPath` through `ops`, so a journal on disk is always complete; a failure writing or renaming the temporary file removes it and returns the raw error with the plan directory untouched.
A stale `.tmp` left by a kill is ignored by the presence check and overwritten by the next write.

Apply order, so no two files ever claim one number: remove deletes card `n`'s file, then moves the later cards in ascending order; insert moves the cards from `n` up in descending order, then writes the new file.
A move writes the rewritten bytes under the new name, then removes the old name.
Both then rewrite `00-overview.md` and remove the journal as the final step.

A raw failure at any apply step, the journal's removal included, is reversed from the in-memory journal: every journaled created name is removed (a missing one is not an error), every journaled file is written back under its pre-call name with its pre-call bytes, then the journal is removed, and the original error is returned unwrapped by any sentinel, so the caller reports it as the grouped transient raw-I/O row.
A failure during that reversal returns an error wrapping both `ErrRenumberReversalFailed` and the original error, and the journal is left on disk.
The reversal is an unexported helper reading the journal model, which card 2's `RestoreRenumber` reuses.

Prose mentions: after the rewrite the call scans the post-call text of `00-overview.md`, skipping the Card Index lines, and of every card file, skipping its heading line, for `\b[Cc]ard (\d+)\b` whose number is a moved card's old number, and reports each matching line by post-call file name and line; those lines are never rewritten (D1).

Tests, in a new external test file `internal/planparser/renumber_test.go`, plans written with `plankit`:
- `TestInsertCard` rows: middle insert into a three-card plan whose overview carries framing and a `## verify:` section, with an explicit `Commit:` on a moved card (moved files, headings, `Commit:` prefixes and index numbers asserted exactly, new heading-only file and index line asserted, `OverviewWithoutCardIndex` of the overview equal before and after the call, the result parsing with zero `index-file-mismatch` and `card-numbering` findings from `Validate`); append at last plus one (no moves); a three-card plan with `first_card: 5` and an insert at 6, renumbered to 5..8; a prose line `see card 2` in a moved card reported and left unchanged; a file under `archive-1/` and under `round-1/prior-generation/` byte-identical after the call; a stale `journalPath + ".tmp"` left beside a valid plan, the insert succeeding and neither the journal nor the `.tmp` left behind; and the insert refusal rows above.
- `TestRemoveCard` rows: middle remove with the same consistency and `OverviewWithoutCardIndex` assertions, `first_card` above 1, the `Removed` field, a file under `archive-1/` and under `round-1/prior-generation/` byte-identical after the call, and the remove refusal rows above.
- `TestRenumber_ApplyFailureLeavesPlanUnchanged` rows inject, through `RenumberOps`, a failure at the journal's temporary write, the journal rename, remove's deletion, insert's new-file write, a move's write, a move's old-name removal, the Card Index rewrite and the journal's removal, for each verb where the step exists; each asserts a non-sentinel error, a plan directory byte-identical to before the call with no new-number file beside an old one, and neither `journalPath` nor its `.tmp` on disk.
  The injected fault is one-shot: the test's closure fails only the first call matching the step's path and passes every later call through to `os`, so the reversal's own write-back of `00-overview.md` and removal of the journal succeed.
  A further row fails a move's write and then every later `WriteFile`, so the reversal's write-back fails, and asserts `ErrRenumberReversalFailed` and the journal still on disk.

`internal/planparser/doc.go` and `pattern/PATTERN-sole-parsers.md` name the new write path beside `SetApproved`, `RewriteRefs` and `AppendAmendment`, four named writers instead of three.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/planparser` by the three new top-level tests.

**ImpactSummary:** additive: new exported functions and types in planparser with no existing caller changed; two doc sentences and one budget row move with them.

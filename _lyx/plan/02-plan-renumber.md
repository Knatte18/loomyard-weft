# Card 2 — plan-renumber

**Create:**
- `plan:internal/planparser#RenumberFS` -> `type RenumberFS interface`
- `plan:internal/planparser#OSRenumberFS` -> `func OSRenumberFS() RenumberFS`
- `plan:internal/planparser#RenumberJournal` -> `func RenumberJournal(anchorPath string) string`
- `plan:internal/planparser#MovedCard` -> `type MovedCard struct`
- `plan:internal/planparser#ProseMention` -> `type ProseMention struct`
- `plan:internal/planparser#RenumberResult` -> `type RenumberResult struct`
- `plan:internal/planparser#InsertCard` -> `func InsertCard(fsys RenumberFS, planDir, journalPath string, n int, slug, intent string, frozenThrough int) (RenumberResult, error)`
- `plan:internal/planparser#RemoveCard` -> `func RemoveCard(fsys RenumberFS, planDir, journalPath string, n, frozenThrough int) (RenumberResult, error)`
- `plan:internal/planparser#RestoreRenumber` -> `func RestoreRenumber(fsys RenumberFS, planDir, journalPath string) ([]string, error)`
- `plan:internal/planparser#ErrRenumberJournalPresent` -> `var ErrRenumberJournalPresent error`
- `plan:internal/planparser#ErrRenumberPlanInconsistent` -> `var ErrRenumberPlanInconsistent error`
- `plan:internal/planparser#ErrRenumberPosition` -> `var ErrRenumberPosition error`
- `plan:internal/planparser#ErrRenumberFrozen` -> `var ErrRenumberFrozen error`
- `plan:internal/planparser#ErrRenumberOnlyCard` -> `var ErrRenumberOnlyCard error`
- `plan:internal/planparser#ErrRenumberSlug` -> `var ErrRenumberSlug error`
- `plan:internal/planparser#ErrRenumberIntent` -> `var ErrRenumberIntent error`
- `plan:internal/planparser#ErrRenumberIO` -> `var ErrRenumberIO error`
- `plan:internal/planparser#ErrRenumberReversalFailed` -> `var ErrRenumberReversalFailed error`
- `plan:internal/planparser#ErrRenumberNoJournal` -> `var ErrRenumberNoJournal error`
- `plan:internal/planparser#ErrRenumberJournalInvalid` -> `var ErrRenumberJournalInvalid error`
- `plan:internal/planparser#ErrRestorePartial` -> `var ErrRestorePartial error`
- `plan:internal/testkit/plankit#FaultFS` -> `type FaultFS struct`
- `internal/planparser/renumber_test.go`
- `plan:internal/planparser_test#TestRenumber` -> `func TestRenumber(t *testing.T)`
- `plan:internal/planparser_test#TestRenumber_FaultsLeaveThePlanUnchanged` -> `func TestRenumber_FaultsLeaveThePlanUnchanged(t *testing.T)`
- `plan:internal/planparser_test#TestRestoreRenumber` -> `func TestRestoreRenumber(t *testing.T)`

**Edit:**
- `internal/planparser/doc.go`
- `pattern/PATTERN-sole-parsers.md`
- `cmd/lyx#pathRules`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#ParsePlan`
- `internal/planparser#cardFileName`
- `internal/planparser#cardHeadingRe`
- `internal/planparser#cardIndexBounds`
- `internal/planparser#splitFrontmatter`
- `internal/planparser#cardIndexLineRe`
- `internal/planparser#commitLabel`
- `internal/planparser#overviewFileName`
- `internal/planparser#PlanDirName`

**Intent:**
Add the plan-format renumber writer to `internal/planparser`, in a new `renumber.go` beside `SetApproved`, since `planparser` is the plan's sole parser and writer (`PATTERN-sole-parsers`, D3).
The tests go in a new `renumber_test.go` in the external `planparser_test` package.

`RenumberFS` is the file-mutation seam every apply, reversal and restore step goes through, with three methods: `WriteFile(name string, data []byte, perm os.FileMode) error`, `Rename(oldpath, newpath string) error` and `Remove(name string) error`.
`OSRenumberFS` returns the `os`-backed implementation production passes; a test passes `plankit.FaultFS`.
`FaultFS`, in `internal/testkit/plankit/plankit.go`, is the one fault-injecting `RenumberFS` both this card's tests and card 3's use (`PATTERN-testkit`): it delegates to `OSRenumberFS()` and fails, once each, the calls its failure list names, each named by operation (`WriteFile`, `Rename` or `Remove`) and its 1-based ordinal among that operation's calls, so a fault at the journal's removal does not also fail the reversal's later `Remove` of the journal, and one row can fail a move and then the reversal's first `WriteFile`.
An optional `OnWrite func()` hook runs before each `WriteFile`, for card 3's lease probe.
Reads (`os.ReadFile`, `os.Stat`, `os.ReadDir`) and the journal directory's `os.MkdirAll` stay direct.

`RenumberJournal(anchorPath)` returns `<anchorPath>/.lyx/plan/renumber-journal.json`, built from `lyxdirs.DotLyxDirName` and `PlanDirName`, the never-tracked mirror of `PlanDir` (`PATTERN-durable-vs-ephemeral-state`, `PATTERN-lyxdirs-single-declarer`); no new file spells `_lyx` or `.lyx`.
Its temporary sibling is the fixed name `renumber-journal.json.tmp`.
Add a `classTransient` row for `planparser.RenumberJournal` to `pathRules` in `cmd/lyx/pathrules_test.go`, so the placement guard checks it.

`InsertCard` makes card `n` a new card with `slug`, and `RemoveCard` removes card `n` (D1).
Each checks, in this order, writing nothing on any refusal:
1. A journal exists at `journalPath`: `ErrRenumberJournalPresent`. A temporary journal alone is ignored.
2. `ParsePlan(planDir)` fails, or the parsed plan has an `index-file-mismatch` or `card-numbering` finding (`checkIndexFileConsistency`, `checkCardNumbering`): `ErrRenumberPlanInconsistent`, wrapping the parse error or naming the findings.
3. `n` outside `FirstCard` to the last card number (remove) or `FirstCard` to the last plus one (insert): `ErrRenumberPosition`, naming the allowed range.
4. `n <= frozenThrough`, where the caller passes the last card of the last begun batch and 0 means no batch is begun: `ErrRenumberFrozen`, naming `frozenThrough`.
5. `RemoveCard` over a plan with one card: `ErrRenumberOnlyCard`.
6. `InsertCard` with a slug not matching `^[a-z0-9]+(-[a-z0-9]+)*$`, which rules out `/`, `..` and the empty slug, so the new file name stays a top-level file of `planDir`: `ErrRenumberSlug`.
7. `InsertCard` with an `intent` that is empty after trimming, or holds `\n` or `\r`: `ErrRenumberIntent`.
Each refusal's reaching input is the one named in its step, given a plan that passes every earlier step, and `TestRenumber` reaches each.

It then computes every new file name and content in memory.
A card moving from `M` to `M±1` changes exactly four things: its file name prefix (via `cardFileName`), the number in its `# Card M` heading line (the first line, matched by `cardHeadingRe`), the `M: ` prefix of a `**Commit:**` value that starts with it, backticks kept, and the leading number of its Card Index line; every other byte is kept.
`InsertCard` writes `NN-<slug>.md` holding only `# Card N — <slug>` and a trailing newline, and inserts the Card Index line `N — <slug> — <intent>` before the line of the old card `n`, or after the last index line on an append.
`RemoveCard` drops card `n`'s Card Index line.
Only the Card Index lines change in `00-overview.md` (located with `splitFrontmatter` and `cardIndexBounds`); frontmatter and every other section stay byte-identical, so `OverviewWithoutCardIndex` is unchanged and rebaseline needs no webster change.
Nothing under `archive-*/`, `round-<N>/prior-generation/` or any other subdirectory, nor `amendments.md`, is read for rewriting or touched.

Before the first apply step it writes the journal: JSON `{"files": {"<base name>": <pre-call bytes>}, "created": ["<base name>", ...]}`, with bytes in Go's default base64 `[]byte` encoding.
`files` holds the pre-call bytes of every file the call rewrites, renames or deletes, `00-overview.md` included; `created` holds every name that did not exist before the call: each move's destination and the new card's file.
It `os.MkdirAll`s the journal directory, `WriteFile`s the temporary name and `Rename`s it onto `journalPath`; a failure of any of these removes the temporary file and returns `ErrRenumberIO` with the plan directory untouched.

Apply order: `RemoveCard` `Remove`s card `n`'s file, then moves the later cards in ascending order; `InsertCard` moves the cards from `n` up in descending order, then `WriteFile`s the new card's file.
A move is `Rename(old, new)` followed by `WriteFile(new, rewritten bytes)`.
Both then `WriteFile` the overview's new Card Index and `Remove` the journal as the last step.
A failure at any apply step, the journal's removal included, is reversed from the in-memory journal: `Remove` every created name (an absent one is fine), `WriteFile` every journaled file back under its pre-call name with its pre-call bytes, then `Remove` the journal; the call returns `ErrRenumberIO` wrapping the cause, and the directory is byte-identical to before the call.
A failure inside that reversal stops it, leaves the journal on disk and returns `ErrRenumberReversalFailed` wrapping both causes.

`RenumberResult` carries `Moved []MovedCard` (each `From`, `To`, `FromFile`, `ToFile`, in apply order), `Created` (the new card's file name, insert only), `Removed` (the removed card's file name, remove only) and `Prose []ProseMention`.
`Prose` lists every line of the post-call top-level card files and overview, other than a card file's heading line, that matches `[Cc]ard <M>` as a whole word for an `M` some moved card had before the call, each as `File` (base name), `Line` (1-based) and `Text`; those lines are left unchanged (D1).

`RestoreRenumber` is the journal's only reader.
No journal at `journalPath`: `ErrRenumberNoJournal`.
A journal that does not decode: `ErrRenumberJournalInvalid`, writing nothing.
A journal naming, in `files` or `created`, any name that is not `00-overview.md` or a top-level card file name: `ErrRenumberJournalInvalid`, writing nothing, so a corrupt journal never reaches `archive-*/`, `round-<N>/prior-generation/` or outside `planDir`.
A card file name is any name `cardFileName` can produce from a Card Index line: a single base name (equal to its own `filepath.Base`, holding no `/` or `\`, not `.` or `..`) matching `^\d{2,}-\S+\.md$`.
It is not the insert slug rule, since `cardIndexLineRe` admits any non-whitespace slug, and a moved card with such a slug must still be restorable.
Otherwise it `Remove`s every created name (an absent one is fine), `WriteFile`s every journaled file's bytes back, then `Remove`s the journal, returning the names it wrote.
A failure partway returns `ErrRestorePartial` wrapping the cause and keeps the journal; a re-run is idempotent.

Tests, all tier 1 over `t.TempDir()` plans written with `plankit.Write`.
Every fixture also plants files under `archive-1/` and `round-1/prior-generation/`, and every row snapshots every file under `planDir`, subdirectories included, before the call, so each row of all three verbs shows those subdirectories untouched:
- `TestRenumber`, one table over both verbs: insert at a middle position (later cards moved in name, heading, an explicit `Commit:` prefix and index number; heading-only new file; new index line; `ParsePlan` plus the two checks report nothing); insert one past the last card (no moves); remove at a middle position; a plan with `first_card: 5` renumbered from 5; a prose line `see card 3` reported with its file and line and left unchanged; an insert over a planted `renumber-journal.json.tmp` succeeds and leaves no temporary journal.
  Every success row asserts `OverviewWithoutCardIndex` of the overview equal before and after the call, so only the Card Index changed.
  One row per refusal of steps 1 to 7, each asserting the sentinel with `errors.Is` and the snapshot unchanged: a journal present; an unparseable overview, an orphan card file (`index-file-mismatch`) and a card file whose heading number differs from its Card Index number (`card-numbering`); `n` below `first_card` and above the range; `n` equal to `frozenThrough`; the only card; slugs `a/b`, `..` and `Bad`; and intents `""`, `"  "` and `"a\nb"`.
- `TestRenumber_FaultsLeaveThePlanUnchanged`, one table whose rows fail one `FaultFS` call: the journal's temporary write, its rename, the deletion, the new-file write, a move's rename, a move's write, the Card Index rewrite and the journal's removal, over the verb that reaches each; every row asserts `ErrRenumberIO`, the snapshot unchanged, no moved card's new-number file beside its old one, and neither journal nor temporary journal on disk.
  A last row fails a `RemoveCard` move and the reversal's first `WriteFile`: it asserts `ErrRenumberReversalFailed` and the journal present, then `InsertCard` and `RemoveCard` each return `ErrRenumberJournalPresent`; the restore of that journal is `TestRestoreRenumber`'s.
- `TestRestoreRenumber`: no journal, a planted temporary journal alone (`ErrRenumberNoJournal`), a journal of non-JSON bytes, and journals naming `../x.md`, `archive-1/01-a.md` and `notes.txt` each return their sentinel with the snapshot unchanged.
  A journal left by a `RemoveCard` reversal failure over a plan whose moved cards carry explicit `Commit:` lines, one moved card with the slug `Fix_x`, is restored through a `FaultFS` failing its second `WriteFile`: it returns `ErrRestorePartial` with the journal kept, and the re-run with `OSRenumberFS()` leaves the snapshot of before the interrupted call, the deleted card's bytes and the moved cards' headings and `Commit:` prefixes included, and removes the journal.

Update `internal/planparser/doc.go`'s package comment, which names three write paths, to name `InsertCard`, `RemoveCard` and `RestoreRenumber` as the renumber writer and the journal path under the path-ownership section.
Update `pattern/PATTERN-sole-parsers.md`, whose line names `SetApproved`, `RewriteRefs` and `AppendAmendment` as "the three write paths, and no others", to add the renumber writer.
Raise `internal/planparser`'s row in `cmd/lyx/testdata/test-budget.yaml` from 86 to 89 for the three new tests.

**ImpactSummary:** Additive: a new planparser file with no caller until card 3, one new test-only type in plankit, one new transient row in the cmd/lyx path guard, two doc sentences and a budget row; no existing member changes behavior.

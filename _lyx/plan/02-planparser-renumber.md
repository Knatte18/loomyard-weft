# Card 2 — planparser-renumber

**Create:**
- `internal/planparser/renumber.go`
- `internal/planparser/renumber_test.go`
- `plan:internal/planparser#PlanScratchDir` -> `func PlanScratchDir(anchorPath string) string`
- `plan:internal/planparser#RenumberJournalPath` -> `func RenumberJournalPath(anchorPath string) string`
- `plan:internal/planparser#RenumberIO` -> `type RenumberIO struct`
- `plan:internal/planparser#OSRenumberIO` -> `func OSRenumberIO() RenumberIO`
- `plan:internal/planparser#MovedCard` -> `type MovedCard struct`
- `plan:internal/planparser#ProseMention` -> `type ProseMention struct`
- `plan:internal/planparser#RenumberResult` -> `type RenumberResult struct`
- `plan:internal/planparser#InsertCard` -> `func InsertCard(ops RenumberIO, planDir, journalPath string, number int, slug, intent string) (*RenumberResult, error)`
- `plan:internal/planparser#RemoveCard` -> `func RemoveCard(ops RenumberIO, planDir, journalPath string, number int) (*RenumberResult, error)`
- `plan:internal/planparser#ErrRenumberInput` -> `var ErrRenumberInput error`
- `plan:internal/planparser#ErrRenumberPlanInconsistent` -> `var ErrRenumberPlanInconsistent error`
- `plan:internal/planparser#ErrRenumberJournalExists` -> `var ErrRenumberJournalExists error`
- `plan:internal/planparser#ErrRenumberIOFailed` -> `var ErrRenumberIOFailed error`
- `plan:internal/planparser#ErrRenumberReversalFailed` -> `var ErrRenumberReversalFailed error`
- `plan:internal/planparser#renumberJournal` -> `type renumberJournal struct`
- `plan:internal/planparser#renumberPrecheck` -> `func renumberPrecheck(ops RenumberIO, planDir, journalPath string) (*Plan, error)`
- `plan:internal/planparser#moveCardBytes` -> `func moveCardBytes(data []byte, from, to int) []byte`
- `plan:internal/planparser#writeRenumberJournal` -> `func writeRenumberJournal(ops RenumberIO, journalPath string, j renumberJournal) error`
- `plan:internal/planparser#reverseRenumber` -> `func reverseRenumber(ops RenumberIO, planDir string, j renumberJournal) error`
- `plan:internal/planparser#proseMentions` -> `func proseMentions(ops RenumberIO, planDir string, plan *Plan, oldNumbers []int) ([]ProseMention, error)`
- `plan:internal/planparser#cardSlugRe` -> `var cardSlugRe *regexp.Regexp`

**Edit:**
- `internal/planparser/doc.go`
- `pattern/PATTERN-sole-parsers.md`
- `cmd/lyx/pathrules_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#SetApproved`
- `internal/planparser#ParsePlan`
- `internal/planparser#PlanDir`
- `internal/planparser#cardFileName`
- `internal/planparser#cardHeadingRe`
- `internal/planparser#cardIndexLineRe`
- `internal/planparser#checkCardNumbering`
- `internal/lyxdirs#DotLyxDirName`

**Intent:** Add the plan-format renumber, the write path the card verbs of card 4 call, beside `SetApproved`, since `planparser` is the plan's sole parser and writer and owns the plan directory's paths.

Paths: `PlanScratchDir` is the plan directory's never-tracked mirror, `<anchor>/.lyx/plan`, joined from `lyxdirs.DotLyxDirName` and `PlanDirName` exactly as `PlanDir` joins its durable form, and `RenumberJournalPath` is `renumber-journal.json` under it; the temporary journal is that path with `.tmp` appended, declared once here.
`internal/lyxdirs` declares only the two directory tokens, so, as every module's scratch accessor does, this one builds the path from them and spells neither literal.
Add both to `cmd/lyx/pathrules_test.go`'s rows as transient, `PlanScratchDir` mirroring `planparser.PlanDir`.

`RenumberIO` is the raw file seam: `ReadFile`, `WriteFile`, `Rename` and `Remove` function fields with the signatures of their `os` namesakes, and `OSRenumberIO` fills them from `os`.
It exists so a test fails one named operation on one named file; production always passes `OSRenumberIO()`.

`MovedCard` holds `OldNumber`, `NewNumber`, `OldFile` and `NewFile`; `ProseMention` holds `File`, `Line` and `Text`; `RenumberResult` holds `Moved`, `Created`, `Removed` and `ProseMentions`.

Both verbs run `renumberPrecheck` before touching anything: a readable journal at `journalPath` is `ErrRenumberJournalExists` (a `.tmp` file alone is ignored); the plan is parsed through `ParsePlanFrom` over `ops.ReadFile`, and a parse error is returned as is; the parsed plan is held to `checkIndexFileConsistency` and `checkCardNumbering`, and any finding is `ErrRenumberPlanInconsistent`, carrying the findings' text.
Then the input checks, each wrapping `ErrRenumberInput` with its own message that names the fix: `number` below `FirstCard` or above the last card (`RemoveCard`) or the last card plus one (`InsertCard`); `RemoveCard` on a plan with one card; a `slug` that does not match `cardSlugRe`, `^[a-z0-9]+(-[a-z0-9]+)*$`, which is what keeps `/` and `..` out of the file name; an `intent` that is empty or whitespace after trimming or holds a line break.
`planparser` names no `lyx` verb in any message; card 4 words each way forward.

Every new file name and content is computed in memory first.
`moveCardBytes` rewrites exactly two things in a card moving from `from` to `to`: the number in its `cardHeadingRe` heading, and the `from: ` prefix of an explicit `**Commit:**` value, inside its backticks when it has them; every other byte is kept.
The Card Index rewrite keeps every line of the section verbatim, blank lines and bullet markers included, except that each `cardIndexLineRe` line of a moved card has its number replaced, `InsertCard` adds the line `N — <slug> — <intent>` at its position, and `RemoveCard` drops the removed card's line; nothing outside the Card Index section of `00-overview.md` is read or written, so the overview's frame hash is unchanged.
The new card's file holds only `# Card N — <slug>` and a newline.

The journal is `renumberJournal`: `Files`, the pre-call bytes of every file the call rewrites, renames or deletes, keyed by base name, and `Created`, every base name the call creates that did not exist before it, each move's destination and the new card's file included.
`writeRenumberJournal` encodes it as JSON, writes it to the temporary name through `ops.WriteFile`, then `ops.Rename`s it into place; a failure of either removes the temporary file and is `ErrRenumberIOFailed` with the plan directory untouched.

Apply order, so no two files ever claim one number: `RemoveCard` removes card `N`'s file, then moves each later card in ascending order; `InsertCard` moves the cards from `N` up in descending order, then writes the new card's file; both rewrite `00-overview.md` last and remove the journal as the final step.
A move writes the card's bytes under its new name, then removes its old name.
A failure of any apply step, the journal's removal included, calls `reverseRenumber`: it removes every `Created` name, then writes every `Files` entry back under its name, then removes the journal, leaving the directory byte-identical; the call returns `ErrRenumberIOFailed` wrapping the cause.
A failure inside the reversal returns `ErrRenumberReversalFailed` and leaves the journal on disk for card 3's restore.

After the apply, `proseMentions` scans every file of the current plan, `00-overview.md` and the card files, for `\b[Cc]ard\s+(\d+)\b` whose number is an old number of a moved card, skipping each card's heading line and the Card Index lines, and reports file, line and trimmed text; the lines are never rewritten.

Tests, in `renumber_test.go`, all tier 1 over `plankit.Write` fixtures in a temp dir, each comparing the directory's files byte for byte: `TestInsertCard` (a middle position moves every later card's file name, heading, explicit `Commit:` prefix and index number, writes the stub and index line, reports the prose mention of an old number and leaves it unchanged, and `ValidateFormat` reports no `index-file-mismatch` or `card-numbering` finding; one past the last card appends with no moves; a `first_card` above 1 renumbers within its range); `TestRemoveCard` (middle and last positions, and a `first_card` above 1); `TestRenumber_Refusals`, a table over both verbs reaching every precheck and input refusal above, the journal-present case included, each leaving the directory byte-identical with no journal or temporary file written; `TestRenumber_IOFailureReversed`, a table over both verbs and each failing step (the temporary journal's write, its rename, the deletion, the new file's write, a move's write and its remove, the Card Index rewrite, the journal's removal), each `ErrRenumberIOFailed` with the directory byte-identical, no moved card's new-number file beside its old one, and no journal or temporary file left, plus a failure injected into the reversal reaching `ErrRenumberReversalFailed` with the journal kept.
`internal/planparser` is at its test budget, so raise its row by four.

Docs: `doc.go` names `InsertCard` and `RemoveCard` as write paths beside the three it lists, and the scratch mirror and journal under "Path ownership"; `pattern/PATTERN-sole-parsers.md`'s write-path line names them too.

**ImpactSummary:** New write paths and a scratch accessor in `planparser` with no existing caller changed; the path-rules guard gains two rows, and the sole-parsers background names two more writers.

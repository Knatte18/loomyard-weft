# Card 2 — planparser-renumber

**Create:**
- `internal/planparser/renumber.go`
- `internal/planparser/renumber_test.go`
- `internal/planparser#PlanScratchDir`
- `internal/planparser#RenumberJournalPath`
- `internal/planparser#RenumberIO`
- `internal/planparser#OSRenumberIO`
- `internal/planparser#MovedCard`
- `internal/planparser#ProseMention`
- `internal/planparser#RenumberResult`
- `internal/planparser#InsertCard`
- `internal/planparser#RemoveCard`
- `internal/planparser#ErrRenumberInput`
- `internal/planparser#ErrRenumberPlanInconsistent`
- `internal/planparser#ErrRenumberJournalExists`
- `internal/planparser#ErrRenumberIOFailed`
- `internal/planparser#ErrRenumberReversalFailed`
- `internal/planparser#renumberJournal`
- `internal/planparser#renumberPrecheck`
- `internal/planparser#moveCardBytes`
- `internal/planparser#writeRenumberJournal`
- `internal/planparser#reverseRenumber`
- `internal/planparser#proseMentions`
- `internal/planparser#cardSlugRe`

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

`RenumberIO` is the raw file seam: `ReadFile`, `WriteFile`, `Rename`, `Remove` and `MkdirAll` function fields with the signatures of their `os` namesakes, and `OSRenumberIO` fills them from `os`.
It exists so a test fails one named operation on one named file; production always passes `OSRenumberIO()`.

`MovedCard` holds `OldNumber`, `NewNumber`, `OldFile` and `NewFile`; `ProseMention` holds `File`, `Line` and `Text`; `RenumberResult` holds `Moved`, `Created`, `Removed` and `ProseMentions`.

Both verbs run `renumberPrecheck` before touching anything: a journal at `journalPath` that `ops.ReadFile` reads is `ErrRenumberJournalExists`, any read error other than `fs.ErrNotExist` is `ErrRenumberIOFailed` with nothing written, so a present but unreadable journal is never overwritten, and a `.tmp` file alone is ignored; the plan is parsed through `ParsePlanFrom` over `ops.ReadFile`, and a parse error is returned as is; the parsed plan is held to `checkIndexFileConsistency` and `checkCardNumbering`, and any finding is `ErrRenumberPlanInconsistent`, carrying the findings' text.
Then the input checks, each wrapping `ErrRenumberInput` with its own message that names the fix: `number` below `FirstCard` or above the last card (`RemoveCard`) or the last card plus one (`InsertCard`); `RemoveCard` on a plan with one card; a `slug` that does not match `cardSlugRe`, `^[a-z0-9]+(-[a-z0-9]+)*$`, which is what keeps `/` and `..` out of the file name; an `intent` that is empty or whitespace after trimming or holds a line break.
`planparser` names no `lyx` verb in any message; card 4 words each way forward.

Every new file name and content is computed in memory first.
`moveCardBytes` rewrites exactly two things in a card moving from `from` to `to`: the number in its `cardHeadingRe` heading, and the `from: ` prefix of an explicit `**Commit:**` value, inside its backticks when it has them; every other byte is kept.
The Card Index rewrite keeps every line of the section verbatim, blank lines and bullet markers included, except that each `cardIndexLineRe` line of a moved card has its number replaced, `InsertCard` adds the line `N — <slug> — <intent>` at its position, and `RemoveCard` drops the removed card's line; nothing outside the Card Index section of `00-overview.md` is read or written, so the overview's frame hash is unchanged.
The new card's file holds only `# Card N — <slug>` and a newline.

The journal is `renumberJournal`: `Files`, the pre-call bytes of every file the call rewrites, renames or deletes, keyed by base name, and `Created`, every base name the call creates that did not exist before it, each move's destination and the new card's file included.
`writeRenumberJournal` encodes it as JSON, creates the journal's parent directory through `ops.MkdirAll`, since no other code creates the plan's scratch mirror, writes the journal to the temporary name through `ops.WriteFile`, then `ops.Rename`s it into place; a failure of any of the three removes the temporary file if written and is `ErrRenumberIOFailed` with the plan directory untouched.

Apply order, so no two files ever claim one number: `RemoveCard` removes card `N`'s file, then moves each later card in ascending order; `InsertCard` moves the cards from `N` up in descending order, then writes the new card's file; both rewrite `00-overview.md` last and remove the journal as the final step.
A move writes the card's bytes under its new name, then removes its old name.
A failure of any apply step, the journal's removal included, calls `reverseRenumber`: it removes every `Created` name, a name that does not exist (`fs.ErrNotExist`) being skipped rather than a failure, since a failure before the last create leaves some unwritten, then writes every `Files` entry back under its name, then removes the journal, leaving the directory byte-identical; the call returns `ErrRenumberIOFailed` wrapping the cause.
A failure inside the reversal returns `ErrRenumberReversalFailed` and leaves the journal on disk for card 3's restore.

After the apply, `proseMentions` scans every file of the current plan, `00-overview.md` and the card files, for a `[Cc]ards?` reference followed by one number or a list or range of numbers (`card 8`, `Card 8`, `cards 2, 3, 4`, `Cards 6–8`, `cards 2 and 3`), reporting the line when any number in the reference, a range's interior included, is an old number of a moved card, skipping each card's heading line and the Card Index lines, and reports file, line and trimmed text; the lines are never rewritten.

Tests, in `renumber_test.go`, all tier 1 over `plankit.Write` fixtures in a temp dir whose plan directory also holds an `archive-1/` and a `round-1/prior-generation/` with card-named files, each comparing every file under the plan directory, those subtrees included, byte for byte, and each passing a `journalPath` whose parent directory does not exist yet: `TestRenumber`, a table over both verbs (insert at a middle position moves every later card's file name, heading, explicit `Commit:` prefix and index number, writes the stub and index line, reports the prose mentions of an old number, a `card N` line and an `Applies to: cards …` list line, and leaves them unchanged, and `ValidateFormat` reports no `index-file-mismatch` or `card-numbering` finding; insert one past the last card appends with no moves; remove at a middle and at the last position; each verb on a `first_card` above 1 renumbers within its range; in every row the archive and prior-generation subtrees are unchanged); `TestRenumber_Refusals`, a table over both verbs reaching every precheck and input refusal above, the journal-present and the unreadable-journal cases included, each leaving the directory byte-identical with no journal or temporary file written; `TestRenumber_IOFailureReversed`, a table over both verbs and each failing step (the journal directory's creation, the temporary journal's write, its rename, the deletion, the new file's write, a move's write and its remove, the Card Index rewrite, the journal's removal), each `ErrRenumberIOFailed` with the directory byte-identical, no moved card's new-number file beside its old one, and no journal or temporary file left, plus a failure injected into the reversal reaching `ErrRenumberReversalFailed` with the journal kept.
`internal/planparser` is at its test budget, so raise its row by three.

Docs: `doc.go` names `InsertCard` and `RemoveCard` as write paths beside the three it lists, and the scratch mirror and journal under "Path ownership"; `pattern/PATTERN-sole-parsers.md`'s write-path line names them too.

**ImpactSummary:** New write paths and a scratch accessor in `planparser` with no existing caller changed; the path-rules guard gains two rows, and the sole-parsers background names two more writers.

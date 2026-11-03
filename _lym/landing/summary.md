# Plan verbs: glyphless test files plannable, bare card N refs renumbered

Two fixes to the loom plan verbs, from issues #4 and #5. Both were seen in task `warp-visibility`.

## edit-test-file admits a test file with no symbol (#4)

Some `_test.go` files pin symbols only through blank-identifier package-level vars, such as `cmd/lym/seamsignature_test.go`.
`lym quarry toc` lists no symbols for such a file.
`edit-test-file` refused it as an Edit target and asked for member glyphs that do not exist, so no card could plan an edit to it.

`planparser.CheckEditTestFile` now takes a third argument, `glyphless func(path string) bool`, and admits any path that test reports true for.
It calls the test only for a path it would otherwise refuse.
The plan index (`internal/planglyph/index.go`) provides `glyphlessFile`, which runs quarry's toc on the file.
It reports true only when the toc lists the file with an empty, requested symbol list and no read error or lossy parse.
A file quarry cannot list, including one that does not exist yet, is still refused, so the check stays refuse-only.
Format and rework validation both pass it in.
Dispatch never runs the check, so it is unchanged.

The plan-checks spec, the plan-cards stencil block and both package docs now describe the exemption.

## insert-card and remove-card rewrite bare card references (#5)

Before this change, a renumber rewrote only `<N>-<slug>` card ids.
Bare references such as `card 4` were listed under `prose_mentions` but never rewritten, so a planner had to fix about twenty of them by hand.

`planparser.InsertCard` and `RemoveCard` now also rewrite each bare reference that `cardMentionRe` matches: `card N`, lists like `cards N, M and K`, and ranges like `cards N–M`.
Each number naming a moved card becomes that card's new number, and each rewrite is reported under `rewritten_refs` with its `from` and `to` text.
Every reference is read against the numbering before the call, the same assumption the card-id rewrite already makes.
Card ids are masked before the bare scan, so `--card 9-beta` is never read as a bare `card 9`.
Card headings and Card Index lines are skipped, as the old scan skipped them.

Some bare references cannot be rewritten without guessing:

- one that names the removed card;
- a range that the inserted or removed card falls inside, which would gain or lose a card.

These are left whole, and each line holding one is reported under `prose_mentions` with its text as the call leaves it.
A remove now also reports a reference to the removed card when no card moves, for example when the last card is removed.
The old scan missed that case.

The post-write `proseMentions` file scan is gone.
The new mentions come from the same in-memory pass that writes the files, so no file is read again after it is written.
`planAfterRenumber`, `movedOldNumbers`, `namesOldNumber` and `renumberEdit.createdSlug` lost their only use and are removed.
The `--help` text of both verbs, `internal/loomcli`'s package doc and the renumber section of `loom-plan-spec.md` now describe the new behavior.

## Design decisions

- **Rewrite rather than only report.** The record allowed either option. Reporting already existed (`prose_mentions`) and did not spare the planner the manual work, so the verbs now rewrite. The old package doc kept references as they were because "a bare number may already mean the new numbering". That case is rare, since an agent writes prose before it inserts a card, and every rewrite is listed under `rewritten_refs` for review.
- **The exemption lives in the plan index, not in quarry.** The record also allowed making `lym quarry toc` show blank-identifier vars as targetable members. Quarry is an external module (`github.com/Knatte18/quarry`), and a blank identifier has no name a glyph could resolve. So the fix accepts the file target in lym, while `planparser` itself still reads no files.

## Testing

- `TestCheckEditTestFile`: new rows for a test file the test reports glyphless, admitted, and for one where it reports a different file, still refused.
- `TestIndex_AppendsPlanChecks`: new rows in a real fixture repository. Format and rework validation admit a test file holding only `var _ = 1`, and format validation refuses one with a `TestX` member.
- `TestRenumber`: now checks the rewritten prose for each insert and remove row, and the lines left reported, including the removed-last-card case.
- `TestRenumber_RewritesCardIDs`: the insert rewrites the overview's bare `card 9` beside its card ids, and the remove reports the line naming the removed card 9.
- `TestRenumberVerbs`: the insert envelope carries the rewritten bare reference and no prose mention, and the remove envelope reports the line naming the removed card.
- `lym gate test` passed (tier0 computed and guard).

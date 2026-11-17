```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fixer carried out as written: F1 to F9, all fixed as proposed.

1. LOW. Card 6's new sentence: "the stencil's new rules sit below the `prior_rounds` marker as unconditional text, worded to apply when a prior-findings block is present".
   Summary: the trigger "when a prior-findings block is present" can be misread on a round after the first that has a nil context. On such a round `priorRoundsBlock` still renders today's text, and that text lists the earlier rounds' review and fixer-report paths. A reviewer can take that list for the block and write `prior` or `carries`, and Card 5 refuses either one when the context is nil.
   Evidence: the nil-context `priorRoundsBlock` text is "Prior-round files exist:" followed by review and fixer-report paths (`internal/burlerengine/prompt.go:366-390`). Card 5 refuses "with a nil context: any `prior` entry or `carries` key" (`_lym/plan/05-review-round-checks.md:53`).
   The proposal needs a test the reviewer cannot get wrong. One option: the rules apply only when the block's own heading or line is present, and a named line in the block sets that.

2. LOW. Card 10's new sentence: "its in-window settle order gains the stalemate check after the automatic accept".
   Summary: the proposal updates only doc.go's in-window settle list. In the main stalemate case, a CONTINUE round with the budget not yet reached (`review_stalemate_rounds` defaults to 2), the check fires below the window. After the change, doc.go's below-window sentence still names only circling.
   Evidence: `internal/shedadapters/doc.go:158` reads "A CIRCLING verdict below the window escalates with cause circling." Card 10 calls `stalemated` from `settleUndecided` whether the window is open or not, and its first test row escalates at `StalemateRounds` 2 with CONTINUE verdicts.
   Also, "the automatic accept" fits two steps doc.go names: the `AutoAccept` step (`bouncer.go:1355`), which Card 10 places the check after, and the soft-cap rule's "records an automatic accept" step. The text should name which one it means.

3. LOW. The reworded overview Decision line in `00-overview.md` (F8): "a zero value that turns its own part off: … a nil `Profile.Round` is a full round with today's prior-rounds text".
   Summary: the reworded line now states Card 6's behavior for a nil `Profile.Round`, but the Decision's **Applies to:** still lists only cards 4, 5, 7, 10 and 12. Its lead clause "turns its own part off" is also false for the case it lists first: a nil `BurlerHalfDeps.Snapshot` still runs incremental rounds.
   Evidence: Card 6 says "`priorRoundsBlock` keeps today's text when `p.Round` is nil" (`06-prior-findings-block.md:18`). The Decision line itself says "an incremental round holds the BLOCKING rule alone".

The F1, F2, F3, F5, F6 and F9 proposals held up. For F2, seeding the index with `read-tree HEAD` before `git add -A` keeps the row where a committed edit and the same edit uncommitted give one id, and still stages deletions. For F1, parsing runs before checking, so the misspelled-key row reaches the stray-key refusal first.

Tool calls:
- One grep of Card 5 for its nil-context `prior` rule, and of `bouncer.go` for `AutoAccept`. My context lacked Card 5 and the auto-accept step's place in `settleUndecided`.

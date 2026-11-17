# Plan-Review round 1 — fixer report

Every finding's premise was checked against the code and holds; all nine are fixed.
`lym loom validate-plan` reports ok after the changes, with only the informational `create-new-unit` finding it reported before.

## Changes

- **F1** (fixed as proposed): Card 2's Intent drops the lenient-decode sentence.
  `ParseFixerReport` now refuses a top-level key other than `dispositions` and `spawned`, as `checkReviewKeys` does, naming the stray key and the two legal keys in a `way forward: rewrite the frontmatter of the fixer-report …` clause.
  `TestCheckFixerReport` gains a row refusing a misspelled `dispositons` key.
- **F2** (fixed as proposed): Card 1's `WorktreeTreeID` seeds the throwaway index with `git read-tree HEAD` before `git add -A`, as `newIntegrateBuild` does, so a tracked file stays staged whatever the ignore rules say.
  `TestFabric_WorktreeTreeID` gains a row for an edit to a tracked file an `info/exclude` pattern matches, and the existing ignored-file row now says "untracked".
- **F3** (fixed as proposed): Card 11 gains a Bound paragraph: the arbiter may accept a stalemated segment with open findings and no parent, through `acceptRound`'s `CarryOver` seam into `## Open risks`; `budget` stays refused; the segment stays Awaiting until an `accept` or `continue`.
- **F4** (fixed as proposed, one part fixed differently): Card 10 moves `internal/shedadapters/doc.go` from Extends to Edit, adds `internal/shedadapters#ErrEscalationMalformed`, and its Intent rewrites the four-cause texts in escalation.go's header and `ErrEscalationMalformed`'s doc, as well as doc.go's three-causes sentences.
  Fixed differently: the review proposed that doc.go's "in-window settle order gains the stalemate check after the auto-accept".
  doc.go's in-window list (`internal/shedadapters/doc.go:149-157`) describes `softCapRule`'s steps, not `settleUndecided`'s, and `AutoAccept` is not one of them (`internal/shedadapters/bouncer.go:1355-1369`).
  The check also fires below the window, which doc.go's sentence at line 158 names only for circling.
  So Card 10 instead says doc.go gains a sentence placing the check after the `AutoAccept` step and ahead of the soft-cap rule, yielding in the window to the rule's accept, and that the below-window sentence names the stalemate escalation too (self-check, fix-proposal 2).
  Card 12's `docs/overview.md` line also names the stalemate escalation beside circling and the hard cap.
- **F5** (fixed as proposed): Card 8 adds `internal/burlercli#validateReviewVerb` under Edit, and its Intent rewrites the help's only-top-level-keys sentence to name `prior`, the `validateReviewVerb` comment and validate.go's file header.
- **F6** (fixed as proposed): Card 10 drops the unreadable-ledger-in-window `TestBouncer_Stalemate` row; the `stalemated` bullet marks that yield as a fail-loud guard the judged-round read already rules out.
- **F7** (fixed as proposed): Card 6 now places the first-round and clean-room text in `priorRoundsBlock` for a nil context, and the stencil's rules below the `prior_rounds` marker as unconditional text.
- **F8** (fixed as proposed, reworded after self-check): the overview's nil-seams-keep-today Decision line now states what each zero value does: a nil `BurlerHalfDeps.Snapshot` writes no target id or changes file; a nil `BouncerConfig.Snapshot` or zero `StalemateRounds` never starts a stalemate escalation; a nil `Profile.Round` is a full round.
- **F9** (fixed as proposed): Card 10's counts-as-changed row attaches "though its two ids are equal" only to the three report variants.

## Self-check

- fix-claims, MEDIUM: the overview's "a zero `StalemateRounds` runs no stalemate check" contradicts Card 10's replay branch, which runs whatever `StalemateRounds` is. Fixed as proposed: it now says "never starts a stalemate escalation".
- fix-consistency, LOW: the nil-seams-keep-today slug and Rationale claimed more than the reworded Decision line. Fixed differently: the Rationale now says hand-built receivers keep working with no seam filled and review as today until they hold a complete new-format round. The slug is kept, since the plan's lens-review records cite it by name (`_lym/plan/lens-review.lens-coverage.md:33`, `_lym/plan/lens-review.lens-row-validity.md:12`).
- fix-proposal 1, LOW: Card 6's stencil-rule trigger could be misread on a later round with a nil context, whose text also lists prior paths. Fixed: the block opens with a `### Prior findings` line, the stencil rules apply only under that heading, and the test gains a row that a context renders the heading and a nil context does not.
- fix-proposal 2, LOW: the doc.go settle-order wording named only the in-window list and an ambiguous automatic accept. Fixed differently, as described under F4.
- fix-proposal 3, LOW: the overview's "turns its own part off" lead clause was false for a nil Snapshot, and Applies to lacked Card 6. Fixed: the lead now says "a legal zero value that no caller must fill", and Applies to lists Card 6.
- fix-affected-set 1, LOW: the `resolvePersistentPreRun` doc comment (`internal/burlercli/cli.go:65`) also says the verb reads one told path. Fixed: Card 8 lists `internal/burlercli#burlerCLI.resolvePersistentPreRun` under Edit and its Intent rewrites the comment.
- fix-affected-set 2, LOW: the comment over the cause constants (`internal/shedadapters/escalation.go:22`) says four. Fixed: Card 10's Intent names it beside `ErrEscalationMalformed`'s doc.

## Disputed

None.

## Deferred

None.

## Rulings

None.

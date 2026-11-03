# Fixer report — code review round 1

## Changes

- **F1** fixed as proposed (`internal/planparser/renumber_test.go`): the fixture's plan section gains a `Spans` range (cards first–first+2) and a `Pair` range (first–first+1). Insert-second and remove-second list both as kept; a new row, insert at the third position, pins the boundary (`Pair` unchanged and unlisted, `Spans` kept).
- **F2** fixed with my own change (`internal/planparser/renumber_test.go`): the proposal faulted the write of the bare-reference-only card. That fault fires before the card's bytes change, so the tree is restored whether or not the card was journaled (evidence: the fix-proposal lens, `applyRenumber` writes stayed cards after moved cards and before the overview). I added `referenceCardFour` and one row in `TestRenumber_IOFailureReversed` that faults the Card Index rewrite after that card was written, which fails when the card is not journaled. The write-fault row was dropped as redundant with the existing moved-card write rows.
- **F3** fixed with my own change (`internal/planparser/renumber_test.go`): delta's Card Index line carries a bare reference to a card that moves in every row (delta's Intent is explicit, so its card body is unchanged); the existing Card Index summary comparison fails if the scan rewrites that line. A heading row was not added: the moved-card comparison already pins a heading, since the scan would shift a renumbered heading a second time.
- **F4** fixed with my own change (`internal/planglyph/index_test.go`): rows for a test file the repository lacks and for one with a syntax error, both keeping the edit-test-file finding. The proposed same-base-name-in-another-directory row was dropped: `TOC` of the requested file path cannot list that entry, so the row duplicated the missing-file row without pinning `path.Base`. The test's doc comment is updated.
- **F5** fixed (`contracts/specs/loom-plan-spec.md`): two sentences. The second reads "A card id is never reported: both verbs rewrite one naming a moved card and leave one naming the removed card", since the proposed "both verbs always rewrite it" overstated the removed-card case.
- Fixture doc comment of `newRenumberFixture` updated to name the range lines and the Card Index reference.

`lym gate test ./internal/planparser ./internal/planglyph`: failed 0. Commits: one per finding group, then one for the self-check changes.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness, LOW, same-base other-directory row does not reach the `path.Base` match: fixed with own change (row dropped, see F4).
- fix-consistency, MEDIUM, spec said a card id is always rewritten: fixed (wording above).
- fix-consistency, LOW, `newRenumberFixture` comment stale: fixed.
- fix-consistency, LOW, `TestIndex_AppendsPlanChecks` comment stale: fixed.
- fix-proposal, LOW, same-base row: fixed (duplicate of the code-correctness finding, row dropped).
- fix-proposal, LOW, alpha write-fault row does not pin journaling: fixed (row dropped, overview-fault row kept).
- fix-affected-set, LOW, hyphen, em dash and `and` forms of `cardMentionRe` have no row: not fixed; the finding names a gap in the review's own proposal, which the rubric excludes as an improvement the review never asked for, and the fix did not make it worse.
- fix-affected-set, unchecked F2 write order: resolved by the fix-proposal lens's read of `applyRenumber`.

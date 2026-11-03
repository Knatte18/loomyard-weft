# Plan round 2 — fixer report

## Changes

- F1 (BLOCKING, fixed as proposed):
  - Card 1's Edit list targets `internal/boardcli#storeVerbs` instead of `internal/boardcli#Command`.
  - Card 1's Uses gains `internal/boardcli#TestPayloadUse_MatchesDeclaredKeys`.
  - Card 1's upsert-help bullet now covers both places in `storeVerbs` that spell the key. The `upsert` `Use` token gains `discussed?` after `on_land?`, since the test fails while the token and `UpsertPayloadKeys` disagree. The `Long` field list gains the `"discussed"` line.
  - Card 5's Edit list keeps `Command`, which builds `rename`, and adds `storeVerbs`.
  - Card 5's Intent places `getCmd` inside `storeVerbs`.
- F2 (LOW, fixed as proposed):
  - Card 8: `pastApproval` is `func(currentRow string) bool`. `preRun` drops `PastDiscussionApproval`'s error under one comment that names `resolveReviewTarget`'s `recipeUnknown` refusal as the reason.
  - Card 8 deletions:
    - the sentence that `pastApproval` has no error branch of its own;
    - "from `pastApproval`," in the generic error branch;
    - the sentence on `pastApproval`'s error in the `wayCommit` computation;
    - "with no error" in the fake's answer.
  - Card 9: `approvalRow` is `func() bool`, and it drops the error under the same comment.
  - Card 9 deletions:
    - the `approvalErr` field of `decisionFake`;
    - the `approvalRow` error clause of the status-read refusal;
    - the bullet that folded a `pastApproval` error into a recipe-read refusal;
    - the `approvalRow` error from the status-read test row.
  - The status read error keeps its refusal and its row.
- F4 (LOW, fixed as proposed): card 2's Intent no longer names `rename.go`'s file header as an edit. The `Board.Rename` doc comment carries the sentence.
- Parent decision "A resume after an accepted round does not store again", acted on as the parent's instruction: card 7's Intent cites the entry and adds that `lym loom decision store` refuses such a run until the segment's next approval, since its row is inside Discussion-Review and so not past the approval as card 6 judges it.
  The parent decision "A rename onto an orphan record removes it" matches card 2 as written, so card 2 needs no change for it.

`lym loom validate-plan` answers ok.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings; no finding left unchecked.

## Disputed

None.

## Deferred

None.

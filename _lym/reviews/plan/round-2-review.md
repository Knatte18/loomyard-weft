---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lym/plan/01-board-decision-record.md:22
    summary: "Cards 1 and 5 edit the wrong boardcli symbol: upsert and get are built in storeVerbs, not Command, and the upsert Use token that TestPayloadUse_MatchesDeclaredKeys couples to UpsertPayloadKeys is never updated"
    fix: >-
      Card 1, Edit list: replace `internal/boardcli#Command` with `internal/boardcli#storeVerbs`.
      Card 1, Intent, the bullet on the `lym board upsert` help: say the `upsert` command's `Use` token in `storeVerbs` gains `discussed?` after `on_land?`, and its `Long` field list gains the `"discussed"` line.
      Add `internal/boardcli#TestPayloadUse_MatchesDeclaredKeys` to card 1's Uses, since adding `discussed` to `UpsertPayloadKeys` turns that test red until the `Use` token spells it.
      Card 5, Edit list: keep `internal/boardcli#Command`, which builds `rename`, and add `internal/boardcli#storeVerbs`.
      Card 5, Intent: change "`getCmd`, inside `Command`" to "`getCmd`, inside `storeVerbs`".
      Checked with `lym loom validate-plan --overlay`: the baseline `{}` and the overlay of these two card files both report ok with no finding.
  - id: F2
    severity: LOW
    class: design
    location: _lym/plan/09-decision-store-verb.md:52
    summary: "Card 9 prescribes an approvalRow error refusal and an unspecified fold of the pastApproval error, but resolveReviewTarget already rejects the only input that makes either fail, and card 8 drops the same error silently in the wayCommit computation"
    fix: >-
      The real upstream check is `resolveReviewTarget`'s `recipeUnknown` refusal in `internal/loomcli/review.go`, which `decisionState.preRun` runs before any dep is built, so a recipe the accessors cannot parse never reaches `decisionVerb` or `decisionStoreVerb`.
      Card 8: make `decisionDeps.pastApproval` infallible, `func(currentRow string) bool`.
      The `preRun` closure calls `loomrecipe.PastDiscussionApproval(recipe, currentRow)`, returns `past` and drops the error, with one comment naming `resolveReviewTarget`'s `recipeUnknown` refusal as the reason.
      Delete the sentence that `pastApproval` has no error branch of its own and is handled by `hasPlan` and `rowWritten`, delete "from `pastApproval`," from the generic error branch of `copyRecordToBoard`, and delete the sentence that `pastApproval`'s error is not handled in the `wayCommit` computation.
      Card 9: make `approvalRow` infallible the same way, `func() bool`, filled from `loomrecipe.DiscussionApprovalRow(recipe)` with the error dropped under the same comment.
      Delete the `approvalErr` field of `decisionFake`, the test row "an `approvalRow` error is refused naming the re-run", the "or an `approvalRow` error" clause of the status-read refusal, and the bullet that folds a `pastApproval` error into a recipe-read refusal.
      The status read error keeps its refusal and its row.
      Neither change touches a target list, so `validate-plan --overlay` needs no new run for it.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lym/plan/02-record-follows-entry.md:32
    summary: "Card 2 adds a removal of an orphan record at the rename target and a second post-save rename error, and D4 prescribes neither; only the failed move is D4's rename error"
    fix: >-
      The parent records the orphan-at-target handling in the decision record under an `Added after Discussion` heading with `lym loom decision add`: a rename of a flagged entry with no file at the old slug's name removes a file at the new slug's name, and a failed removal is reported as the rename's error after the save.
      D4's own rationale, that an orphaned file would read as a record of a later entry reusing the slug, licenses the removal, but the decision record states the second error nowhere.
      Until the entry exists, the fixer leaves card 2 as written and defers this finding with the missing decision as the reason, rather than dropping the removal, since without it a flagged entry renamed onto an orphan `decision-<new>.md` serves that orphan as its record.
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/plan/02-record-follows-entry.md:46
    summary: "Card 2 says rename.go's file header is edited, but rename.go is not a target and cannot be listed beside Board.Rename without redundant-file-target refusing it"
    fix: >-
      Card 2, Intent, the sentence "`Board.Rename`'s doc comment and `rename.go`'s file header say this": drop the file header.
      The header's sentence "Every refusal happens before the save, so a refused rename writes nothing" stays true after the card, so the header needs no edit; the `Board.Rename` doc comment, already a target, carries the new "record move follows the save" sentence.
  - id: F5
    severity: LOW
    class: decision
    location: _lym/plan/07-discussion-approve-seam.md:82
    summary: "Card 7 states that on an accepted round a resume does not store again, but D6 and its acceptance line say a resume stores, and no decision record entry records the departure"
    fix: >-
      The parent records under an `Added after Discussion` heading with `lym loom decision add` that D6's resume sentence holds for a converged, skipped and zero-round settle only: on an accepted round the accept is settled before the seam runs, so a failed store is not retried by the resume, the run re-enters Discussion-Review, and the record is stored at the next approval.
      Card 7's Intent, the paragraph that ends "the record is stored at the segment's next approval", adds that `lym loom decision store` refuses such a run until then, since its row is inside Discussion-Review and so not past the approval as card 6 judges it.
      Until the entry exists the fixer adds only the Intent sentence and defers the decision part with the missing entry as the reason.
---

### [BLOCKING] Cards 1 and 5 edit the wrong boardcli symbol and miss the upsert Use token

**Location:** `_lym/plan/01-board-decision-record.md:22`, `_lym/plan/05-board-get-record.md:4`
**Issue:** Card 1 lists `internal/boardcli#Command` and says the `lym board upsert` help lives there.
`internal/boardcli/cli.go` builds `upsert` and `get` in `storeVerbs` (line 475 on), while `Command` (line 44 on) builds `promote`, `rename`, `prune` and the maintenance verbs, as `internal/boardcli/doc.go` says.
The `upsert` command's `Use` token at `cli.go:479` spells every payload key by hand, and `TestPayloadUse_MatchesDeclaredKeys` (`payload_test.go:36`) fails the build's tier-0 gate when it differs from `UpsertPayloadKeys`.
Card 1 adds `discussed` to `UpsertPayloadKeys` and never edits that token, so the card's own gate goes red, and its `Command` target names a function that holds none of the text it changes.
Card 5 makes the same slip for `getCmd`, which sits at `cli.go:700` inside `storeVerbs`; its `Command` target is right only for the `rename` help.

### [LOW] Card 9's recipe-parse refusal has no input past resolveReviewTarget

**Location:** `_lym/plan/09-decision-store-verb.md:52`, `_lym/plan/08-decision-add-copies.md:57`
**Issue:** `decisionState.preRun` calls `resolveReviewTarget`, whose `recipeUnknown` case refuses a seed naming no embedded recipe before any dep exists.
`PastDiscussionApproval` and `DiscussionApprovalRow` fail only for such a recipe, so no real input reaches card 9's `approvalRow` error refusal, whose only row drives a fake `approvalErr`.
Card 9 also says a `pastApproval` error is "folded into that recipe-read refusal" and names no mechanism, and card 8 drops the same error in the `wayCommit` computation, so the two cards treat one error differently.
Round 1 settled the card 8 half by sending the error to the generic store-error branch; that branch is still reached by the failing-store row, so it is not raised again, and only the card 9 refusal and the two inconsistent treatments remain.
The finding names the upstream check: `resolveReviewTarget`.

### [MEDIUM] Card 2's orphan removal and its error go beyond D4

**Location:** `_lym/plan/02-record-follows-entry.md:32`
**Issue:** D4 prescribes one error after a saved rename, "a failed move is reported as the rename's error".
Card 2 adds a second: when the old slug's file is absent it removes `decision-<new>.md`, and a failed removal is returned as `rename saved the entry, but the orphaned decision record ... could not be removed`.
Round 1 asked for the removal itself, and the fixer's self-check then gave its failure the second error, so this part was never judged against D4.
The step is sound, and its test row at line 56 reaches the case: a flagged entry with no file renamed onto an existing `decision-<new>.md` answers found false.
But the decision record's Auto-mode assumptions call such an orphan harmless, and nothing in it licenses the removal or the second error, which card 5 then documents in the `rename` help and the board skill.
`Board.RemoveTask`, `Board.Prune` and `Board.MergeTasks` gain no error: the card logs a failed removal at Warn and returns nil, so those methods and their callers' "nothing saved" reading stay correct.

### [LOW] Card 2 edits a file header it does not target

**Location:** `_lym/plan/02-record-follows-entry.md:46`
**Issue:** The Intent names `rename.go`'s file header as an edit, but the card's targets are the members `Board.Rename`, `Board.boardCriticalSection` and `doc.go`.
A file self glyph beside `Board.Rename` is refused by `redundant-file-target`, so the header edit has no admissible target, and its sentence "Every refusal happens before the save" is still true.

### [LOW] Card 7 leaves the accepted-round resume departure unrecorded

**Location:** `_lym/plan/07-discussion-approve-seam.md:82`
**Issue:** D6 says a failed store halts the run and "a resume re-settles and stores again", and the D6 acceptance line says "a resume stores it".
Card 7 states, correctly against `TestBouncer_Circling_AcceptWithFailedCommitResumesByClearingWithoutApprove` and `bouncer.go:1207`, that an accept is settled before `Approve` runs, so on that path the resume archives the generation and the store waits for the next approval.
The card's `TestApproveDiscussion_Real` rows and the new `TestBouncerEntry_ApproveSeam` subtests do not test a resume, so the divergence is untested as well as unrecorded.
After that resume the run's row is inside Discussion-Review, so `lym loom decision store` refuses it, which the card does not say.

## Rejected

- Card 2, errors after a saved `RemoveTask`, `Prune` and `MergeTasks`: the removal failure is logged at Warn and returns no error, so these methods gain no post-save error and need no doc change; only `Rename` does, and card 2 and card 5 document it.
- Card 2, rename onto a slug whose `decision-<new>.md` exists: with the old file present, `os.Rename` replaces the target; with it absent, the card removes the target and the named row reaches the case; a move that fails with a regular-file orphan at the target needs a filesystem fault no test reaches, so it is not raised.
- Card 2, the refused-rename row: the card builds its own `boardengine.Config` with a `ValidateSlug` that refuses the target, and `checkNewSlugs` (`slugrule.go:30`) judges the renamed slug after the mutation and before the save, so the row reaches the refusal and the file stays at the old name.
- Card 7, store failures at approval: a done entry returns nil with a Warn, and an absent entry and a board `OpenHub` cannot open fail the settle, as the decision record's `Added after Discussion` entry on store failures rules; the done-entry and absent-entry rows of `TestApproveDiscussion_Real` match it; the new `approve_seam` comment in `tapestry-recipe.yaml` and the `TestApproveSeam_FailsToBuild` doc comment state the accepted values and not the failure rule, so they agree with it.
- Cards 8 and 9, `committed()` in `copyRecordToBoard`: a read error is a propagated I/O error, and a not-found answer shares that branch with no text of its own, so it needs no row of its own; card 9 reaches its own not-found refusal with a named row.
- Card 5, the `doc.go` sentence on stored-only payload keys: the plan makes only `discussed` stored-only and the sentence is edited to name it; the sentence's omission of `beam` and `owner_machine` predates the plan.

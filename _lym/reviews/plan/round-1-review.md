---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: lens:guard-reach
    location: _lym/plan/07-discussion-approve-seam.md
    summary: approveDiscussion swallows ErrRecordEntryDone although D6 says a failed store fails the settle, while an absent entry or an OpenHub failure still halts the run at Discussion-Bouncer, a case SetRunStatus and the board status seam tolerate
    fix: Bring card 7's store-failure policy and D6 into one rule. Either drop the ErrRecordEntryDone branch from approveDiscussion so every store error fails the settle as D6 says, and drop the done-entry row of TestApproveDiscussion_Real in favour of one asserting the settle fails; or have the parent record the departure with lym loom decision add, naming which store failures (done entry, and whether an absent entry or an unopenable board) only warn, and then make card 7's Wiring bullets and its TestApproveDiscussion_Real rows match that entry.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lym/plan/02-record-follows-entry.md
    summary: card 2 makes a saved remove, prune or merge return an error when a record file cannot be deleted, which D4 does not prescribe, and no card documents the new post-save error on those Board methods or on the board verbs and skill whose callers read an error as nothing saved
    fix: In card 2's Intent, make a failed removal of decision-<slug>.md log at Warn through internal/logger and return no error, since an orphan file is unread while its slug carries no flag, and keep the returned error for the failed rename move alone, as D4 requires. Then say in card 5's Intent, which already edits internal/boardcli#Command and extends plugins/ly/skills/board/SKILL.md, that the rename verb's help and the skill's rename text state that a rename can report an error after it is saved, when the record move fails, and what that error names.
  - id: F3
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-decision-add-copies.md
    summary: the pastApproval error branches in cards 8 and 9 cannot occur, since PastDiscussionApproval fails only on an unparseable recipe and hasPlan and rowWritten in decisionVerb, and approvalRow in the store verb, parse the same recipe first and refuse
    fix: In card 8, drop the judge-whether-the-run-passed error text of copyRecordToBoard, the or-when-it-errs fallback of wayCommit and the pastApproval-error row of TestDecisionVerb_CopiesRecordToBoard, and have a pastApproval error take the generic store-error branch. In card 9, fold the pastApproval error into the recipe-read refusal that approvalRow's error already gives, and drop its separate test row. Name hasPlan, rowWritten and approvalRow as the upstream checks in each card.
  - id: F4
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-decision-add-copies.md
    summary: copyRecordToBoard's branch for a committed record HEAD does not hold cannot occur, since the copy runs only after a successful commit or after afterFailedCommit found every entry at HEAD
    fix: In card 8, treat a not-found committed() inside copyRecordToBoard as the read error it would be, with no text of its own, and drop the TestDecisionVerb_CopiesRecordToBoard row for a committed record HEAD does not hold.
  - id: F5
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/02-record-follows-entry.md
    summary: a rename of a flagged entry whose old record file is absent leaves any decision-<new>.md already at the target name in place, so DecisionRecord serves an orphan of another entry as this entry's record, the case D4's rationale exists to prevent
    fix: In card 2's rename bullet, when decisionFileName(from) is absent, remove decisionFileName(renamedTo) if it exists, so the renamed entry reads as flagged without a file; add a TestDecisionRecordFollowsEntry row that renames a flagged entry with no file onto a slug whose decision-<new>.md exists and asserts DecisionRecord answers found false.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/02-record-follows-entry.md
    summary: TestDecisionRecordFollowsEntry's refused-rename row needs a refusing ValidateSlug seam, but the card builds its rows on newRenameBoard, whose Config sets none, so checkNewSlugs refuses nothing there
    fix: In card 2's test paragraph, state that the refused-rename row builds its own boardengine.Config with a ValidateSlug that refuses the target slug, rather than newRenameBoard's board.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/05-board-get-record.md
    summary: internal/boardcli/doc.go line 29 states the stored-value-only payload rule for front and on_land, and no card adds discussed to it
    fix: In card 5's Intent, which already edits internal/boardcli/doc.go, add that the upsert sentence naming front and on_land as accepted only as the stored values also names discussed.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/07-discussion-approve-seam.md
    summary: card 7 leaves two comments untrue in files it edits, the Plan-Bouncer comment saying Discussion-Bouncer never gets approve_seam, and TestApproveSeam_FailsToBuild's doc comment saying any value but plan fails to build
    fix: In card 7's Recipe paragraph, reword the whole Plan-Bouncer comment block in tapestry-recipe.yaml, including the sentence that neither Discussion-Bouncer nor Code-Bouncer gets the key; in its Tests list, have the TestApproveSeam_FailsToBuild bullet also reword that test's doc comment to name plan and discussion as the accepted values.
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/03-design-prefix-refusal.md
    summary: card 3 says lym config board edits the file without LoadConfig, but setChecked loads the board config through configreg CheckLoad before and after a --set; the way forward still works only because a write that clears the failure passes
    fix: In card 3's Intent, replace the sentence about lym config board with one saying that a --set is checked by loading the config after the write and passes when the new value loads, so the named step clears the refusal.
---

### [MEDIUM] Card 7's store-failure policy departs from D6

**Location:** `_lym/plan/07-discussion-approve-seam.md`, Wiring bullets.

**Issue:** D6 says "A failed store fails the settle as a failed `ApprovePlan` does".
Card 7 instead logs a store refused with `ErrRecordEntryDone` at Warn and returns nil.
The decision record carries no `Added after Discussion` entry for that departure.
The card's reason is sound: a done entry makes no store progress, so a halt would repeat on every resume.
The same logic covers an absent entry and an unopenable board, which card 7 still turns into a halt.
`Board.SetRunStatus` (`internal/boardengine/board.go:189`) writes nothing for an absent or done entry, and the board status seam only warns on failure (`internal/loomcli/wiring.go:217-223`).
Card 7 must follow D6, or the parent records the narrower rule.
The guard-reach lens raised the absent-entry half; the handler raised the done-entry half, and the two are merged here.

### [MEDIUM] A post-save removal error reaches callers documented as saving nothing

**Location:** `_lym/plan/02-record-follows-entry.md`, second step bullet and failure paragraph.

**Issue:** D4 names one post-save error, the failed rename move.
Card 2 also returns an error when a saved remove, prune or merge cannot delete a record file.
`Board.RemoveTask`, `Board.Prune` and `Board.MergeTasks` (`internal/boardengine/board.go:214-234`, `:393-409`) are not re-documented for it.
Their callers read an error as "nothing saved": the board verbs, `pairteardown/claim.go` and `fronttask/hub.go`.
An orphaned record file is harmless while its slug is unflagged, because `DecisionRecord` requires the flag.
Logging the failure is enough there; the rename move keeps its error.

### [LOW] Unreachable `pastApproval` error branches

**Location:** `_lym/plan/08-decision-add-copies.md` (copy and `wayCommit`) and `_lym/plan/09-decision-store-verb.md` (Refusals).

**Issue:** `PastDiscussionApproval` errors only when the embedded recipe fails to parse.
In `decisionVerb`, `hasPlan` and `rowWritten` (`internal/loomcli/decision.go:688-698`) parse the same seed recipe first and refuse.
In the store verb, `approvalRow` does the same.
The planned error texts and their fake-driven test rows therefore pin states the verbs never reach.

### [LOW] Unreachable not-found branch in `copyRecordToBoard`

**Location:** `_lym/plan/08-decision-add-copies.md`, The copy.

**Issue:** `decisionVerb` reaches the copy only after `deps.commit()` succeeded, or after `afterFailedCommit` returned landed (`internal/loomcli/decision.go:585-591`).
That check requires `committedLacksEntry` to find every entry at HEAD.
In both cases HEAD holds the record, so the branch and its test row pin a state that cannot occur.

### [LOW] A rename can adopt an orphaned record file at the target name

**Location:** `_lym/plan/02-record-follows-entry.md`, rename bullet.

**Issue:** An absent old file is "no error" and nothing else happens.
A flagged entry with no file can come from an older binary's rename or a failed move.
When such an entry is renamed onto a slug whose `decision-<new>.md` survives, `DecisionRecord` answers that file as this entry's record.
The surviving file could be left by an older binary's remove, by a removal that failed, or by hand.
D4's rationale is that "an orphaned file would read as a record of a later entry reusing the slug".

### [LOW] Refused-rename row has no refusing seam

**Location:** `_lym/plan/02-record-follows-entry.md`, test rows.

**Issue:** `newRenameBoard` (`internal/boardengine/rename_test.go:24-28`) builds its `Config` without `ValidateSlug`.
So `checkNewSlugs` (`internal/boardengine/board.go:144`) refuses nothing on that board, and the row as written cannot reach the post-mutation refusal it means to pin.

### [LOW] boardcli doc keeps the payload rule to front and on_land

**Location:** `_lym/plan/05-board-get-record.md`; evidence `internal/boardcli/doc.go:29`.

**Issue:** Card 1 adds `discussed` to the keys accepted only as their stored value.
The boardcli package doc still names only `front` and `on_land` for that rule.

### [LOW] Card 7 leaves two comments untrue

**Location:** `_lym/plan/07-discussion-approve-seam.md`; evidence `contracts/recipes/tapestry-recipe.yaml:300-305` and `internal/loomrecipe/approveseam_test.go:95`.

**Issue:** The Plan-Bouncer comment says "Neither Discussion-Bouncer nor Code-Bouncer gets this key".
`TestApproveSeam_FailsToBuild`'s doc says any value but `plan` fails to build.
Card 7 makes both false, but rewords only the "only row in the whole list" clause and the test's row.

### [LOW] Card 3 misstates how `lym config board` checks a write

**Location:** `_lym/plan/03-design-prefix-refusal.md`.

**Issue:** `setChecked` (`internal/configcli/configcli.go:273-296`) runs the module's `CheckLoad`, which is `boardengine.LoadConfig`, before and after the write.
It refuses only a new load failure.
The named way forward works, but for a different reason than the card gives, and a writer following the card's sentence would describe the verb wrongly in code or doc.

## Focus items settled

- Fidelity: D1 to D10 and every Constraint are each carried.
  - D1: cards 1 and 3.
  - D2 and D3: card 1, including the merge that removes and upserts a slug.
  - D4: card 2.
  - D5: card 4, including the lost report for a written record file whose entry origin lacks.
  - D6: card 7.
  - D7: cards 6 and 7.
  - D8: card 8, including the path where a failed commit is found to have landed.
  - D9: card 9.
  - D10: card 5, with all four `--record` refusals.
  - No card adds unlicensed scope beyond F1's done-entry branch.
- Test economy: not a defect for any named pair.
  - `TestStoreDecisionRecord` and `TestDecisionRecordFollowsEntry` pin different behaviours.
  - `TestPastDiscussionApproval` pins the row rule, and `TestDiscussionApprovalRowsCarryApproveSeam` pins the seam guard over every loom recipe.
  - The edit to `TestShippedRecipe_SeamsWiredOnPlanBouncerOnly` is forced, since tapestry's new key would otherwise fail it.
  - `TestDecisionVerb_CopiesRecordToBoard` and `TestDecisionStoreVerb` test two verbs with separate caller and refusal code.
- Refusal reach: every prescribed refusal has a named input and test, except the branches in F3, F4 and F6.
  Card 3's `design_prefix` rows, the empty prefix included, are not stopped by any earlier check in `LoadConfig` or `configengine`.
- Card order: holds. Card 9 follows card 8's `decisionDeps` and `decisionFake` edits, cards 8 and 9 follow cards 6 and 7, and cards 2 and 4 use only card 1.
- Attack surface: each card states its bound.
  - Card 1 states the run-lock exemption for the flag and the file only.
  - Card 7 states the goto bound.
  - Card 9 states its three callers and the past-approval bound.
- Member glyphs, re-sign arrows and `Extends`: hold.
  - `applyDesignFile` and `afterFailedCommit` carry arrows.
  - `boardCriticalSection`, `decisionVerb` and `preRun` keep their signatures.
  - The `Extends` targets (`SKILL.md`, and the `doc.go` files of loomrecipe and loomcli) are only added to.
- Board-entry rules: hold.
  - No card edits this task's entry, files a task or promotes a note.
  - `TestApproveDiscussion_Real` and the `TestCLI` step write entries only into their own temporary hubs.

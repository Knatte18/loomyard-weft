---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/26-merge-in-fixer.md
    summary: The AlignFrom fork point is written inside align after the merge and the gate, so an error stop in that window loses it and the resumed Merge-In skips the fixer over parent commits it never aligned
    fix: In card 26's Intent, write AlignFrom into the entry's attempt record right after readBefore and before the merge (the merge base readBefore read, when the merge record holds no aligned tip and the record no AlignFrom), not inside align; add a TestMergeIn_Fixer row where a run's first Merge-In stops with an error after the merge (for example a gate infrastructure error) and the resumed entry, whose merge is now up to date, still spawns the merge-in fixer.
    origin: lens:guard-reach
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-fixer-run.md
    summary: A timeout, no-report, dirty or pushed end on the entry's last turn leaves Spent equal to the cap with no spent note, and cards 26 and 29 define no resume for that state, so the resume spawns a turn past the cap against D4
    fix: In cards 10, 26 and 29, key the exhausted-entry path on Spent reaching the row's cap whatever the Halt note says; card 26 states that a resumed Merge-In with no attempts left reads the operator's fix as the alignment (passing gate advances the aligned tip and is Done, failing gate halts naming the log), and card 29 states that a resumed Finalize with no attempts left over a failing gate reroutes or halts as a spent cap does; card 11 states run is never called with zero turns left; add a row per row test for a resume after a last-turn timeout.
    origin: lens:guard-reach
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/10-fixer-attempts.md
    summary: The undecodable-attempt-record halt names removing the file to grant a fresh cap but carries no LandingStopPrefix, so HandsBack leaves it unmarked and a tackler or arbiter may take that step and reset the fixer cap
    fix: In cards 10, 26 and 29, the undecodable-attempt-record Stuck with Halt opens its reason with LandingStopPrefix, so card 34's HandsBack marks it and only the orch's lyx loom resume lifts it; card 34's TestHandsBack gains a row for that reason at Merge-In and at Finalize.
    origin: lens:way-forward
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/22-recipe-tails.md
    summary: D5's rubric change is uncarried, since no card rewrites loom-rubric-code-review's measuring stick from the plan alone to the decision record and the plan, nor makes it review the code in general
    fix: In card 22's Intent, state that loom-rubric-code-review's measuring-stick section names the decision record and the plan as its answer key, its Do-not-flag rule names both as never the subject, and its Also-flag section reviews the code in general (correctness beyond plan compliance) as well as against both, per D5.
    origin: handler
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/51-drop-retired-row-names.md
    summary: Card 51 deletes loomshed.NamePRGate, but TestResumeVerb's own-leftover row (resume_test.go:222) and TestLoopLockExit's awaiting rows (looplock_test.go:75-76) still name it and no card moves them off it, so the build breaks at card 51
    fix: Add internal/loomcli#TestResumeVerb and internal/loomcli#TestLoopLockExit to card 51's Edit list, and its Intent says the own-leftover row of TestResumeVerb names another current row such as Finalize and TestLoopLockExit's awaiting rows name a current awaiting row.
    origin: lens:affected-set
  - id: F6
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/13-darn-write-no-reroute.md
    summary: Card 13 removes the Darn.ReadRejection, Darn.ClearRejection and other Darn seams, but TestWire_DarnRecipe's darnSeams list (wiring_darn_test.go:49) still names them and card 13 does not list that test
    fix: Add internal/loomcli#TestWire_DarnRecipe to card 13's Edit list, and its Intent says the test's darnSeams list shrinks to DarnTable, Darn.Commit and Darn.LatestOutcome.
    origin: lens:affected-set
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/26-merge-in-fixer.md
    summary: Card 26 does not say what a nil Deps.CodeReviewOff or a failed config read answers, so the budget-0 cap at Merge-In can silently not apply, while card 29 states its RerouteReviewed nil default fails safe
    fix: In card 26's Intent, state that a nil CodeReviewOff, or one whose config read fails, reads as the budget being 0, so the caps apply and a fixer change is judged rather than let through, mirroring card 29's RerouteReviewed default.
    origin: handler
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/21-code-perch-keys.md
    summary: Card 21 says the D14 retirements of later cards append to its rename table, but card 23 creates a separate retiredLoomKeys drop list that cards 44 and 45 append to
    fix: In card 21's Intent, replace the clause saying later retirements append to the same table with one saying the later cards' retired keys go to the drop list card 23 adds, which runs after this rename table.
    origin: handler
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/plan/15-round-gate-drops-failure-record.md
    summary: TestVerifyGate_RoundScenario writes PublishFailure records (verifygate_integration_test.go:296-375); card 15 lists it but its Intent names only TestVerifyGate_Scenario dropping failure-record rows, and card 46's rename does not list it
    fix: In card 15's Intent, state that TestVerifyGate_RoundScenario drops its failure-record rows and its writeFailure helper with the confirm step, so no PublishFailure reference is left for card 46's rename.
    origin: lens:affected-set
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/plan/47-deploy-branches.md
    summary: internal/landingshed/doc.go:133-136 keeps a paragraph explaining require_pr_to_base as the pull-request switch, and neither card 47 nor card 42 names it
    fix: Add internal/landingshed/doc.go to card 47's Edit list, and its Intent says that paragraph is replaced by one stating deploy_branches is read only by batten's Land row to decide the deploy.
    origin: lens:affected-set
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/plan/52-docs-and-pattern.md
    summary: contracts/specs/shed-recipe-spec.md:59 still says every kind tells the Bouncer Env.RejectReview after card 31 renames it ReviewInput; card 52 edits that spec only to drop PR-row examples
    fix: In card 52's Intent, the shed-recipe-spec.md sentence also says the spec names Env.ReviewInput where it named Env.RejectReview.
    origin: lens:affected-set
  - id: F12
    severity: LOW
    class: design
    location: _lyx/plan/11-fixer-run.md
    summary: The row where a turn returns past fixer_timeout_min ends as the timeout end names no clock the closure measures through, so the row is not deterministically reachable as PATTERN-test-speed asks
    fix: In card 11's Intent, state that the gate closure measures each turn's duration through the verifyGate's injectable now clock (verifygate.go:157), and that fakeFixerSeats advances that clock by the scripted turn duration.
    origin: lens:guard-reach
  - id: F13
    severity: LOW
    class: consistency
    location: _lyx/plan/29-finalize-gate-and-fixer.md
    summary: TestFinalize_GateAndFixer's row that a verified tree runs no gate and proceeds to the landing repeats TestFinalizeVerify_Lands' verified-tree-skips row, which card 28 keeps
    fix: Drop the verified-tree row from TestFinalize_GateAndFixer in card 29's Intent; TestFinalizeVerify_Lands' verified tree skips row (finalize_verify_test.go:98) keeps pinning it.
    origin: handler
---

### [MEDIUM] Merge-In's fork point is lost on an error stop before align

**Location:** `_lyx/plan/26-merge-in-fixer.md`

**Issue:** Card 26 fixes `AlignFrom` "the first time `align` finds no aligned tip", taking "the merge base Merge-In read before this entry's first merge".
`align` runs after the merge, the post-merge clean check and the `landing` gate, and `readBefore` re-reads `MergeBase(ParentBranch)` on every `Call` (`internal/landingshed/mergein.go:146`, `:272`).
After an error stop in that window, the resumed entry's merge is up to date and its fresh merge base is already post-merge.
`HoldsWork(align-from, merged tip)` then reads true, and over a passing gate the entry is Done with no fixer, which breaks D4 and the criterion "the first Merge-In of a run included".

### [MEDIUM] A cap spent by a non-spent end has no defined resume

**Location:** `_lyx/plan/11-fixer-run.md`, cards 10, 26 and 29

**Issue:** Card 11 charges each turn when it starts.
A `timeout`, `no-report`, `dirty` or `pushed` end on the last turn therefore leaves `Spent` equal to the cap with an empty `Halt` note, since card 10's notes are only `spent`, `over-cap` or empty.
Card 26 defines the no-attempts-left resume only for the `spent` and `over-cap` notes, and card 29 defines none.
A resume from that state spawns the fixer with zero turns left, which either charges a turn past `merge_fix_attempts` or `finalize_fix_attempts` or stamps a gate with zero attempts.
Either way it contradicts D4's "no resume grants a fresh cap", and the halt's way forward has no defined next state.

### [MEDIUM] The undecodable-attempt-record halt can be lifted by an agent and resets the cap

**Location:** `_lyx/plan/10-fixer-attempts.md`, cards 26 and 29

**Issue:** The halt's way forward is to remove the record by its path, "which grants the entry a fresh cap", then `lyx loom resume`.
No card gives this halt `LandingStopPrefix`, and card 34's `HandsBack` matches only that prefix, or the reroute-cap prefix at Finalize.
So the halt is unmarked, and a tackler or arbiter may take the named step itself.
D11 sends the landing halts to the orch so that no driver resume grants fresh fixer attempts (`decision-record.md:242`).

### [MEDIUM] D5's rubric content is uncarried

**Location:** `_lyx/plan/22-recipe-tails.md`

**Issue:** D5 decides that the Code-Perch's "answer key is the decision record and the plan", and that its rubric "reviews the code in general and against both".
Today's `loom-rubric-webster-review.md:15` says "The measuring stick is the plan", and its Do-not-flag rule (`:62-64`) names the plan alone.
Card 22 renames the rubric's title and segment wording only, and defers review content to cards 31 and 32.
Card 31 adds the fixer report as an input and card 32 changes the range, so no card rewrites the measuring stick or adds the general code review.
Card 22's recipe `fasit.paths` naming both is not enough while the rubric text still names the plan alone.

### [MEDIUM] Card 51 leaves two references to NamePRGate

**Location:** `_lyx/plan/51-drop-retired-row-names.md`

**Issue:** `internal/loomcli/resume_test.go:222` (`TestResumeVerb`, the "own leftover at another row" row) uses `loomshed.NamePRGate`.
Card 43 drops only the PR-Gate row at `:200`.
`internal/loomcli/looplock_test.go:75-76` (`TestLoopLockExit`) uses it too; card 41 rewrites that test's expectation but not its producer.
Card 51 deletes the constant without listing either test, and its ImpactSummary claims its remaining references are the tests it lists.

### [MEDIUM] Card 13 misses TestWire_DarnRecipe's seam list

**Location:** `_lyx/plan/13-darn-write-no-reroute.md`

**Issue:** `internal/loomcli/wiring_darn_test.go:49` lists `Darn.ReadRejection`, `Darn.ClearRejection`, `Darn.PublishFailure`, `Darn.RunStart`, `Darn.TaskHead`, `Darn.Dirty`, `Darn.ParentTips` and `Darn.IsAncestor` as seams the darn seed fills.
Card 13 removes them all, but only the later cards 31 and 33 list that test, so card 13's own gate runs a stale seam list.

### [LOW] Card 26 does not say how a nil or failing CodeReviewOff reads

**Location:** `_lyx/plan/26-merge-in-fixer.md`

**Issue:** The budget-0 cap at Merge-In is gated on the new told `Deps.CodeReviewOff`, which loom fills from the config at call time.
The card does not say what a nil seam or a failed config read answers.
If either reads as "budget above 0", the cap is bypassed and the fixer's change goes unreviewed.
Card 29 states the safe default for its analogous `RerouteReviewed`, so the two cards disagree.

### [LOW] Card 21 and card 23 disagree on where retired keys go

**Location:** `_lyx/plan/21-code-perch-keys.md`

**Issue:** Card 21 says "the D14 retirements of later cards append to the same table", meaning `retiredPerchKeyRenames`.
Card 23 instead creates a separate drop list, `retiredLoomKeys`, applied through `yamlengine.DropKeys`, and cards 44 and 45 append to it.

### [LOW] Card 15's Intent does not cover TestVerifyGate_RoundScenario's failure-record rows

**Location:** `_lyx/plan/15-round-gate-drops-failure-record.md`

**Issue:** `internal/loomshed/verifygate_integration_test.go:296-375` writes and removes `verifytree.PublishFailure` records in `TestVerifyGate_RoundScenario`.
Card 15 lists that test but its Intent names only `TestVerifyGate_Scenario` dropping failure-record rows, and card 46's rename does not list the test.

### [LOW] landingshed's doc keeps the require_pr_to_base paragraph

**Location:** `_lyx/plan/47-deploy-branches.md`

**Issue:** `internal/landingshed/doc.go:133-136` explains `require_pr_to_base` as the pull-request switch.
Card 47 renames the key, but extends only `internal/battenshed/doc.go`.
Card 42's doc drop names only the Publish, PR-Align, PR-Gate and pull-request paragraphs, so the paragraph survives under a retired name.

### [LOW] shed-recipe-spec still names Env.RejectReview

**Location:** `_lyx/plan/52-docs-and-pattern.md`

**Issue:** `contracts/specs/shed-recipe-spec.md:59` reads "Every kind also tells the Bouncer `Env.RejectReview`".
Card 31 renames that field to `ReviewInput`, and card 52 edits the spec only to drop its PR-row examples.

### [LOW] Card 11's timeout row names no clock

**Location:** `_lyx/plan/11-fixer-run.md`

**Issue:** The closure ends a turn that "ran past `fixer_timeout_min`", and the fake "takes a scripted turn duration".
The card names no clock the closure measures with, although `verifyGate` already carries an injectable `now` (`internal/landingshed/verifygate.go:157`).
Without one, the row is not deterministic as `PATTERN-test-speed` asks.

### [LOW] A Finalize test row duplicates an existing row

**Location:** `_lyx/plan/29-finalize-gate-and-fixer.md`

**Issue:** `TestFinalize_GateAndFixer`'s first row, "a verified tree runs no gate and proceeds to the landing", is the behavior `TestFinalizeVerify_Lands`' "verified tree skips" row pins (`internal/landingshed/finalize_verify_test.go:98`).
Card 28 edits that test and keeps it.

## Focus verdicts

- Fidelity: D6 is carried by cards 22 and 23, D8 by cards 26, 29, 33 and 48, D12 by cards 26, 29 and 46, and D14 by cards 21, 23, 44, 45, 47, 48 and 49.
  Every other Decision and Constraint is carried, apart from D5's rubric content (F4).
- Scope: card 52's PATTERN shortening and its moved clauses are licensed by the `Added after Discussion` entries.
  Card 50's timing tables and `tools/tokencount` are readers forced by D1's removal of the row names and D5's `code` run subdirectory, which the Scope's "Readers updated in the same commit" licenses.
  Neither is a defect.
- Card order: cards 21, 22, 23, 24 and 46 use no symbol a later card creates, and delete none an earlier card still needs.
  Card 51's missed readers are F5.
- Granularity and test economy: no private helper has its own card; F13 is the one repeated row.
- Attack surface and reachability: the fixer caps, the hand-back mark and the Finalize reroute each state their bound, apart from F3 and F7.
  Every prescribed refusal has a named reaching test, apart from F12's clock.
- Edit cards: the re-sign arrows are present where signatures change, and the ImpactSummaries state conclusions.
- Board and verify: no card edits this task's board entry, and no card carries a `**Verify:**` line.

## Rejected

- way-forward F3 (card 30's lock re-check halt names no step that lifts an owner pause): the halt's wording is D10's own decided way forward, "`lyx loom resume` once no owner pause stands".
  An owner pause is the operator's deliberate act, so the plan follows the answer key there.

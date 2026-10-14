---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-validate-discussion-darn.md:25
    summary: Card 6 gives validate-discussion a weft commit on a darn run but its Intent states neither what that commit lets through nor what bounds it; a writer edit made after validation and then re-validated is committed, so card 7's committed-copy check no longer sees the record as touched
    fix: In card 6's Intent, after the paragraph that starts with On a clean darn check, add one paragraph stating the new edge and its bound, for example - The commit is a new side effect of a verb any session of the darn worktree may run. It lets a writer's post-validation edit, followed by another clean run, become the committed copy, so decision add's working-copy-equals-committed-copy check (card 7) passes over it. It is bounded to a darn seed (c.recipe is shedrun.RecipeDarn), to commitDiscussion's pathspec (the discussion directory alone), and to a record that passes the whole darn check, whose history check refuses any commit that drops an Added after Discussion entry, so no appended entry can be erased this way. No target changes.
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/07-decision-add-darn.md:39
    summary: Card 7 prescribes a post-commit refusal for a committed record that fails ParseAddedDecisions, but no TestDecisionVerb_Refusals row reaches that parse-failure branch; only the post-commit read-error and missing-entry branches are tested
    fix: In card 7's Intent, give decisionFake a field holding the bytes its commit leaves as the committed copy (overriding the copy-the-file behavior), and add a TestDecisionVerb_Refusals row with afterCommit set whose post-commit committed copy holds an Added after Discussion heading that does not parse (one with no date), asserting the refusal carries decisionWayEntryLost. The targets internal/loomcli#decisionFake, internal/loomcli#decisionFake.deps and internal/loomcli#TestDecisionVerb_Refusals are already listed; no target changes.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/09-darn-describe-record.md:17
    summary: Card 9 makes DarnDescribeSpec read the decision record but leaves its own doc comment (internal/landingshed/describe.go:152), which says the darn Describe has no decision record or run record to read, unmentioned
    fix: In card 9's Intent, after the sentence about DarnDescribeInputs and DarnDescribeSpec, add that DarnDescribeSpec's doc comment says the darn Describe session reads the decision record beside the board entry and still reads no run record. internal/landingshed#DarnDescribeSpec is already a target; no target changes.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/13-darn-recipe-rows.md:5
    summary: Card 13 adds the top-level test TestDarnPerch_ConvergedOverAnOpenDecisionFindingAcceptsAndCarriesItOver for behavior existing and already-extended tests cover
    fix: Drop card 13's whole Create group (TestDarnPerch_ConvergedOverAnOpenDecisionFindingAcceptsAndCarriesItOver and decisionOpenConvergedBouncerJudge), its Edit bullet cmd/lyx/testdata/test-budget.yaml, its Uses bullets internal/loomrecipe#writeBouncerJudge and internal/loomrecipe#haltingProducer, and the three Intent sentences that describe the helper, the new test and the 54-to-55 budget raise. Coverage stays with internal/shedadapters#TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit (a CONVERGED round over open MEDIUM entries returns Done and carries them over, class-agnostically), TestBouncerRowsCarryOverTheirOwnSegment (Darn-Bouncer's carry_over is Darn-Review) and card 13's extended TestEscalation_DarnPerchRuleContinuesThenHardCapAcceptReachesDescribe (a darn perch accept files a Darn-Review carry-over entry). validate-plan --overlay with this change reports no finding beyond the baseline's informational ones.
---

The plan was judged against `_lyx/discussion/decision-record.md` alone: no `_lyx/loom/rework/` directory exists, so this is generation 0.
The overview's `## verify:` section, the card order, every Decision D1–D11 and every Constraint were checked.
Code claims were checked against the tree: `wireLightweight` runs after `resolveRecipe`, so it knows `c.recipe`; `wire` already fills `CommitDiscussion` and `CarryOver` for every recipe; `gitrepo.PathRevisions`, `FileAtRevision` and `ErrPathNotAtRevision` exist as card 2 uses them; `discussionparser.Validate` does not parse added headings, so card 3's record-side parse finding is reachable through the gate; the `landingDeps` callers are `loomPreRun`, `buildLoomShed` and the tests card 12 lists; and the test-budget rows card 3 and card 13 raise match today's counts of 18 and 54.
`lyx loom validate-plan --overlay` with `{}` reports only informational `create-new-unit` findings.

### [MEDIUM] validate-discussion's new commit is unbounded in card 6's Intent

**Location:** `_lyx/plan/06-validate-discussion-darn.md:25`

**Issue:** On a darn run, card 6 makes `lyx loom validate-discussion` commit the discussion directory through the fabric at every clean run, which the verb never did before.
The rubric's attack-surface rule asks the card adding an edge to say in its Intent what the edge lets through and what bounds it.
Card 6 states neither.
The edge matters for D5's bound: card 7 refuses `decision add` when the working copy differs from the committed copy, and that is the check meant to catch a writer editing its record after validating it (D3).
A writer that edits after validation and re-runs `validate-discussion` gets the edit committed, so card 7's check passes over it.
D4 permits the re-commit ("A later clean run ... commit[s] again"), so this is not a defect in the design.
What is missing is the card stating the edge and naming the history check as the bound that still protects appended entries.

### [LOW] Card 7's post-commit parse-failure refusal has no test reaching it

**Location:** `_lyx/plan/07-decision-add-darn.md:39`

**Issue:** The Intent says "a read or parse failure there is refused with the same way forward".
The `TestDecisionVerb_Refusals` rows on line 52 cover the post-commit missing entry and the post-commit read error, but no row makes `ParseAddedDecisions` fail on the committed copy.
The fake as specified can't produce that input: its commit either copies the file's bytes, which passed the post-append discussion check, or leaves the earlier committed copy, which passed the pre-append history check.
So the branch has no named input or test that reaches it.

### [LOW] DarnDescribeSpec's doc comment still says it reads no decision record

**Location:** `_lyx/plan/09-darn-describe-record.md:17`

**Issue:** `internal/landingshed/describe.go:152` reads: "DarnDescribeSpec builds the shuttleengine.Spec for the Describe session of a darn run, which describes the diff from the board entry with no decision record or run record to read."
Card 9 makes `DecisionRecordPath` a required input of that function and updates `wireDarn`'s doc, `darnwiring.go`'s header and `landingshed/doc.go`.
It never mentions this doc comment, which the change falsifies.
The decision record's scope lists docs that describe darn as having no record.

### [LOW] Card 13's new perch test repeats covered behavior

**Location:** `_lyx/plan/13-darn-recipe-rows.md:5`

**Issue:** Go's Bouncer refuses a CONVERGED verdict only over an open BLOCKING entry (`internal/shedadapters/bouncer.go:654`).
On a CONVERGED settle it calls the carry-over seam with every open MEDIUM-or-worse entry, whatever its class.
`TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit` already pins that: open `design` and `scope` MEDIUM entries are carried over and the row returns Done.
The darn-specific half is Darn-Bouncer's `carry_over: Darn-Review` and the darn recipe filing a Darn-Review entry on accept.
`TestBouncerRowsCarryOverTheirOwnSegment` pins the first, and the card's own extension of `TestEscalation_DarnPerchRuleContinuesThenHardCapAcceptReachesDescribe` pins the second.
Go treats a `decision`-class entry no differently on this path, and the judge's own CONVERGED ruling is an LLM call the fake judge only stubs.
So the new top-level test, its helper and the budget raise add no behavior that isn't already pinned.

## Rejected

- Card order (focus 2): card 3 uses only existing `discussionparser` symbols; card 4 uses card 3; card 5 uses card 4; card 6 uses cards 2 and 4; card 7 uses card 6's `discussionHistory` and card 3; card 12 uses card 11's measures; card 13's gate and PR-Align row rest on cards 5, 6 and 12. No card uses a later card's symbol.
- Verify coverage (focus 3): every Go target is in the root module and `./...` covers it in both run tiers. The contract stencils, specs and recipes are embedded into `contracts/*` packages whose tests run there. `cmd/lyx/testdata/test-budget.yaml` is read by `cmd/lyx`'s tests. The section's order matches the plan stencil's.
- Edit-card glyphs (focus 5): every Edit card naming symbols lists member glyphs. The re-signed members (`composePrompt`, `DiscussionSpec`, `DiscussionTable`, `landingAlignmentDeps`, `landingDeps`) carry the arrow. The others keep their signatures and gain only struct fields or bodies.
- Attack surface on cards 3, 4 and 7 (focus 4): card 7 states D5's bound in full. Card 4's gate only adds refusals and commits only a passing record. Card 3 states why a garbled historical heading contributes nothing. Card 5's missing-`CommitDiscussion` guard is a fail-loud wiring guard.
- Fidelity (focus 1): D1–D11, A2, A5 and every Constraint map to a card. No card edits a board entry, adds a `darn.yaml` key, splits darn's rows or renames a row. The `d7-bouncer-premise-false` entry sits in a carry-over block and is not part of the answer key.
- Card 5's resolver subtest runs both a failing and a passing darn case. These are subtest rows, not a new test function, and each one shows the history seam selects the darn gate, so this was not raised.

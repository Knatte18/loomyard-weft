---
verdict: APPROVED
findings:
  - id: F2
    severity: LOW
    class: design
    location: internal/runplan/postbatch.go:303
    summary: "The wrapped DroppedCreates error is fixed for a read failure, but its malformed-heading branch in internal/discussionparser/append.go#DroppedCreates is unreachable (addedDecisionSpans keeps only headings isAddedDecisionHeading already parsed), so its doc sentence is false; and the way forward says re-run the same webster verb, which is wrong on the two weaverengine call sites."
    fix: "In internal/discussionparser/append.go#DroppedCreates, delete the parseAddedDecisionHeading error branch and the doc line that says an unparsable added-decision heading is an error, reading the title from the parsed heading as the span guarantees. In internal/runplan/postbatch.go:303, change the closing step from re-running the same webster verb to re-running the verb that failed, so the clause holds for weaverengine's checkRecordable and checkStoppedBatch as well as record-batch and recover-batch."
    origin: assist:assist
    carries: F2
  - id: F5
    severity: LOW
    class: design
    location: internal/burlerengine/roundcheck.go#namedByAmendment
    summary: "namedByAmendment admits any .md file in the plan directory, so an amending entry that uses the word overview exempts 00-overview.md, which is no card."
    fix: "In namedByAmendment, return false for the plan overview file before matching names. Export planparser's overviewFileName (or add a planparser predicate that reports whether a plan-directory file is a card) and use it, never a second literal. Add a TestCheckReviewRound row whose amending entry names a card and the word overview and whose new BLOCKING finding sits on _lym/plan/00-overview.md, refused with does not list."
    origin: assist:assist
    carries: F5
  - id: F9
    severity: LOW
    class: design
    location: internal/shedrecipe/entries_burlerhalves_test.go:159
    summary: "The new subtest pins that only Plan-Burler-Review reads the halt file, but no test pins that only that row gets the amendment reader; dropping the AmendmentTitles assignment in internal/shedrecipe/entries_burler.go:186 leaves every test green."
    fix: "Extend the HaltReasonReachesOnlyThePlanReviewRow subtest of TestBurlerReviewEntry_ProfileMapping, or add a sibling subtest, that lays out a complete round 1, writes an amendment record at env.AmendmentPath listing the title of an added entry in env.DecisionRecordPath, captures the round-2 profile, and asserts Round.AmendingTexts holds that entry for Plan-Burler-Review and is empty for review-row."
    origin: assist:assist
    carries: F9
prior:
  - id: F1
    status: resolved
    answer: "withoutDroppedCreates now strips the dropped handle's declarations from the card and its group; BindHandles counts OwnedHandles, which reads Declarations, so the dropped handle raises no bind-count-mismatch. TestPostBatchChecks_DroppedHandleDeclarationIsNotBound pins the two-handle case."
  - id: F2
    status: open
    answer: "The read-error path carries the record path and a way forward and a test reaches it; the malformed-heading branch it also claims is dead code, and the clause names a webster verb on the weaver paths."
  - id: F3
    status: resolved
    answer: "CardNotDoneError.DroppableCreate is set for create-not-done alone, and createTargetWayForward offers the drop only then; the rename-only row expects amend and recover-batch."
  - id: F4
    status: resolved
    answer: "priorFindingsBlock lists amending entries from an earlier snapshot under their own line and writes the amending paragraph whenever AmendingTexts is non-empty."
  - id: F5
    status: open
    answer: "Whole-token matching and the plan-directory check fix the substring and non-card-path cases, but the overview inside the plan directory still matches."
  - id: F6
    status: resolved
    answer: "burler-cluster-fan.md, burler-assist-cluster-fan.md and burler-assist-consolidation.md all keep origin webster on the halt finding."
  - id: F7
    status: resolved
    answer: "doc.go now says refusal and prompt text name a loom verb only as a step for the run's parent."
  - id: F8
    status: resolved
    answer: "The parent directive's last bullet passes the same targets again with --drops-create."
  - id: F9
    status: open
    answer: "Halt delivery to spawned, probed and resumed reviewers and the plan-row-only halt wiring are tested; the plan-row-only amendment reader is not."
  - id: F10
    status: resolved
    answer: "DroppingOneTargetKeepsTheOtherBlocking asserts the error names only the undropped ref and the one warning names the dropped one."
  - id: F11
    status: resolved
    answer: "RecordBatch, weaver TestRun and TestRun_Reentry each run with a record that drops the unresolved target, and the weaver fake reports not-done only while the target is still asked about."
  - id: F12
    status: resolved
    answer: "The row asserts none follows the parent-decisions paragraph and no Decision 1 line appears."
  - id: F13
    status: resolved
    answer: "newFindings admits one outside origin webster finding and refuses a second, pinned by SecondNewWebsterFindingUnderAHaltReasonIsRefused."
  - id: F14
    status: resolved
    answer: "TestParseAddedDecisions carries the keep marker naming what it pins."
  - id: F15
    status: resolved
    answer: "The Long text again states the --from title and PATTERN uniqueness, the withdraws refusal and the sampler's --by operator limit."
  - id: F16
    status: resolved
    answer: "Fixed differently and accepted: tapestry-recipe.yaml gives Plan-Burler-Review the same run_subdir as Plan-Bouncer, and wiring.go's CommitPlan already reads it through loomrecipe.BouncerRunSubdir."
---

### [LOW] The dropped-Create error claims a branch it never reaches and names a webster verb on weaver paths

**Location:** internal/runplan/postbatch.go:303

**Issue:** `DroppedCreates` iterates spans from `addedDecisionSpans`, which skips any heading `isAddedDecisionHeading` rejects (`internal/discussionparser/identity.go:120`), and that check is `parseAddedDecisionHeading` itself.
So the `is not an added-decision heading` error can never fire, and the doc sentence promising it is false; a malformed heading's `Drops Create:` line is ignored and the batch blocks with the normal create-not-done way forward.
The discussion check refuses such a heading on every `lym loom decision add`, so this is reachable only by hand-editing, and it fails safe.
Separately, the wrapper's way forward ends "re-run the same webster verb", while `weaverengine/record.go:162` and `weaverengine/resume.go:440` reach the same error from a weaver run.
The read-error half of F2 is fixed and `TestPostBatchChecks_UnreadableDecisionRecordNamesTheWayForward` reaches it.

### [LOW] The amendment exemption still admits the plan overview

**Location:** internal/burlerengine/roundcheck.go#namedByAmendment

**Issue:** The directory and `.md` checks pass for `_lym/plan/00-overview.md`, and stripping `00-` leaves the whole token `overview`, so an amending entry that mentions the overview exempts a file that is not a card.
The decision record's scope exempts "the cards it names".
The practical effect is small, since an amendment that mentions the overview may well need an overview finding, so this is carried at LOW.

### [LOW] The plan-row-only amendment reader has no reaching test

**Location:** internal/shedrecipe/entries_burlerhalves_test.go:159

**Issue:** `burlerReviewEntry` gives `AmendmentTitles` and `HaltConflictDir` to the Plan-Burler-Review row alone.
The new subtest pins the halt half; `TestBuildRoundContext` passes its own reader, so nothing reaches the shedrecipe assignment of the amendment reader through a round-2 profile.

## Rejected

- The assistant's F2 at BLOCKING: the unreachable branch fails safe (the batch blocks with a valid way forward) and lym's own writer cannot produce the malformed heading, so it is carried at LOW.
- Mutation survivor `internal/runplan/postbatch.go:197` (drop `DroppableCreate = true`): `TestFailCardNotDone_WayForward` builds its error through `runplan.CardNotDoneFor` and expects the drop route for create-not-done, so it kills this mutant; the survivor is an artifact of running only runplan's own tests.
- Mutation survivors `internal/shedadapters/webster.go:64` and `:71`: each negates only a logging condition and changes no outcome.

The rest of the diff holds against the parent's scope entry.
The Create check reads the decision record, and a dropped target warns instead of blocking.
Record-batch and recover-batch give the same way forward.
The drift halt names the full report and the parent's `lym drift amend` route.
An `--amends-plan` decision exempts the cards it names.
The halt reason reaches Plan-Burler-Review as an `origin: webster` finding.
The master and implementer prompts mark a later decision as superseding an earlier one.

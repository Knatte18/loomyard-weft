---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/runplan/postbatch.go:368
    summary: "A dropped Create declared through a plan handle still blocks the batch: the done-check skips it, but BindHandles receives the unfiltered in.Cards and reports bind-count-mismatch for the handle no delta symbol matches, which returns ErrCardNotDone."
    fix: "In PostBatchChecks, pass BindHandles a copy of the batch's cards whose Declarations omit every declaration whose Handle is a dropped target, built in withoutDroppedCreates beside the filtered TargetGroups, and keep in.Cards for the scope guard, RebindResigns and drift. Add a row to TestPostBatchChecks_DroppedCreateSkipsItsDoneCheck (or a sibling row in its table) whose card declares two handles, drops one, has the delta create the other, and asserts the pass returns no ErrCardNotDone."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/runplan/postbatch.go:282
    summary: "PATTERN-refusal-way-forward: a DroppedCreates read error or an unparsable Added after Discussion heading now fails every record-batch and recover-batch call with a raw error and no way forward clause, since both verbs return a non-ErrCardNotDone error unchanged."
    fix: "Wrap the DroppedCreates error in PostBatchChecks with the record path and a trailing way forward clause, such as fixing the named heading of the decision record through the run's parent (lym loom decision replace) and re-running the same webster verb. Add a row reaching it, a record with a malformed added heading, that asserts the message carries the way forward."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/websterengine/recordbatch.go:106
    summary: "A rename-not-done finding gets the --drops-create route in its way forward, but withoutDroppedCreates filters Create groups only, so following that route leaves the batch failing on the same finding."
    fix: "Carry whether the not-done findings include a Create (not only a Rename) from runplan.CardNotDoneError into failCardNotDone, and have createTargetWayForward offer the --drops-create route only for a create-not-done finding. Change the TestFailCardNotDone_WayForward row RenameNotDoneOffersAmendDropAndRecover to expect the amendment and recover-batch without the drop route."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/prompt.go:481
    summary: "priorFindingsBlock renders and marks amending entries only among AddedTexts and returns early when AddedTexts is empty, so a pending amendment already in the previous snapshot disappears from the prompt while the round check still exempts its cards."
    fix: "In priorFindingsBlock, list every RoundContext.AmendingTexts entry with amendsPlanMark and write the amending paragraph whenever AmendingTexts is non-empty, independent of AddedTexts, writing an entry once when it is in both lists. Add a TestComposePrompt_PriorRounds row with AmendingTexts set and AddedTexts empty asserting the marked entry and the paragraph."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/roundcheck.go:326
    summary: "namedByAmendment admits files no amending entry names: a substring match of the stem or slug lets an entry naming 14-card-slug-extra admit 15-card-slug.md, a stem like overview or test match ordinary words, and the directory is ignored so a non-card file with a matching stem passes too."
    fix: "Match only plan card files (the plan directory's card paths) and require the stem or slug to appear in the entry text as a whole token, bounded by a non-identifier character on both sides, not as a plain substring. Add TestCheckReviewRound rows for an overlapping longer slug and for a non-card file sharing a card's stem, each refused."
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: assist:assist
    location: contracts/stencils/burler/burler-cluster-fan.md:32
    summary: "The cluster consolidation rule tags every kept finding origin lens or handler, and the assistant consolidation rule tags assistant findings with its own origin, so the halt finding loses origin webster in a cluster round and its changed-file exemption never applies."
    fix: "In the consolidation sentence of burler-cluster-fan.md, and in burler-assist-consolidation.md's keep rule, state that a finding raising the plan executor's halt reason keeps origin webster, as burler-review-format.md requires, instead of a lens, handler or assistant origin."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/websterengine/doc.go:346
    summary: "The package doc still says no refusal or prompt text names a loom verb, while the diff adds lym loom decision add to createTargetWayForward and lym loom resume to the drift route."
    fix: "Rewrite the sentence at doc.go:346 to state the rule as it now holds: refusal and prompt text name a loom verb only as a step for the run's parent, never as a step for Master or a fork."
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/parent/parent-directive-parent.md:33
    summary: "The parent rule has the parent fold added entries with decision replace and record each folded call again with a plain decision add before a Webster row, but DroppedCreates reads only added entries, so a folded drop is lost and the create-not-done block comes back."
    fix: "Extend the last bullet of the parent rule so a re-recorded call that carried Drops Create lines passes the same targets again with --drops-create."
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/burlerreview.go:379
    summary: "No test seeds a halt file and checks that the review producer hands HaltReason to the reviewer, nor that only Plan-Burler-Review gets the halt directory and amendment reader; the mutants at burlerreview.go:360, 379 and 383 survived."
    fix: "Extend an existing BurlerReviewProducer Call test to write a halt reason into HaltConflictDir and assert the profile the runner receives carries it, also under a differing probe fan. Extend TestBurlerReviewEntry_ProfileMapping, or the shedrecipe row table it belongs to, to assert the Plan-Burler-Review row reads the halt file and another review row does not."
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/runplan/postbatch_test.go:46
    summary: "TestPostBatchChecks_DroppedCreateSkipsItsDoneCheck drops every ref of the card, so an implementation that skips all Create refs once any is dropped passes; no row shows an undropped unresolved target still blocks."
    fix: "Add a case to the test whose record drops one of the two refs and assert the pass returns ErrCardNotDone naming only the undropped ref, with the warning naming only the dropped one."
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/websterengine/recordbatch_test.go#TestRecordBatch_DoneChecksBlockOnUnresolvedCreate
    summary: "No record-batch, recover-batch or weaver test runs with a decision record that drops a Create, so removing DecisionRecordPath from postBatchInputs or the weaver PostBatchInputs sites leaves every test green."
    fix: "Add a row to TestRecordBatch_DoneChecksBlockOnUnresolvedCreate whose geometry's DecisionRecordPath names a record dropping the unresolved Create target and assert RecordBatch records the batch with the dropped-target warning; add the same case to the weaver record test that reaches checkRecordable."
  - id: F12
    severity: MEDIUM
    class: consistency
    origin: lens:code-tests
    location: internal/websterengine/template_test.go:905
    summary: "The row master renders none for no parent decision lost its check that the section renders none, and no commit carries a Loosened-Check trailer; an empty rendering now passes."
    fix: "In that row, assert the prompt holds the parent decisions paragraph followed by the line none, using the stencil's literal text, besides the existing notContains checks."
  - id: F13
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/roundcheck.go:306
    summary: "Any number of new findings tagged origin webster skip the changed-file rule while a halt reason is set, so the exemption is not tied to the one halt it exists for."
    fix: "In newFindings, admit at most one new origin webster finding outside the changed files and refuse a second with a rewrite naming the rule; add a TestCheckReviewRound row with two webster findings."
  - id: F14
    severity: LOW
    class: consistency
    origin: assist:assist
    location: internal/discussionparser/append_test.go#TestParseAddedDecisions
    summary: "The redundancy report lists TestParseAddedDecisions as contained by TestCheckAddedDecisionsKept, but it pins the exact parsed entries in record order, which the covering test does not assert, and it carries no keep marker."
    fix: "Add a //testtiming:keep line directly above func TestParseAddedDecisions naming what it pins, the exact parsed author, date and title of each entry in record order."
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/decision.go:1984
    summary: "The decision add help text rewrite dropped rules still in force, among them that --from refuses two entries sharing a title or PATTERN entry and that --pattern-action withdraws is refused when the record declares none."
    fix: "Restore those two refusal statements and the sampler session's --by operator limit in the Long text, shortening other sentences to stay under the CLI length cap."
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/burlercli/validate.go:31
    summary: "planReviewRunSubdir restates the recipe's run_subdir plan as a second literal, while the review producer picks the plan segment by row name, so a renamed subdir silently makes validate-review disagree with the round gate."
    fix: "Pick the plan segment in validate-review from the recipe's Plan-Burler-Review row run_subdir, read through the loom recipe, instead of the planReviewRunSubdir literal, and delete the constant."
---

Reviewed `git diff 57015a52cd1f436ce96f5137889045e9a2ce5d7d..HEAD` against the parent's entry "Scope after the corrupted board body", which replaces the corrupted Goal body.
Every change traces to its three parts or to #178 part 1.
The auto rebaseline after a fix (#170 bullet 2) is not built.

### [BLOCKING] A dropped handle-declared Create still blocks the batch

**Location:** internal/runplan/postbatch.go:368

**Issue:** `withoutDroppedCreates` filters only `TargetGroups`.
`BindHandles` is called with `in.Cards`, and `planparser.OwnedHandles` reads `Card.Declarations` (`internal/planparser/handle.go:85`).
`planglyph.BindHandles` emits a blocking `bind-count-mismatch` for a card whose handles the delta matches fewer of (`internal/planglyph/handle.go:492`).
`PostBatchChecks` then returns `ErrCardNotDone` (`postbatch.go:393-397`).
So the drop the parent recorded does not unblock a Create declared through a `plan:` handle, which the decision says must not block.
That error is a plain wrapped error, not a `CardNotDoneError`, so the batch fails with no create way forward at all.

### [BLOCKING] A failed decision-record read stalls every batch with no way forward

**Location:** internal/runplan/postbatch.go:282

**Issue:** `DroppedCreates` returns an error for a read failure and for an `### Added after Discussion` heading that does not parse (`internal/discussionparser/append.go:239-262`).
`RecordBatch` and `recoverBatch` return any non-`ErrCardNotDone` error unchanged (`recordbatch.go:578-580`, `recoverbatch.go:729-731`).
The new failure path of these webster verbs therefore names no way forward, which PATTERN-refusal-way-forward requires, and has no reaching test.

### [MEDIUM] Rename-not-done is offered a drop route that cannot clear it

**Location:** internal/websterengine/recordbatch.go:106

**Issue:** `failCardNotDone` calls `createTargetWayForward` for create-not-done and rename-not-done alike.
The new text offers `--drops-create`, but `withoutDroppedCreates` skips only `CardTypeCreate` groups (`postbatch.go:225-246`), and the rename done check reads `g.Pairs`.

### [MEDIUM] Pending amendments vanish from the prompt after one round

**Location:** internal/burlerengine/prompt.go:481

**Issue:** `BuildRoundContext` keeps every pending amending entry in `AmendingTexts`, including those already in the previous snapshot.
`priorFindingsBlock` returns early when `AddedTexts` is empty and marks only entries in `AddedTexts`.
The reviewer is no longer told about the amendment, while the round check still exempts its cards.

### [MEDIUM] The amendment matcher admits files no entry names

**Location:** internal/burlerengine/roundcheck.go:326

**Issue:** `namedByAmendment` drops the directory, keeps the stem and its digit-stripped slug, and admits any of them of four or more characters found anywhere in an amending entry's text.
An entry naming `14-card-slug-extra` admits `15-card-slug.md`.
A card stem like `overview` or `test` matches ordinary words.
A source file sharing a card's stem passes too.
The parent's entry exempts the cards a decision names, not files whose names overlap them.

### [MEDIUM] Consolidation rules strip origin webster

**Location:** contracts/stencils/burler/burler-cluster-fan.md:32

**Issue:** The cluster consolidation tags every kept finding `lens:<name>` or `handler`, and the assistant rule tags its kept findings with the assistant's origin (`burler-assist-consolidation.md:11`).
A finding holds one origin, so in a cluster round the halt finding reaches the round check without `origin: webster`.
`newFindings` then holds it to the changed-file rule, and the halt cannot reach Plan-Burler-Fix when it sits on an unchanged card.

### [MEDIUM] websterengine's doc contradicts the new texts

**Location:** internal/websterengine/doc.go:346

**Issue:** The doc still says "No refusal or prompt text names a loom verb", but `recordbatch.go:117` names `lym loom decision add` and `verifygate.go:611` names `lym loom resume`.
The parent's entry asks for both routes, so the doc sentence is what is stale.

### [MEDIUM] The parent's fold rule loses a dropped Create

**Location:** contracts/stencils/parent/parent-directive-parent.md:33

**Issue:** After a `decision replace` fold, the rule has the parent record each folded call again with `lym loom decision add`.
`DroppedCreates` reads only added entries, and the rule does not tell the parent to pass `--drops-create` again.
The create-not-done block then returns at the next Webster row.

### [MEDIUM] Halt-reason delivery to the reviewer is untested

**Location:** internal/shedadapters/burlerreview.go:379

**Issue:** The mutants at `burlerreview.go:360`, `379` and `383` survived.
No producer test seeds a halt file and checks the profile the runner gets.
No recipe test checks that only the Plan-Burler-Review row gets `HaltConflictDir` and `AmendmentTitles`.

### [MEDIUM] Selective dropping is untested

**Location:** internal/runplan/postbatch_test.go:46

**Issue:** The test drops both refs of its card, so it cannot tell a drop of the named target from a drop of the whole Create group.

### [MEDIUM] The verbs' drop path is untested end to end

**Location:** internal/websterengine/recordbatch_test.go#TestRecordBatch_DoneChecksBlockOnUnresolvedCreate

**Issue:** Only `runplan` tests a decision record that drops a Create.
The `DecisionRecordPath` fields in `postBatchInputs` (`recordbatch.go:902`), `weaverengine/record.go:170` and `weaverengine/resume.go:448` can be deleted with every test still green.
A verb-level row would also have caught the handle case in F1.

### [MEDIUM] The none-rendering check was removed

**Location:** internal/websterengine/template_test.go:905

**Issue:** The row "master renders none for no parent decision" used to compare its output with the rendering of the literal `none`.
The diff removes that check and asserts only the absence of `Decision 1:` and the marker.
No commit of the range carries a `Loosened-Check:` trailer.

### [LOW] The webster exemption is unbounded

**Location:** internal/burlerengine/roundcheck.go:306

**Issue:** While a halt reason is set, every new BLOCKING finding tagged `origin: webster` skips the changed-file rule, with nothing tying the exemption to the one halt finding the prompt asks for.

### [LOW] TestParseAddedDecisions needs its keep marker

**Location:** internal/discussionparser/append_test.go#TestParseAddedDecisions

**Issue:** The redundancy report lists it as contained by `TestCheckAddedDecisionsKept`.
That test asserts findings, while this one asserts the exact parsed entries in order, and the diff changed it.
The rubric asks for a `//testtiming:keep` finding for such a test.

### [LOW] The decision add help dropped live rules

**Location:** internal/loomcli/decision.go:1984

**Issue:** Commit e5b6b3a6 shortened the `Long` text to fit the CLI length cap.
In doing so it dropped the `--from` duplicate refusal (whose `decisionWayFromTwice` still exists), the withdraw refusal and the sampler's `--by operator` limit.

### [LOW] validate-review restates the plan run_subdir

**Location:** internal/burlercli/validate.go:31

**Issue:** `planReviewRunSubdir = "plan"` repeats `run_subdir: plan` from `contracts/recipes/tapestry-recipe.yaml`.
The producer side keys the same behaviour on the row name.

## Focus verdicts

- Create check: a dropped Create skips only the target it names in the done-check; record-batch and recover-batch run the same checks and give the same way forward, with F1, F2 and F3 against it.
- `--drops-create`: the several-entries and malformed-target refusals hold, and the written line round-trips through `DroppedCreates`. An unknown target matches no Create and exempts nothing.
- Verify gate: the run-level entry branch names the full report whether it cuts or not. The drift way forward names `lym drift amend <slug> --entries <file>`, which `driftcli/amend.go` admits at a blocked Webster. The other branches already name `report.LogPath`.
- Round check: F5 and F13 cover what the exemptions admit beyond the named cards and the one halt finding.
- Halt reason: written on every Stuck, cleared on Done, and a read failure fails the review row loudly. The survivors at `webster.go:64` and `:71` negate log-only branches, so they are no finding.
- Survivors at `burlerreview.go:360-383`: they drop the halt-reason read and its assignment to the profiles. F9 covers this.
- Decision order: the master prompt numbers entries in record order, and both stencils say a later decision supersedes an earlier one.
- Scope: the weaverengine, envkit and validate-review changes are forced by the traced changes. The auto rebaseline after a fix is not implemented.

## Rejected

- PATTERN-shed-recipe-registry violation for `Env.AmendmentPath` and `Env.HaltConflictDir` (code-affected-set, BLOCKING): the entry's "row values by key" governs config values. These fields are told geometry paths, like the existing row-specific told seams on `Env`.
- `webster_test.go:290` and `:448` writing a halt file under the package directory (code-correctness note): those tests reach the unrecognized-outcome and cancelled-context paths, which write no halt file.
- Missing rows for `amendingEntryTexts` error paths and the `DroppedCreates` heading error (code-tests, LOW): the first only propagate a read error. F2's fix adds the reaching row for the second.
- Missing empty-target row for `dropsCreateRefusal` (code-tests): the condition is shared with the backtick row, and `TestAppendDecision_DroppedCreates` covers empty and newline targets at the writer.
- Assistant F2 and F3 at BLOCKING: both are kept, as F5 at MEDIUM and F14 at LOW. Neither breaks the decision's required behaviour or a PATTERN entry.
- Recipe-wiring tests at LOW (code-tests 4): merged into F9.

---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/runplan/postbatch.go:286
    summary: A dropped Create declaration still blocks completion during handle binding.
    fix: Carry the dropped-target filter through handle binding in PostBatchChecks, excluding the matching Create declarations as well as their target refs from the completion view while retaining every undropped declaration and Rename check. Keep the original cards for scope accounting. Extend the post-batch behavior test with two declared handles, one dropped and one present in the delta, and assert completion passes binding.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/roundcheck.go:343
    summary: Amendment exemptions admit substring matches and non-card files instead of only the named cards.
    fix: Resolve amendment references to actual plan-card identities and compare the finding location with those card paths. Match complete identifiers rather than substrings, and support explicitly named short card identities without the minimum-stem heuristic. Extend TestCheckReviewRound with a longer overlapping slug, a source file sharing a card stem, and an explicitly named short card.
  - id: F3
    severity: BLOCKING
    class: consistency
    origin: lens:code-tests
    location: internal/discussionparser/append_test.go:412
    summary: The retained parser test lacks the required explanation for a contained redundancy candidate.
    fix: Add a testtiming:keep directive immediately above TestParseAddedDecisions explaining that it pins exact parsed author, date and title values in record order, which TestCheckAddedDecisionsKept does not assert. Retain the test.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/prompt.go:486
    summary: Pending amendments disappear from the review instructions once the preceding snapshot contains them.
    fix: Render all RoundContext.AmendingTexts and their exemption instructions independently of AddedTexts in priorFindingsBlock. Emit an entry once when it belongs to both lists. Extend the existing prompt table with a pending amendment already in the previous snapshot, both with and without an unrelated newly added entry.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/burlerengine/roundcheck.go:306
    summary: Cluster and assistant consolidation can overwrite the origin required by the halt exemption.
    fix: Make the origin contract consistent across the new halt instructions, review format, burler-cluster-fan.md and burler-assist-consolidation.md. Preserve origin webster for a retained halt finding instead of replacing it with lens, handler or assistant provenance, and keep the round-check exemption limited to that halt classification.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/burlerreview.go:379
    summary: Tests do not assert that the review producer delivers the halt reason to spawned and resumed reviewers.
    fix: Extend the existing producer profile and resume tests to seed a halt file and assert the exact HaltReason received by the runner, including a differing probe fan and a refreshed reason on a later call. Exercise an unreadable halt file before spawning. Extend the recipe profile table to assert that the plan row receives halt and amendment inputs and another segment does not.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/runplan/postbatch_test.go:58
    summary: The dropped-Create test never verifies that an undropped target still blocks.
    fix: Add a mixed-target row to TestPostBatchChecks_DroppedCreateSkipsItsDoneCheck that drops one Create and leaves another unresolved. Assert the exact remaining blocking target and the warning for only the dropped target, preserving the caller's original refs and declarations.
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: handler
    location: internal/websterengine/template_test.go:911
    summary: The no-parent-decisions test loses its assertion that the rendered value is none.
    fix: Restore an explicit assertion in the existing no-parent-decision row that the parent_decisions section renders none. Use the expected section text rather than comparing against a synthetic decision whose numbering now changes its output. Keep the placeholder and numbering checks.
---

Reviewed the committed range from `57015a52cd1f436ce96f5137889045e9a2ce5d7d` to HEAD.
The parent's “Scope after the corrupted board body” entry supplies the scope after the corrupted Goal.
All four lens reports are saved beside this report.

### [BLOCKING] Dropped declarations still fail binding

**Location:** internal/runplan/postbatch.go:286

**Issue:** A card declares `plan:sub#One` and `plan:sub#Two`, and the decision drops the second target.
With only `sub#One` created, the filtered target groups pass `DoneChecks`.
The binding call at `postbatch.go:368` still receives the original `in.Cards`.
`OwnedHandles` reads `Card.Declarations`, independently of target groups, in `internal/planparser/handle.go:85`.
`BindHandles` then emits `bind-count-mismatch` for the absent second handle in `internal/planglyph/handle.go:492`.
The post-batch pass returns `ErrCardNotDone`, so the parent's dropped Create still blocks both terminal paths.
The existing partial-binding fixture in `internal/planglyph/handle_test.go:696` establishes this concrete case.

### [BLOCKING] Amendment matching exceeds the named cards

**Location:** internal/burlerengine/roundcheck.go:343

**Issue:** An entry naming `14-card-slug-extra` also admits `_lym/plan/15-card-slug.md` through `card-slug`.
It also admits `internal/card-slug.go`, because the matcher discards the directory and extension.
Conversely, the explicit identity `1-a.md` cannot pass the four-character minimum.
Both new findings and moved carried findings consume this matcher.
The amended scope permits exemptions for the cards the decision names, not files sharing substrings of their names.

### [BLOCKING] Retained parser coverage needs its marker

**Location:** internal/discussionparser/append_test.go:412

**Issue:** The supplied current redundancy report marks `TestParseAddedDecisions` as contained.
Its covering test, `TestCheckAddedDecisionsKept`, asserts preservation findings rather than exact returned models.
The candidate separately asserts author, date, title and record order, so deleting it would lose those checks.
The rubric and `PATTERN-test-speed` require a `testtiming:keep` explanation for this retained candidate.
There is none above its declaration.

### [MEDIUM] Pending amendments vanish from prompt guidance

**Location:** internal/burlerengine/prompt.go:486

**Issue:** `BuildRoundContext` deliberately retains all pending amendment texts, including previously snapshotted entries.
`priorFindingsBlock` renders and marks only `AddedTexts`, and returns early when that list is empty.
The next round can therefore retain an amendment exemption without presenting its marked entry or instructions.
If an unrelated decision was added, the text refers to marked amending entries while omitting the pending amendment.
The full decision record remains available, which limits the severity but does not repair this contradictory guidance.

### [MEDIUM] Consolidation conflicts with halt classification

**Location:** internal/burlerengine/roundcheck.go:306

**Issue:** The new exemption requires `origin: webster` and a nonempty halt reason.
`contracts/stencils/burler/burler-cluster-fan.md:32` tells the handler to tag every retained finding with lens or handler origin.
`contracts/stencils/burler/burler-assist-consolidation.md:11` similarly requires the assistant origin on retained findings.
Following either instruction can strip the halt classification and make an outside-changes halt finding fail the round check.
The change must reconcile semantic halt classification with these existing provenance instructions.

### [MEDIUM] Halt delivery lacks a behavioral check

**Location:** internal/shedadapters/burlerreview.go:379

**Issue:** The reader and prompt renderer have separate tests, but the producer tests do not seed or assert `HaltReason`.
The supplied mutants removing the assignments at lines 379 and 383 survived.
Existing profile and resume tables check other forwarded fields and cannot detect this broken connection.
Plan-only recipe selection and failure before spawn on an unreadable halt file likewise lack direct assertions.
The write/removal mutants in `webster.go` affect logging branches and are not evidence of this separate delivery gap.

### [MEDIUM] Selective dropping is not tested

**Location:** internal/runplan/postbatch_test.go:58

**Issue:** The fixture declares two Create refs and drops both.
It compares that result with a call having no decision record.
An implementation that suppresses every Create whenever any drop exists would satisfy these assertions.
The required boundary is selective: an undropped unresolved target must still block.

### [MEDIUM] The none-rendering assertion was weakened

**Location:** internal/websterengine/template_test.go:911

**Issue:** The no-parent-decisions row previously compared its output with the rendering of the literal `none` value.
The diff removes that equality and checks only that no placeholder or `Decision 1:` remains.
An empty rendering now passes even though the row and renderer contract require `none`.
Numbering makes the old synthetic-decision comparison unsuitable, but an independent expected-section assertion is still needed.
No commit in the review range carries the rubric's required `Loosened-Check:` trailer for the removed check.

## Rejected

- The proposed separate BLOCKING finding for empty/newline CLI tests overstates the refusal rule: the backtick row reaches the same malformed-target refusal and asserts its way forward, while writer tests cover empty and newline rejection.
- Duplicate dropped-handle and pending-amendment findings from code-affected-set are consolidated into F1 and F4.
- A broader demand that every verify-gate branch name the report exceeds the amended scope's drift-halt requirement; the run-level drift branch names it and its acceptance command matches the CLI.
- Unknown drop targets currently match no Create and exempt nothing; the scope does not require a new unknown-target refusal.
- No separate defect follows from the logging-only survivors in Webster halt write/removal handling.
- The Weaver, envkit and CLI-validation edits are forced propagation of scoped changes; automatic rebaseline after a fix was not added.
- An arbitrary finding falsely claiming webster origin is a reviewer-contract violation, not an independently established parser defect.

## Validation

The requested tier-0 gate could not start.
The command was `timeout 3600s lym gate test ./internal/runplan ./internal/discussionparser ./internal/burlerengine --tier 0`.
The hub's gate-holder and process-registration records were read-only under this session's filesystem permissions.
No tests executed; findings above are grounded in source traces and the supplied current reports.
`lym board get webster-parent-calls-flow` returned the same invalid-JSON error recorded in the Goal.
The focus directive expressly identifies the parent's added entry as the replacement measuring stick.

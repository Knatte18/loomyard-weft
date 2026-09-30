---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/09-stage-wiring.md:406
    summary: The D7 baseline is taken from batch 1's StartSHA, but BeginBatch overwrites StartSHA with the current head on a re-begin, so after a re-begun batch 1 the baseline sits inside batch 1 and a regression can be excused as pre-existing; no card keeps the recorded start.
  - id: F2
    severity: LOW
    location: _lyx/plan/09-stage-wiring.md:424
    summary: Card 9 names only the RunResult.Warnings and runIntegrationStage doc comments for update, leaving RunResult.Outcome/StuckReason (taken verbatim from outcome.yaml) and Run's call-site comment about the loud done-over-a-failed-suite error stale after the demotion.
  - id: F3
    severity: LOW
    location: _lyx/plan/09-stage-wiring.md:435
    summary: The dirty-tree restore case does not say what dirt the baseline verify leaves, while RestoreBranch is a plain git checkout that refuses conflicting local changes, so the test can pin a guarantee the D7 surface does not give.
  - id: F4
    severity: NIT
    location: _lyx/plan/09-stage-wiring.md:417
    summary: The stage-return sentence lists the no-bisector notice unconditionally while Phase 2 emits it only on a regression, and the nil-bisector flaky test does not pin that the notice is absent.
  - id: F5
    severity: NIT
    location: _lyx/plan/10-stencil-verdict-handoff.md:451
    summary: Card 10's Intent cites only D2, leaving the Describe stencil change without the D11 grounding that licenses it.
---

### [BLOCKING] Baseline StartSHA does not survive a re-begin of batch 1

**Location:** `_lyx/plan/09-stage-wiring.md:406` (Phase 1, "the baseline is the `StartSHA` of the first batch in execution order").

**Issue:** D7 fixes the baseline as "the HEAD webster recorded before the first batch in execution order began", and rules out any point inside the batch.
`BeginBatch` writes `StartSHA: head` unconditionally when it records a batch (`internal/websterengine/beginbatch.go:396-398`), including on a re-begin.
A re-begin of a batch whose earlier fork already landed its work is a live path: commit `4ab6f6dd6` ("begin-batch re-begin skips a batch already begun") exists for exactly that case, and `beginbatch.go:258-260` documents it.
After such a re-begin of batch 1, `batches[0]`'s `StartSHA` names a commit inside batch 1's own work.
A test the first fork broke is then red at that "baseline" too, so triage classifies it pre-existing and the run ends `done` over a real regression — the one direction D7's rationale forbids ("triage can only remove halts it can prove unwarranted").
`BatchState.StartSHA`'s own doc comment (`state.go:159-164`) already promises the value "never names a point partway through the batch's own committed work", and `recover-batch` honours that by inheriting the prior value (`recoverbatch.go:200-209`); only the begin-batch re-begin breaks it.
No card touches `beginbatch.go`, so the plan as written does not deliver D7's baseline.

**Fix:** Add a card that makes `BeginBatch` keep a prior record's non-empty `StartSHA` on a re-begin, mirroring `recover-batch`, with an `integration`-tagged test in `beginbatch_test.go` pinning that a re-begin over a record carrying a `StartSHA` keeps it and a first begin records the head.
Card 9's Phase 1 then states that the baseline's provenance rests on that card.

### [LOW] Card 9 leaves stale comments the demotion falsifies

**Location:** `_lyx/plan/09-stage-wiring.md:424`.

**Issue:** Card 9 lists two comment updates: `RunResult.Warnings` and `runIntegrationStage`'s doc comment.
The demotion also falsifies `RunResult.Outcome`'s and `RunResult.StuckReason`'s doc comments, which say both are taken verbatim from the parsed `outcome.yaml` (`runlevel.go:180-184`), and the comment at `Run`'s `runIntegrationStage` call site, which explains the error path in terms of "its loud done-over-a-failed-suite error" (`runlevel.go:700-705`) that card 9 removes.

**Fix:** Name those comments in card 9's update sentence: `Outcome` and `StuckReason` are verbatim except when the integration stage demotes a `done`, and the call-site comment describes the remaining error paths.

### [LOW] Dirty-tree restore case does not pin a dirt shape the restore surface can undo

**Location:** `_lyx/plan/09-stage-wiring.md:435`.

**Issue:** D7 requires the restore to use "the same `FabricBisector` surface and restore-failure handling bisect uses".
`RestoreBranch` is a plain `git checkout <ref>` (`internal/gitrepo/gitrepo.go:268-274`), which refuses when an uncommitted change to a tracked file conflicts with the branch.
The card's case "the worktree is back on its branch ... including when the baseline verify leaves the tree dirty" names no dirt shape, so an implementer can write a test whose dirt conflicts and then find that the only D7-compliant outcome is a surfaced restore error, not a restored branch.

**Fix:** State the dirt the test leaves (an untracked file and a change to a tracked file identical between baseline and head), and state that a conflicting change surfaces as a restore error per the Constraints, which card 5's restore-failure case already pins.

### [NIT] No-bisector notice is listed unconditionally in the stage return

**Location:** `_lyx/plan/09-stage-wiring.md:417`.

**Issue:** Phase 2 keeps today's "could not be localized" warning only on a regression, matching D8.
The return sentence then lists "the no-bisector notice" alongside triage's warnings with no condition, and the nil-bisector flaky test in `runlevel_test.go` asserts only the verdict.
An implementer following the return sentence can emit "could not be localized" over a flaky run where nothing needed localizing.

**Fix:** Qualify the return sentence (the no-bisector notice only on a regression), and have the nil-bisector flaky case assert its only warning is the flaky one.

### [NIT] Card 10's Intent omits the D11 grounding for the Describe change

**Location:** `_lyx/plan/10-stencil-verdict-handoff.md:451`.

**Issue:** The Intent cites D2 only, but the Describe edit is licensed by D11, not D2 (see the focus settlement below).

**Fix:** Cite D11 in the Intent and state why the Describe rule is needed.

## Focus directive settlements

**Card 10's Describe stencil edit — licensed, not a defect.**
D11 calls `summary.md` "the PR text" and its rationale says `summary.md` "is the channel that survives to the operator and the PR".
The Describe stencil reads the run record "only as a source of manual-check items" and carries only an `## Integration suite failed` section into the description (`contracts/stencils/landing/landing-template-describe.md:20-21`);
it also forbids describing verify gates.
`webster-spec.md:44` confirms the failed section reaches the PR only because Describe carries it.
Without card 10's rule, the triage section never reaches the PR, so D11's stated outcome requires the edit.
The Intent's missing citation is F5.

**Card 11's `manifest/designs/` claim — holds.**
No doc under `manifest/designs/` mentions the integration stage, its fork, bisect, or `runIntegrationStage`;
the only webster doc there, `webster-parallel-execution.md`, mentions verify once, about serializing commit-and-verify for parallel forks.
Card 11 also edits `contracts/specs/webster-spec.md`, which line 44 ties to the integration-failure section; keeping it true is documentation follow-through, not new scope.

**Card 9's baseline provenance — defect, F1.**
`batches[0]` is the first batch in execution order, and `recover-batch` and `record-batch` preserve `StartSHA`, but a begin-batch re-begin overwrites it.
The absent-record and empty-`StartSHA` routing is correct: card 9 passes an empty baseline, and card 5's step 4 treats an empty `baselineSHA` like a nil repo (regression plus the baseline-unavailable warning), while card 9 still localizes through bisect whenever a bisector exists.

**Warning contract across cards 5, 8, and 9 — composes correctly.**
Nil bisector, red rerun: card 5's step 4 emits "baseline comparison unavailable" once, and card 9's Phase 2 emits "could not be localized" once.
Bisector present, empty baseline, red rerun: card 5 emits "baseline unavailable" once; card 9 localizes through `LocalizeIntegrationFailure` and emits no not-localized warning.
Card 8's `triageWarnings` covers only flaky and pre-existing, so it adds neither D8 warning a second time.
The only wording gap is F4.

**Granularity of cards 3, 7, and 8 — not defects.**
Card 3: the log-path helpers are one-line path builders with no behavior to test on their own; they belong with the capture seam that writes those logs.
Card 7: both appenders write `summary.md` sections through one helper the card factors out, so the two form one reviewable change, and neither half owns the shared helper alone.
Card 8: three small pure helpers with one consumer card and one shared test file, all recording a triage verdict; splitting them would yield cards too small to review on their own.

**Acceptance-criteria coverage — every criterion is exercised.**
"A Master done with no integration report still fails loud" is pinned by the existing `TestIntegrationStage_MissingReport_DoneOutcomeFailsLoud` (`integration_test.go:365`), which runs through `Run` and which card 9 leaves in place while keeping that error.
Baseline restore is split soundly: card 5 covers a baseline run error and a restore failure with fakes; card 9 covers the dirty tree for real, with the dirt-shape gap recorded as F3.
The remaining criteria map onto card 2 (parser cases), card 4 and card 10 (rendered prompts), card 5 (classification, missing log, rerun-only failure, opaque cases, nil repo), card 6 (bisect past a pre-existing failure), card 9 (stage outcomes, friction note, two-commit batch 1, Master `stuck`), and card 11 (docs).

**Verify coverage.**
The cards target `internal/websterengine`, `contracts/stencils`, `contracts/specs`, and, conditionally, `internal/landingshed`'s tests;
the `## verify:` section runs all of them untagged and `integration`-tagged.

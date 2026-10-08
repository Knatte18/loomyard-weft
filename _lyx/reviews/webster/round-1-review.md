---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/websterengine/state.go:130
    summary: PlanOverviewFrameHash comment still says a state without the field refuses any overview change, which card 5 made false; doc.go:202 also still reads as if the state must carry the hash
  - id: F2
    severity: LOW
    class: consistency
    location: tools/tokencount/count_test.go:153
    summary: readMerriamStart branches with no coverage - a launch line inside a list text item and a prompt Read with no assistant message after its result
---

Review range: `status.json` gives `product.parent` = `main`, and no `_lyx/loom/rework/` rounds exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`, seven commits, one per card.
`go build ./...`, `go vet` under the untagged, `integration`, `tmux` and `llm` tags, `go test ./...` and `go test -tags integration ./...` are all green.

### [LOW] Stale comments left behind by card 5

**Location:** `internal/websterengine/state.go:130`, `internal/websterengine/doc.go:202`
**Issue:** The `PlanOverviewFrameHash` field comment ends with "A state written before this field existed leaves it empty, and refuses any overview change."
Card 5 made that false: such a state now takes the recorded frame from the stored baseline copy through `recordedOverviewFrame` and accepts a Card-Index-only change.
The same card's commit rewrote the neighbouring `doc.go` paragraph and the `Rebaseline` doc comment but left this comment, which a reader of `State` meets first.
`doc.go:202` ("Rebaseline accepts it if the state carries that hash and the file's frame hash still equals it") is amended by the sentence two lines below it, but it reads as the whole rule until then.
**Fix:** Reword the `state.go` sentence to say a state without the field takes the frame from the stored baseline copy of the overview until the first restamp, and refuses any overview change only when that copy is absent or has no parseable Card Index.
Reword `doc.go:202` to "the run's recorded overview frame" so it agrees with the paragraph's later sentences.

### [LOW] Uncovered branches of `readMerriamStart`

**Location:** `tools/tokencount/count_test.go:153` (the `TestCountRun` Merriam fixture), `tools/tokencount/count.go` `readMerriamStart` and `userTexts`
**Issue:** The new `TestCountRun` fixture writes every launch line as string content.
`userTexts` has a list-of-items branch, and the plan names "string content or a text item" as the launch line's forms, but the only list-content user message in the fixture ("load the skills") does not hold the launch line, so a launch line inside a text item is never matched by any test.
The `NoStart` reason "no assistant message after the prompt Read's result" is also reached by no test; only "no Read of the prompt file" is.
Both are behaviors the diff adds at the `CountRun` surface (`PATTERN-test-economy`: a behavior the diff adds with no coverage).
**Fix:** Add rows to the existing Merriam fixture in `TestCountRun`, not a new test: one session whose launch line is a text item of list content and which measures normally, and one whose transcript ends at the prompt Read's result.

### Per-focus verdicts

These are answers to the round's focus questions, not findings.

- **Cards 2 and 3, `resolveReportHead` at every consume site.** The consume sites of a report's `head_sha` are record-batch's main path (`recordbatch.go`, right after `ParseReport`, before `refuseMidMerge`, `reconcileReportHead`, the digest and `CardSHAs`), its correctness-failure path, `awaitTerminal` (after the mid-merge refusal, before `reconcileReportHead`; the only digest source for `PersistRecoveryTerminal`) and `reportHeadSHA` (before `SHAExists`).
  A grep of every `HeadSHA` read in `internal/` finds no other consume site of a report's value.
  An unresolvable or ambiguous value is refused with `ErrHeadSHAUnresolved` and the `git rev-parse HEAD` way forward, a git failure does not wrap the sentinel, a non-hex or 40-plus-digit value passes through to its old refusal, and the spec row, `doc.go` text and tests land in the card's commits.
  Not a defect.
- **Card 4, bracket transcripts.** `BracketTranscripts` is a separate `BatchState` field; begin-batch and recover-batch build a fresh record without it, and the `TestBeginBatch_Record` row pins the empty list while `ForkTranscripts` carries over.
  The engine is handed the seen set minus the bracket's transcripts, `SettleRetry` still takes the full set, so `newReports` and the counted own transcripts are disjoint and the union has no duplicates.
  `forkKey` is untouched and keyed by transcript, class and ordinal; the "later part reported, earlier part not reported again" rows pin identity across a re-read.
  Not a defect.
  The plan's test row about another batch's record-batch and `auditTerminalFork` not claiming a no-report call's transcript is asserted only through the seen-set and list contents, not by running those callers; that attribution was already how the no-report path advanced, so I did not raise it.
- **Card 5.** `recordedOverviewFrame` takes the told `WebsterDir`, derives no path, falls back only to a baseline copy of the recorded overview hash and yields `""` for an absent copy or an unparseable Card Index.
  The signatures of the five re-signed helpers match the card's `->` lines and every caller passes its told `WebsterDir`.
  A Card-Index-only change passes and any other overview change is refused with the follow-up-card way forward; both are pinned in `TestRebaseline_EditedPlan` and `TestPlanEditErrors`.
  Not a defect apart from F1.
- **Cards 6 and 7.** The start measurement lands at the first assistant message with usage after the result of the Read of the launch line's prompt path.
  `readMerriamStart`, `merriamTexts`, `readRunStartBase` and `addFixed` share the one `merriamTexts`, which matches `MerriamBase`'s texts (`CLAUDE.md`, the raw Master template, the rendered orchestrator directive) apart from the plan overview and the untracked `CLAUDE.local.md`, whose exclusion is documented.
  Not a defect apart from F2.
- **Card 1.** `Directive` calls `FillDirective` after its inactive-overview return and its stencil read, so it still reads no stencil for an inactive PATTERN and its fill-error text is unchanged.
  `FillDirective` renders the same bytes for the same inputs, and `pattern`'s imports are unchanged.
  Not a defect.
- **Per-card mechanical checks.** None of the seven cards carries a `Rename` or `Delete` group; every group is `Create`, `Edit` or `Uses`.
  Every symbol the cards create exists and the five `->` re-signs match their stated signatures.
- **Test economy.** `TestRecordBatch_ResumedForkAcrossNoReportCall` and `TestRecordBatch_UnresolvableHeadSHARefused` are the only new test functions, `test-budget.yaml` rises by exactly that, and neither repeats an existing test's coverage.
  The other tests extend existing tables as the cards direct.
- **Docs and refusal rows.** Each card's `doc.go` text and `refusal-spec.md` row (`report head_sha unresolved`, reworded `report unattributable` and `rebaseline overview changed`) land in that card's own commit; the tokencount package comment lands with card 7.

# Card 7 — Record-batch fills the digest's deviation list from the scope guard alone

**Edit:**
- `internal/websterengine#postBatchChecks` -> `func postBatchChecks(in postBatchInputs) (deviations, warnings []string, err error)`
- `internal/websterengine#RecordBatch`
- `internal/websterengine#PersistRecoveryTerminal`
- `internal/websterengine#distill`
- `internal/websterengine#Report`
- `internal/websterengine#Digest`
- `internal/websterengine#digestSummaryLine`
- `internal/websterengine#TestClassify`
- `internal/websterengine#TestReport_RoundTrip`
- `internal/websterengine_test#TestRecordBatch_PostBatchChecks`
- `internal/websterengine_test#TestRecordBatch_ScopeGuardDegradesOnDeltaUnavailable`
- `internal/websterengine_test#TestRecordBatch_ParentMovedHead`
- `internal/websterengine_test#TestBeginBatch_PromptFile`
- `internal/websterengine_test#recoverFixture`
- `internal/websterengine_test#newRecoverFixtureOver`
- `internal/websterengine_test#TestRecoverBatch_SecondCall`
- `internal/websterengine/doc.go`

**Intent:**
D4's webster half: the batch digest's existing `deviations` field holds the scope guard's deduplicated, sorted list of worktree-relative files, never a fork's guess, and the scope findings reach a reader once, in the digest.

`postBatchChecks` returns the deviations beside its warnings.
Where it ran `delta.ScopeGuard(in.Cards)` and appended each finding's `Error()` to the warnings, it now collects each finding's `File` (card 6), skipping empty ones, deduplicated and sorted, as the deviation list; none of them joins the warnings.
A batch whose changes stay inside its targets gets an empty list.
When `deltaErr` is set it returns no list and keeps the existing "glyph scope guard could not run" warning.
Every other return passes a nil list.

Its two callers store the list in the digest they persist:
`RecordBatch` sets `digest.Deviations` after `distill`, and `PersistRecoveryTerminal` sets its told digest's `Deviations` before `bs.Digest = digest`, so a done recovery's digest carries the same guard list.
On their failure paths the list is dropped, as the failed digest records reasons instead.

`distill` no longer copies `Report.Deviations`; its doc says `Deviations` is filled by record-batch from the guard.
`Report.Deviations` stays only so `ParseReport`'s known-fields decoding still accepts the key from an older fork: its tag becomes `yaml:"deviations,omitempty"` and its doc says the key is accepted and never read; `ParseReport` is otherwise unchanged, so a report with or without the key parses.
`Digest.Deviations`' comment names the guard as its source and stays informational: it never changes `Status`.
On disk the field keeps `omitempty`, so a digest with no deviation and one whose guard could not run both carry no list, and `PATTERN-tolerant-records` reads a missing field as an empty list; the guard-could-not-run warning tells the two apart.

`digestSummaryLine`, the fork prompt's predecessor digest line, renders the list as ` scope-guard deviations=a,b` instead of ` deviations=a,b`, so the line names the guard as its source.

`internal/websterengine/doc.go`'s `# Fork-return contract` section says the fork returns `status` and `head_sha` only, an older fork's `deviations` key is accepted and ignored, and the digest's deviation list is the guard's file list over the batch delta, informational as before; the `# every terminal batch runs the same mechanical pass` paragraph says the guard's result lands in the digest.

Tests:
- `TestClassify`'s two rows "a report's deviations carry through …" become "a report's deviations are ignored and never change an OK status" and the FAILED twin, and its loop's comparison asserts `got.Deviations` is empty instead of equal to the report's.
- `TestRecordBatch_PostBatchChecks`'s "scope guard findings land in the warnings" subtest becomes "scope guard findings land in the digest's deviations, not the warnings": it asserts `result.Digest.Deviations` is exactly `[]string{"internal/foo/impl.go"}` and that no warning contains `scope-outside-plan`.
  Its "binds a handle from the delta" subtest, whose change stays inside its target's package, asserts `result.Digest.Deviations` is empty.
- `TestRecordBatch_ScopeGuardDegradesOnDeltaUnavailable` also asserts `result.Digest.Deviations` is empty beside the could-not-run warning.
- `TestRecordBatch_ParentMovedHead`'s row "the parent merge's symbols are not the batch's own" also asserts `result.Digest.Deviations` is empty, since its warning loop alone no longer proves the guard skipped the parent side.
- `TestBeginBatch_PromptFile` gains the row "the predecessor digest names the guard's deviations": the predecessor's digest carries `Deviations: []string{"a.go", "docs/b.md"}` and the prompt wants `scope-guard deviations=a.go,docs/b.md`.
- `recoverFixture` gains `Index *fakeIndex`, kept by `newRecoverFixtureOver` from `indexOver` (nil over a real repository), and `TestRecoverBatch_SecondCall` gains the case "a landed report records the guard's deviations in the persisted digest": its `config` sets `fx.Index.delta` to three `Created` symbols, `other#C` and `other#B` in `other/b.go` and `other#A` in `other/a.go`, in that order, and its check asserts `fx.Deps.State.Batches[1].Digest.Deviations` is exactly `[]string{"other/a.go", "other/b.go"}`, deduplicated and sorted, and that no warning contains `scope-outside-plan`.
- `TestReport_RoundTrip`'s row "FAILED with populated deviations" keeps its report content, so a report carrying the key still parses, and the loop's `Deviations` equality is dropped, since the field is never read; its row "OK with empty deviations" now writes no `deviations` key through `WriteReport`'s `omitempty`, so it is the row proving a report without the key parses.

**ImpactSummary:** One private pass re-signed with its two callers in `websterengine`, the digest's fill site moved from the fork report to the guard, one prompt line relabelled, and their tests; the digest's JSON keys and the record-batch envelope are unchanged.

**Commit:** 7: Fill the batch digest's deviations from the scope guard

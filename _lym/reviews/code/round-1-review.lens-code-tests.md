```yaml
lens: code-tests
outcome: reported
```

Restating the task: for each behaviour the diff adds or changes, name the test row that would fail if it broke, and file a finding for any behaviour with none.

## Per behaviour: the test that pins it

- **`ValidateDescription`: one forbidden-term finding per line, first match only, word-bounded, nil patterns check nothing.** Pinned by `TestValidateDescription_ForbiddenTerms` (`internal/summaryparser/description_test.go:102-150`).
- **`NewDescriptionGate` passes its patterns through.** Pinned by `TestNewDescriptionGate`, row "forbidden term" (`internal/landingshed/describe_test.go:367`).
- **`LoadConfig` refuses a malformed `describe_review`, a review timeout below 1 and a pattern that does not compile.** Pinned by `TestLoadConfig_InvalidKeys` rows (`internal/landingshed/config_test.go:225-227`).
- **The template's defaults and patterns.** Pinned by `TestLoadConfig_TemplateDefaults` (`config_test.go:197-209`).
- **`LoadConfig` skips the model-spec parse when `describe_review` is empty.** None. No `LoadConfig` row loads an empty `describe_review`. `TestDescribeTable_ReviewerSeat` builds `Config` as a literal, so it never reaches this.
- **The configured patterns reach the callers.** This covers `loomcli` validate-description through `DescriptionTerms`, including its "read landing.describe_forbidden_terms" error path; shedrecipe's "description" gate forwarding `env.DescriptionTerms`, including its error path; `wiring.go`'s full wire; and beamcli's recorders, `describeGate` and the Findings closure. None:
  - every test passes `nil`: `dobbydeps_test.go:108,117`, `beamdescribe_test.go:191,288`, `parity_test.go:801`;
  - `envkit.FullEnv` returns `nil, nil` (`envkit.go:427`);
  - a grep of the `loomcli`, `shedrecipe` and `beamcli` tests found nothing that asserts the terms are forwarded.
- **`DescribeTable` adds the reviewer seat (model, skip reason, marker, outputs, timeout), puts the report in the lead's inputs and builds a one-entry gate.** Pinned by `TestDescribeTable_ReviewerSeat` (`describereview_test.go:579-634`).
- **`DescribeTable` refuses an empty `ReviewDir`.** Not checked; I spent no call on it.
- **The `describeReviewGateEntry` verdicts.** These are: pending on a missing marker, pass on a review that did not finish, pass on approval, findings once and then pass, pass on a done review with no report, and pass at Final. Pinned by `TestDescribeReviewGateEntry` (`describereview_test.go:636-685`). The `ReadEnding` error path and a report read error other than not-exist have no row.
- **`describeReportGate` accepts the two verdict lines and refuses any other.** Pinned by `TestDescribeReportGate` (`describereview_test.go:687-710`). A missing report has no row.
- **`describeEntry` puts the row's gates ahead of the table's own entries and keeps those entries.** None. `TestDescribeEntry_HappyPath` (`internal/shedrecipe/entries_describe_test.go:83-86`) asserts `len(gate) == 1` named "description". Its table source carries no gate of its own, so the old line `table.Gate = gate`, or the reverse order, passes the test unchanged.
- **End to end: the writer's turn end is held by the pending entry and the findings are delivered once through shuttle.** None. The change description itself says so (`summary.md:67`). The shuttle machinery for pending, PassOnCap and WriterOnly comes from the base.
- **`MultiLLMProducer`: a chair that started and timed out is Stuck with an empty path and a detailed reason (seat limit over table limit, unknown or none recorded); a chair that never started stays an error.** Pinned by the `TestMultiLLMProducer_MapsTheLeadResult` rows (`multillm_test.go:171-200`).
- **The writer seat's timeout detail.** Pinned by the `TestMultiLLMProducer_MapsAWriterThatEndedTheStep` rows (`multillm_test.go:237-253`).
- **`pullrequest.Client.Status` returns the title and body.** Pinned by `TestClient_Calls` row 1 (`client_test.go:76`).
- **PR-Review's `syncText` compares against GitHub's text, ignoring line endings and trailing whitespace.** Pinned by the `TestPRReview_Wait` rows "ABodyEditedOnGitHubIsSetBack…" and "…OnlyInLineEndings…" (`prreview_wait_test.go:695-709`), and by the renamed `TestPRReview_WaitThatStartsWithNoTextSet…` (`:888-910`).
- **A description edited after the PR exists is followed at the next poll.** Pinned by the existing row "AnEditedDescriptionSetsTheTitleAndBodyAgain…" (`prreview_wait_test.go:686`).
- **`openPullRequest` stamps the entry status with the text it just set, so the first poll does not edit again.** Pinned by the re-entry rows `wantCalls {"Status 7", wantEditSeven, "Status 7"}` (`prreview_test.go:303,313,322`). A second Edit would appear there.
- **Finalize's `setRecordedText`.** Pinned by the `TestFinalize_PullRequest` rows (`finalize_pr_test.go:76-100`) for a body that differs, a title that differs, a line-ending difference that is not edited and a failed edit that halts; and by the beam rows with no edit (`:210-224`). A failed status read in `setRecordedText` and a transient edit failure have no row.

## Findings

1. **MEDIUM · design · `internal/shedrecipe/entries_describe.go:53`.** Nothing pins that the row's gates are put ahead of the table's own `Describe-Review` entry and that the entry is kept. Reverting to `table.Gate = gate` would drop the reviewer gate silently and every test would still pass.
   - Evidence: `entries_describe_test.go:83-86` asserts only the single "description" entry, and its fake table carries no gate.
   - Fix: give the test's table source a gate entry, then assert the order: `[description, &lt;table entry&gt;]`.
2. **MEDIUM · design · `internal/loomcli/wiring.go:423`, `internal/shedrecipe/entries_gate.go:289-297`, `internal/beamcli/deps.go:338-342,466` and `internal/beamcli/dobbydeps.go:74-79`.** No test shows the configured `landing.describe_forbidden_terms` reach any of the paths that check a description. Those paths are the Describe row's description gate, `lym loom validate-description` with its error exit, the beam describer's gate and findings, and the beam recorders. Every test passes `nil` or `nil, nil`, so dropping the forwarding anywhere goes undetected.
   - Evidence: `dobbydeps_test.go:108,117`, `beamdescribe_test.go:191,288`, `parity_test.go:801`, `envkit.go:427`; a grep found no assertion that the terms are forwarded.
   - Fix: add one row per seam with a non-nil pattern and a description that matches it.
3. **LOW · design · `internal/landingshed/config.go:221-225`.** `LoadConfig` skips the model-spec check for an empty `describe_review`, the documented way to turn the reviewer off, and no `LoadConfig` row covers it. Removing the guard would make every hub that sets the key empty fail at load.
   - Evidence: `config_test.go:205-227` has no row with `describe_review: ""`.
4. **LOW · design · `internal/landingshed/finalize_pr.go:787-789`.** `setRecordedText`'s failed status read, which halts or re-steps naming "read the pull request's status", has no row; neither does a transient edit failure.
   - Evidence: the `finalize_pr_test.go` rows at 76-100 cover only a failed edit that halts.
5. **LOW · design · `internal/landingshed/describereview.go:524-547`.** The error paths have no row: an end marker that cannot be read, a report read error other than not-exist, and `describeReportGate` reading a missing report.
   - Evidence: `describereview_test.go:636-710`.

## Tool calls

1. Grepped the `shedrecipe`, `loomrecipe`, `loomcli` and `beamcli` tests for `describeEntry`, `DescriptionTerms` and forbidden-term coverage: the manifest did not list those tests.
2. Grepped `prreview_test.go` and `finalize_pr_test.go` for the edit rows and the status-read failure rows: the manifest held those tests only as diff hunks.
3. Read `entries_describe_test.go:60-120`, the body of `TestDescribeEntry_HappyPath`, to see what its gate assertion checks: it is not in the manifest.
4. Read `prreview_test.go:300-330`, the re-entry rows, to confirm they pin the entry-status stamp: the manifest did not hold them.

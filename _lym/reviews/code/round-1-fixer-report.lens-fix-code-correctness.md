```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes for correctness (lens fix-code-correctness), including whether each added test would fail with its defect in place.

Findings:

- **LOW**: row `AmendmentGenerationRendersNoReviewInputNote` in `internal/shedadapters/bouncer_amendment_test.go` passes with the defect in place.
  - In that row the amendment run has no review input configured, so the generation record holds no input paths.
  - `ReviewInputNote` (`internal/shedadapters/amendment.go:161`) returns `noNote` on `!found || generation.Amendment || len(paths) == 0`.
  - Dropping the `generation.Amendment` clause still gives `noNote` here, because `len(paths) == 0` already holds.
  - F3 asked for the note to be none for an amendment record, so the row needs an amendment record that carries a review input path.
  - Example: write the generation record by hand with `Amendment: true` and a `review_input` path, as the `RecordWithOnlyReviewInputReadsAsThatOnePath` row does, then assert `ReviewInputNote` equals `noNote`.

Checked with no finding:

- **`writeSendBackNote`** (`internal/loomcli/prfindings.go`)
  - With n=1 and `pr-review-send-back-1.md` present, it returns nil and writes nothing.
  - With an empty `frictionDir`, it returns nil before any stat.
  - A stat error other than not-exist is returned, and the caller turns it into the "write the friction note" refusal.
  - The new row pre-creates the note and asserts the directory holds exactly that one file with the prior content.
  - Without the guard, `NotePath` would return `pr-review-send-back-1-2.md`, so the row would fail.
  - The only caller, `prFindingsVerb`, is unchanged.
- **New `TestPRFindingsVerb` rows**
  - A missing `Code-Bouncer` row gives "the run's recipe has no Code-Bouncer row", wrapped by "read whether the send-back edge can fire".
  - An entry gate that errors gives "read the Code-Bouncer row's entry gate: gate unreadable", with the same wrapper.
  - Both refuse before any write, so the default zero wants for the record, scratch and friction directory are correct.
- **`TestLoosenedTrailerLines`**
  - The CRLF middle-paragraph row fails if the `\r\n` replacement is removed. The last-paragraph cut would then not be found, and the trailer would be collected.
  - The unsorted row fails without `slices.Sort`.
  - The duplicate row and the middle-paragraph row fail without `slices.Compact` and without the last-paragraph cut.
- **`resolveRunID` row for `pr-findings`** fails without the exemption, since `runID` would become the file path.
- **Command-mode `TestDriftVerify` subtests**
  - The failing-compare subtest asserts the order of the two reasons and the stored file ruling.
  - The exit-0 subtest asserts `seats.Calls` stays at 1 after the head moves. It fails if the stored file ruling is not reused, because a second judge run would follow.
- **`TestPullRequestDeps_ShedStatus`**
  - The unparsable-file subtest errors as expected.
  - The no-run-directory subtest passes the gate. I accepted the premise in the code comment that a missing run directory fails the lock open with not-exist, and did not confirm it.
- **F7 deletion**: the package compiled and the gate passed, so no import was left unused.

Further tool calls: none.

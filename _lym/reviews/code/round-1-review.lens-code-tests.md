```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it. I made no tool calls and judged from the inherited context.

**Behaviour → pinning test**
1. A fanned seat gets the fanned stencil and `lens_report_path`, and a solo seat keeps the solo stencil: `TestEngine_Review_AssistantPromptFollowsItsFan` (`assist_test.go`), both rows. It checks through prompt text, not the stencil name.
2. Lens reports are named in `output_files` after the outputs: the seatengine sub-test `AFannedAssistantIsToldItsSubagentsAreItsOwnWorkAndItsReportsAreItsToWrite` (`prompt_test.go`), and the `Lens...Path(report,"style")` assertion in the burlerengine test above.
3. The reports stay off the spec's `OutputFiles`: both tests above assert it.
4. `Table.Validate` requires the matching directive: three new rows in `table_test.go`.
5. A fresh run removes stale subagent reports: `engine_test.go`, `AssistantRemovesStaleFilesBeforeTheChairStarts`.
6. `keepAssistReports` names `MissingLenses`:
   - `TestKeepAssistReports_MissingLenses` covers every lens returned, a solo seat, an absent file, a `no report` file, and a non-done ending.
   - It also pins fan order.
7. `lensreport.Returned`: `TestReturned` covers reported, no report, no block and an absent file.
8. `lensreport.Path`: `TestPath_...`.
9. `assist_ending` and `assist_lenses_missing` land in the usage record: `TestBurlerReviewProducer_Call_RecordsAssistOutcome`, four rows.
10. The fanned-directive and stencil-registry markers: the new row in `markers_test.go`, plus runtime fill in the prompt tests.
11. `concludeReview` passes `e.assistLensNames(opts.AssistFan)` into `keepAssistReports`: **none**.
12. An unknown fan gives a solo stencil with no subagent reports: **none direct**.
13. `ProbeReview` and `ResumeReview` with a fan build the fanned table: **none**.
14. A re-accepted or retried round overwrites a stale `AssistEnding` and `AssistLensesMissing`: **none**.

**Findings**

1. MEDIUM, design. The wiring in `review.go:242` is unpinned. `keepAssistReports` is tested only by calling it directly with `lensNames`. `TestEngine_Review_AssistantPromptFollowsItsFan` runs `Review` with a fan but never asserts `ReviewResult.Assists[].MissingLenses`. If `concludeReview` passed `nil` or the wrong fan, every test would still pass and the fanned-seat record would silently never list a missing lens. This is the behaviour the Idea's second bullet asks for. Fix: add an `Assists[0].MissingLenses` assertion to that test's fanned row, since the scripted shuttle writes no lens files and it should read `["style", ...]`.

2. MEDIUM, scope. Test economy: new assertions on a stencil's static prose, which `PATTERN-test-speed` ("No test pins a stencil's static prose") forbids.
   - The strings are "You work alone: no other session works with you.", "are part of your own work and are no other session", and the absent "you write your output and nothing else" / "Your output is the only file set you write".
   - They appear in `assist_test.go` (`wantInText`/`wantAbsent`) and `prompt_test.go`. The solo row also asserts the absent words "subagent" and "lens" in `assist_test.go`.
   - Allowed in a stencil test: marker positions and values Go fills, such as the `output_files` content and the lens paths. Keep those; drop the wording assertions.

3. LOW, scope. The two tests overlap. `prompt_test.go`'s `AFannedAssistantIs...` and `assist_test.go`'s `...FollowsItsFan` both assert the same prose strings, the same report-listing and the same "spec outputs are the report alone". `assist_test.go` covers the real composed stencil and `prompt_test.go` a test-dir copy. Once the prose assertions go, keep the `output_files` list and `OutputFiles` check in one place, ideally the seatengine test for the marker and the burlerengine test for the real stencil choice. Fold the duplicate `OutputFiles` assertion.

4. LOW, design. There is no test for an unknown or empty fan keeping `assistLensNames` nil and the seat solo. `TestEngine_Review_AssistantThatDoesNotStart` covers the unknown-fan row but does not assert the stencil or `SubagentReports`. If the unknown-fan guard broke, an unstarted seat would carry a fanned stencil with no reports. Add an assertion to that existing row rather than a new test.

5. LOW, design. There is no probe or resume row with a fan. `probeTable` calls `assistSeat`, which now builds the fanned table, so `Validate` in `ProbeReview` runs on the fanned stencil for the first time. `TestEngine_ProbeReview_AssistantSeat` is not shown to set `AssistFan`. Add a fan row to it, so a mismatch between probe and fresh table would surface.

6. LOW, design. The usage-record test starts from an empty record in every row. A retry or re-accept that leaves `AssistEnding` or `AssistLensesMissing` from an earlier attempt, when the new attempt has no assistant, is not exercised. `recordRoundAttempt` resets `Assist` and `AssistFan` but not the new keys, and `acceptedExit` only sets them when `Assists` is non-empty. This is a correctness gap for the code-correctness lens, and also an uncovered case. Add a row that pre-seeds a record.

**Tool calls:** none. The manifest already held every test diff and the code they exercise.

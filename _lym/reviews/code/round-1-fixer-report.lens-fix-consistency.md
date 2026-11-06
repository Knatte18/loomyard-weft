1. **LOW** — `internal/seatengine/prompt_test.go:329`, subtest `AFannedAssistantIsToldItsSubagentsAreItsOwnWorkAndItsReportsAreItsToWrite`: after the F1 and F6 fix it asserts only the `output_files` list, but its name still claims it checks what the assistant is told. A matching name would be `AFannedAssistantListsItsSubagentReportsAfterItsOutputs`.

Checked and consistent: spec agreement for F4 (`contracts/specs/twill-lens-report-spec.md` lines 19-35); no leftover `reportedLine`; F5 wording agrees with `fannedAssistantDirectiveBlock` and `doc.go` line 115; the `recordRoundAttempt` comment stays true.

Tool calls made, 2 of 6: grep for `reportedLine`, "every assistant", `outcome: reported` (files outside the changed ones); Read of `contracts/specs/twill-lens-report-spec.md` (not in context).

```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the `2c4864bb..HEAD` diff. I ran each case from the inherited context and made no tool calls.

**Function / case list**
- `assistLensNames`, fan `""`: returns nil, a solo seat. Correct.
- `assistLensNames`, a fan the lens library lacks (`ResolveFan` errors): returns nil, the seat stays solo, and `composeAssist` already set `SkipReason` so it never starts. Correct.
- `assistLensNames`, fan `"standard"`: names come back in fan order. Correct.
- `assistSeat`, fanned: `SubagentReports` holds one path per lens, `Stencil` becomes the fanned one, and `Values["lens_report_path"]` is set on the fresh map `reviewSeatValues` returns. `Outputs` stays the report alone, so the shuttle spec waits only on the report. Correct.
- `assistSeat`, solo or nil: stencil, values and `output_files` are unchanged, the same bytes as before. Correct.
- `assistSeat` on the probe/resume path (`probeTable`): it builds the same fanned seat from `opts`. Correct.
- `Seat.assistantDirective` / `Table.Validate`: reports with the solo stencil, no reports with the fanned stencil, and the matching pairs. The three table_test substring expectations match the new message text. For a non-assistant, `directive` is computed but not used. Correct.
- `seatValues`, `output_files`: outputs followed by subagent reports. `outputs` and `seatSpec.OutputFiles` exclude the reports. Correct.
- `clearAssistantFiles`: `slices.Concat` includes the reports. The paths carry the attempt token, so a retry never shares them. Correct.
- `keepAssistReports`, done with a report: copy, then `missingLensReports`. Correct.
- `keepAssistReports`, ending not done, or `lensNames` nil: `MissingLenses` stays nil and no warning is logged. Correct.
- `keepAssistReports`, unreadable marker: ending is `""`, nothing is computed. Correct.
- `missingLensReports`: a file that is absent, unreadable, or `no report` counts as missing, in fan order. Correct.
- `Returned`, canonical reported file, `no report` file, no block, absent file: true, false, false, false. Correct.
- `Returned`, a file with CRLF line endings, or trailing whitespace after `reported`: false. Wrong; see finding 2.
- `acceptedExit`, assistant present: `AssistEnding` and `AssistLensesMissing` are overwritten together, so a non-done ending clears the missing list. Correct.
- `acceptedExit`, `result.Assists` empty: the loop does not run and the fields keep their old values. Wrong; see finding 1.
- `concludeReview`, `lensNames`: taken from `opts.AssistFan`, which is the fan recorded at the spawn. Correct on the resume path too.

**Findings**

1. MEDIUM, class `design`: `assist_ending` and `assist_lenses_missing` can go stale in `round-<N>-usage.yaml`.
   - `recordRoundAttempt` (`internal/shedadapters/roundusage.go:86-93`) resets `Attempt`, `Assist` and `AssistFan` for each new attempt, but not the two new keys.
   - `acceptedExit` (`internal/shedadapters/burlerreview.go:425-428`) writes them only when `result.Assists` is non-empty.
   - A round accepted once with an assistant and reviewed again with none (for example, the assistant turned off in config) is left with `assist: nil`, `assist_fan: ""`, but the old `assist_ending: done` and a stale missing-lens list.
   - That record misreports which assistant ran. The record exists so a silent solo seat cannot pass as a fanned one.
   - The test (`burlerreview_assist_test.go`) uses a fresh `runDir` per row, so it does not see this.
   - Fix: reset both fields in `recordRoundAttempt`, or assign them unconditionally in `acceptedExit`.

2. LOW, class `design`: `Returned` (`internal/lensreport/lensreport.go:22,50-53`) does not read the `outcome` key, though its doc and the spec say it does.
   - It does `bytes.Contains(content, "\noutcome: reported\n")` over the whole file.
   - A valid report with CRLF line endings, or trailing whitespace after `reported`, reads as not returned, so the lens is logged and recorded as missing although it ran.
   - A `no report` file with that line in its body would read as returned. The spec says a `no report` file ends at the block, so this needs an agent that breaks the format.
   - Fix: read only the opening fenced block, and compare the trimmed value of its `outcome` line.

I found no other defect in the changed paths.

Tool calls: none. The diff and every definition the lens needed were already in the inherited context.

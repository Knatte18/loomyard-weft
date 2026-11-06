---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/burlerengine/assist_test.go:440
    summary: Missing-lens coverage bypasses the public review path
    fix: Move the missing-lens scenarios into TestEngine_Review_AssistedRound, configuring AssistFan and the lens library and having the fake assistant write the scenario's lens files before it ends. Assert ReviewResult.Assists[0].MissingLenses from Engine.Review, including the unfanned and non-done cases, and remove TestKeepAssistReports_MissingLenses.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/lensreport/lensreport.go:54
    summary: Returned matches literal bytes anywhere instead of reading the opening outcome
    fix: Read the outcome from the opening fenced YAML block only, respecting scalar spelling and line endings while preserving the package's standard-library-only import boundary. Return false for an absent or malformed opening block and for unreadable files. Extend TestReturned with a quoted reported value, CRLF input, and a body-only occurrence of the reported line.
    origin: handler
---

### [BLOCKING] Missing-lens coverage bypasses the public review path

**Location:** internal/burlerengine/assist_test.go:440

**Issue:** `PATTERN-test-speed` requires behavior tests at the module's public surface and explicitly requires an unexported helper to be tested through that surface.
The new test supplies the expected lens names directly to `keepAssistReports` at line 474.
It therefore never exercises the new connection from `RunOpts.AssistFan` through `assistLensNames` to the returned outcome in `concludeReview` at `internal/burlerengine/review.go:242`.
Replacing that call's lens-name argument with nil would leave the added assertions passing while every completed fanned assistant reported no missing lenses.
`TestEngine_Review_AssistantPromptFollowsItsFan` discards the returned result, and `TestBurlerReviewProducer_Call_RecordsAssistOutcome` supplies preconstructed outcomes.
The existing `TestEngine_Review_AssistedRound` is the public-surface test to extend; this is a coverage-boundary finding, not a claim that another test already covers missing lenses.
The BLOCKING severity follows the directive's explicit mapping for repository-constraint violations.

### [MEDIUM] Outcome detection depends on formatting and unrelated report text

**Location:** internal/lensreport/lensreport.go:54

**Issue:** The format specifies an opening fenced YAML block whose `outcome` value determines whether the lens reported.
`Returned` instead searches the entire file for the exact LF-delimited bytes `outcome: reported`.
A valid opening block containing `outcome: "reported"`, or the documented block written with CRLF line endings, returns false.
`missingLensReports` then records that returned lens in `assist_lenses_missing`.
Conversely, a malformed file with no opening block but containing `\noutcome: reported\n` in its message returns true, so it escapes the missing-report record.
The new `TestReturned` covers a blockless message only when it lacks that text, leaving both directions untested.
This also contradicts the changed spec's claim that the reader distinguishes a reported outcome from any other file.

The fanned opening stencil and seat directive remove the identified work-alone conflict in the target diff.
The subagents are explicitly permitted, the write surface names the lens-report pattern, and `output_files` receives the concrete paths.
Those paths remain outside shuttle's required outputs and are cleared before a fresh run.
The solo stencil files are unchanged.
The new outcome reader is the only production reader found, and its spec and sole-parser background change together.
No Pattern-Departure trailer is present.

The requested real fanned-seat proof remains unperformed, as the landing summary explicitly acknowledges.
This review does not supply that proof.

Validation was attempted with `timeout 3600s lym gate test --tier 0 ./internal/lensreport ./internal/burlerengine ./internal/seatengine ./internal/shedadapters ./contracts/stencils`.
The command reported that it could not register the runner because the hub's runproc lock directory is read-only, then reported zero discovered tests and no log.
No test pass is claimed.

## Focus departures

- The cluster instruction says to “run one subagent per lens” and save separate lens reports.
  The authoritative opening prompt instead says “You work alone” and restricts the write surface to this report “and nothing else,” then reiterates that this output is the only file set to write.
  I followed that top-level restriction, reviewed the focus areas directly, and wrote no lens files.
  This instruction conflict belongs to the running review's old prompt, not the target's replacement stencils.

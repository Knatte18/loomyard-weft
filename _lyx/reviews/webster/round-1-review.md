---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/websterengine/triage.go:1
    summary: New and changed Go comments across the diff are wrapped at a fixed column instead of using semantic line breaks, violating CONSTRAINTS.md's Comment Line-Break Convention and card 11's explicit instruction
  - id: F2
    severity: MEDIUM
    location: internal/websterengine/verifyparse.go:52
    summary: parseVerifyFailures matches an indented --- FAIL line anywhere, so a test log line containing one yields a phantom test identity that can also make a real head failure look pre-existing
  - id: F3
    severity: LOW
    location: internal/websterengine/runlevel.go:1100
    summary: A failed friction-note write in the non-regression path aborts Run with an error and a zero RunResult, turning an informational Tier-2 note into a run failure after the report and summary were already written
  - id: F4
    severity: LOW
    location: internal/websterengine/integration.go:1
    summary: integration.go's file header still says the stage reads the fork's OK/FAILED through ParseReport and that a FAILED report triggers the bisect and escalation, both no longer true after triage
  - id: F5
    severity: LOW
    location: contracts/stencils/webster/webster-template-integration.md:35
    summary: The redirect example leaves the log path unquoted, so a path with spaces or Windows backslashes breaks the shell redirect and silently loses the first-run log
  - id: F6
    severity: NIT
    location: contracts/stencils/webster/webster-template-integration.md:59
    summary: The integration stencil still says the fork touches no file beyond reading, which now contradicts its own instruction to write the verify log
---

Review range: `product.parent` is `main` (from `_lyx/shed/verify-triage/status.json`); the range is `git diff 676d9ec03..HEAD`, cards 1–12 plus a content-free merge of `main`.
The plan's `## verify:` command passes in full on HEAD (build, untagged tests, `integration`-tagged tests, `go vet -tags smoke`).

### [MEDIUM] New Go comments use fixed-column wrapping, not semantic line breaks

**Location:** `internal/websterengine/triage.go`, `verifyparse.go`, `integration.go`, `integrationreport.go`, `report.go`, `runlevel.go`, `summary.go`, `beginbatch.go`, `doc.go`

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line, with further breaks only at independent-clause boundaries; card 11 repeats that rule for every changed comment line.
Most comments the diff adds or rewrites break mid-phrase at roughly column 100 (column 72 in `doc.go`).
Examples: `triage.go`'s file header ("touches no state, report, or / summary file"), `triageOutcome` ("the triage / classification"), `triageIntegrationFailure` ("leaves the first / run's failure set"), `runAtBaseline`, `triageWarnings`, `triageStuckReason`, `writeTriageFrictionNote`;
`verifyparse.go`'s `opaqueFailureID` and `parseVerifyFailures`;
`integration.go`'s `bisect` predicate paragraph, `checkoutAndVerify`, `bisectPassed`, `verifyRunner`, `runVerifyCapture`, and `LocalizeIntegrationFailure`'s regressions note;
`integrationreport.go`'s `IntegrationReport` and `ParseIntegrationReport`; `report.go`'s `validateReportFields`;
`runlevel.go`'s `RunResult.Outcome`/`StuckReason`/`Warnings`, the `Run` call-site comment, `runIntegrationStage`'s rewritten doc and phase comments, the baseline comment, and `recordTriageResult`;
`summary.go`'s `AppendIntegrationFailure` and `AppendIntegrationTriage`;
`beginbatch.go`'s `BeginResult.StartSHA` and the re-begin inline comment;
`doc.go`'s rewritten integration paragraph, including the stub line "BisectAndEscalate then records that".

**Fix:** Rebreak every new or changed comment at sentence and independent-clause boundaries; leave untouched legacy lines alone.

### [MEDIUM] Indented `--- FAIL:` log lines create phantom test identities

**Location:** `internal/websterengine/verifyparse.go:52`

**Issue:** The header match trims leading whitespace before checking for `--- FAIL:`, so any indented line starting with that text counts as a failing test.
A test that logs a multi-line string (for example, captured `go test` output, common in this repo's own tests) gets continuation lines indented under its entry.
Real `go test` output, reproduced in a scratch module, shows the phantom:

```
--- FAIL: TestA (0.00s)
    a_test.go:3: sub output:
        --- FAIL: TestPhantom (0.00s)
    a_test.go:3: boom
FAIL	ex.com/m/a	0.002s
```

The parser returns both `ex.com/m/a.TestA` and `ex.com/m/a.TestPhantom`.
The phantom reaches the report, the stuck reason, and the PR.
It can also excuse a regression: if a baseline test logs `--- FAIL: TestY` and `TestY` really fails at head, triage classifies the head failure pre-existing.
Top-level `--- FAIL:` headers always start at column 0 in `go test` output (with or without `-v`), and subtest headers are nested inside their top-level test's block, which already collapses them.

**Fix:** Match `--- FAIL:` only at column 0.
Add a `verifyparse_test.go` case with an indented `--- FAIL:` log line inside a failing test's block that must not yield an identity.

### [LOW] Friction-note write failure aborts the run

**Location:** `internal/websterengine/runlevel.go` (`recordTriageResult`, the `writeTriageFrictionNote` call)

**Issue:** On a flaky or pre-existing verdict, `recordTriageResult` returns `writeTriageFrictionNote`'s error, and `Run` then returns the zero `RunResult` with an error.
By then the report and the summary section are already written, and the run outcome is otherwise `done`.
The friction package treats note plumbing as best-effort: `friction.EnsureDir` logs a Warn instead of failing.
So a failed friction-directory creation also leads to a failed `WriteFile`, which fails the whole run.

**Fix:** In `recordTriageResult`, log a Warn on a friction-note error and continue, so state is still saved and the run keeps its outcome.

### [LOW] Stale `integration.go` file header

**Location:** `internal/websterengine/integration.go:1-12`

**Issue:** The header says the stage uses "webster's own ParseReport for the fork's OK/FAILED" and that "the in-process SHA-bisect + escalation path a FAILED report triggers".
The stage now parses with `ParseIntegrationReport` and triages first, and only a regression reaches bisect and escalation.

**Fix:** Update the header to name `ParseIntegrationReport` and to say that bisect and escalation run on a triaged regression.

### [LOW] Unquoted log path in the redirect example

**Location:** `contracts/stencils/webster/webster-template-integration.md:35`

**Issue:** `( <the verify command above> ) > {{.verify_log_path}} 2>&1` fills in an absolute path unquoted.
On Windows, `filepath.Abs` returns backslashes, and Git Bash consumes them as escapes.
A path with a space splits the word.
Either way, the log lands somewhere else or the redirect fails.
Triage then treats the first-run set as unknown and quietly loses the first-run identities.

**Fix:** Single-quote the path in the example: `> '{{.verify_log_path}}' 2>&1`.

### [NIT] Integration stencil still says the fork touches no file

**Location:** `contracts/stencils/webster/webster-template-integration.md:59`

**Issue:** "`deviations` is always empty — you touch no file in this repo beyond reading it to run the command above" now contradicts the instruction to redirect output into the scratch log.

**Fix:** Say the fork edits no file besides the verify log it redirects into.

## Settled focus items (no finding)

- **Parser package boundaries:** go test prints each package's output as one buffered block, so pending tests never cross a package.
  `pending`/`pendingSeen` reset at every `FAIL\t<pkg>` that closes a block with test failures.
  An `&&` chain stops after the first failing `go test`, so two invocations never both contribute failures.
  A `# pkg` build preamble ends in `FAIL\t<pkg> [build failed]` and yields the package identity.
  Tails match card 2: test tails are the indented lines after the header, and package and opaque tails are the last lines of the output, all capped at 40.
- **triage.go classification:** each branch matches card 5, and `Failures` puts head identities with rerun tails before first-only identities with first-run tails.
  In `runAtBaseline`, a `CurrentBranch` error returns before the deferred restore is armed.
  A `CheckoutDetached` error and a run error both go through the deferred restore, which joins its own error on.
- **Lease split in `runIntegrationStage`:** triage, the baseline checkout, and bisect all run unleased, and the report, summary, friction-note, and state writes all happen under the lease.
  The baseline comes from `batches[0]`'s `StartSHA`, as card 9 requires.
  The non-regression path writes no `-1` record.
  The `OutcomeDone` demotion applies only to a Master `done` and keeps `BatchesDone` and `SummaryTitle`.
  If a later write fails, earlier ones stay on disk, but a re-run archives `summary.md` and re-triages.
- **Re-entry:** `Run` archives `summary.md` at entry and runs the integration stage once, so no duplicate sections appear.
  A removed log leaves the first-run set unknown, and the rerun decides the verdict.
- **Bisect predicate:** `bisectPassed` matches card 6.
  `checkoutAndVerify` keeps its spawn-error path (only the message prefix changed) and its restore handling.
- **Blast radius:** every caller of `RenderIntegrationPrompt`, `LocalizeIntegrationFailure`, `BisectAndEscalate`, and `AppendIntegrationFailure` is updated.
  `ParseReport` behaves as before, and `runVerifyCommand` has no callers left.
  `webstercli`'s `start_sha` envelope field reports the inherited value.
  No stencil consumers in `internal/landingshed` pin the changed text.
- **Duplicated helpers:** none of the new helpers duplicates existing code; `appendToSummary` factors out the old append body.
- **Stencils:** the header's marker count (four required) matches the file.
  The Master `FAILED → outcome: done` rule agrees with the stencil's final-action definition of `done`.
  Stencil prose uses semantic line breaks.
- **Chained verify:** a head run can be classified pre-existing while later `&&` links never ran.
  The summary text ("did not attribute the failure to this run") and the warning ("also fail at baseline") do not claim the later links passed.
  Treating the unrun links as uncovered is the plan's own D7 design, so it falls outside this segment.

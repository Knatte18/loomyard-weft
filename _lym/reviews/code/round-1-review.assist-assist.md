---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/gatecli/tierrun.go:487
    summary: The vstest counter now changes the invocation protocol of existing non-test runners without migration.
    fix: "Separate the controlled-results protocol from the output counter in gate configuration and invocation rendering. Preserve existing non-dotnet-test commands; register a config reinterpretation for any changed setting meaning, and update docs/dotnet-hub.md and the landing summary's merge-danger classification accordingly."
    origin: lens:code-affected-set
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/gatecli/tierrun_test.go:453
    summary: The new TRX and environment tests spawn real shells while assigned to tier0.
    fix: Move TestRunTierReadsTheVSTestResultsFile and TestRunTierNamesEnvThatDiffersFromTheSession into a tier1 test file with a !windows build constraint, require tier1, and remove their Windows runtime skips.
    origin: lens:code-affected-set
  - id: F3
    severity: BLOCKING
    class: design
    location: internal/gatecli/tierrun.go:243
    summary: Environment notices disclose both the session credential and its tier replacement in retained logs.
    fix: Report differing environment variable names without raw values in envOverrideLine, update its documentation and expected notice, and assert that neither credential value appears in stderr or the tier log.
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/gateslot/tiers.go:436
    summary: MeasureCall emits commands that the gate refuses when selected tiers lack argument support or no runnable tier exists.
    fix: "Validate the measuring form against the effective selected tiers, including {args}, target handling after covers, and scope requirements. Choose a declared usable tier when the key cannot measure; return an unavailable result when none exists, and render a parent-directed way forward instead of a fictitious command. Cover absent config, missing keys, argument-free tier0, and mixed tiers."
    origin: lens:code-correctness
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/gatecli/tierrun.go:492
    summary: Pass-through TRX logger options can replace the gate-owned logger and restore console-dependent counts.
    fix: Normalize or refuse conflicting pass-through logger and results-directory options before invocation rendering so the gate retains its own result location and TRX configuration, while allowing ordinary console loggers. Add a boundary case with a conflicting TRX filename and a detailed console logger.
    origin: lens:code-correctness
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/gatecli/tierrun.go:545
    summary: Buffered console output is parsed after TRX replacement and can change the final counts or failures.
    fix: Flush each invocation's pending console line before replacing that invocation with TRX results. Verify an unterminated summary and failed-test line cannot add results after replacement or bleed into the next invocation.
    origin: lens:code-correctness
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/gatecli/tierrun_test.go:515
    summary: TRX tests do not pin replacement across invocations or the new result-file boundary cases.
    fix: Extend the existing TRX behavior table with sequential invocations whose parseable console counts disagree with their files, multiple files, missing and malformed output, and skipped/Error/Timeout/Aborted outcomes. Assert literal totals and failure lists, retention of earlier invocations, and cleanup after startup failure.
    origin: lens:code-tests
  - id: F8
    severity: MEDIUM
    class: design
    location: internal/gatecli/tierrun_test.go:512
    summary: The crash fixture supplies the expected message without checking that the real logger and collector produce it.
    fix: Make the boundary fixture enforce the TRX logger and collector options. Extend the existing real .NET boundary fixture to exercise a command with {args}, detailed console logging, and a crashing host, asserting captured crash evidence and parsed results.
    origin: lens:code-tests
  - id: F9
    severity: MEDIUM
    class: design
    location: internal/gatecli/run.go:104
    summary: No test verifies capture of the invoking session's environment for the new notices.
    fix: Extend the CLI integration test with an inherited harmless variable and a differing tier override, asserting its name in stderr and the tier log; add an equal-value case. Do not inject sessionEnv directly in this boundary check.
    origin: lens:code-tests
---

### [BLOCKING] Existing runner configurations change meaning

**Location:** internal/gatecli/tierrun.go:487

**Issue:** The new branch selects a command protocol solely from `counter: vstest` and the presence of `{args}`.
The timing example in `docs/dotnet-hub.md:145` runs a benchmark application through `dotnet run -- {args}`.
It now receives test-runner arguments that were never part of its configured invocation.
`Tier.Counter` describes output counting, not the executable's accepted options.
The gate registration in `internal/configreg/configreg.go:196` supplies no reinterpretation.
This violates `PATTERN-config-layers` when changing the setting's meaning.
The landing summary's two-way door also conflicts with the rubric's rule for changed config semantics.

### [BLOCKING] New subprocess tests belong in tier1

**Location:** internal/gatecli/tierrun_test.go:453

**Issue:** Both new tests declare tier0 and call `runTier`, which reaches `exec.CommandContext` and `runTiedChild`.
The TRX fixture additionally invokes `cp`.
`PATTERN-test-speed` explicitly reserves subprocesses for tier1 and leaves indirect spawning to review.
The new runtime skips on Windows also violate its platform-file rule.
This is not a duplicate of a failing mechanical scan: the spawn occurs through production helpers.

### [BLOCKING] Environment differences expose secrets

**Location:** internal/gatecli/tierrun.go:243

**Issue:** A session holding `API_TOKEN=production-secret` and a tier setting another token prints both values.
The notice is copied to stderr and the retained tier log by `runTierCommands`.
Thus a credential the child never receives is disclosed by the diagnostic feature itself.
Naming the differing variables satisfies the requested warning without exposing either value.

### [MEDIUM] Rendered measuring calls can still be refused

**Location:** internal/gateslot/tiers.go:436

**Issue:** Consider tier0 without `{args}`, tier1 with `{args}`, and no `darn-write` key.
The renderer selects tier0 and appends `-- <args>`.
`planTierRuns` rejects that form at `internal/gatecli/tiers.go:117`.
A present key containing an argument-free retained tier fails the same way.
The zero-config fallback is refused because tier0 is absent;
an absent configuration is rejected even earlier by `loadGate`.
Checking `Gate` and target placeholders alone does not establish that the emitted measuring call can run.

### [MEDIUM] Caller options can replace the controlled logger

**Location:** internal/gatecli/tierrun.go:492

**Issue:** The gate's logger precedes the caller's arguments.
A later TRX logger naming a file outside the temporary directory can replace its configuration.
Microsoft's [logger settings implementation](https://raw.githubusercontent.com/microsoft/vstest/main/src/vstest.console/Processors/Utilities/LoggerUtilities.cs)
replaces an existing matching logger's configuration when later parameters are supplied.
The gate then finds no TRX file and preserves console counts.
With a detailed console logger, the original zero-count failure returns.

### [MEDIUM] Pending console lines survive replacement

**Location:** internal/gatecli/tierrun.go:545

**Issue:** `lineReader.Write` retains a final line without a newline in `pending`.
If the TRX reports one pass and the pending line is a one-pass console summary, replacement records one pass.
The later `counter.Flush` in `runTierCommands` parses the summary and increases the total to two.
A pending failed-test line can similarly add a console-derived failure after replacement.
The replacement must occur after the invocation's complete console contribution has been consumed.

### [MEDIUM] Result replacement and file boundaries lack coverage

**Location:** internal/gatecli/tierrun_test.go:515

**Issue:** Every new TRX row runs one invocation and writes one valid result file.
Its console output is deliberately unparseable, so it never verifies removal of already-counted console results.
It also cannot catch loss of earlier invocations when the later file replaces them.
The snapshot mutation survivor at `counters.go:475` corresponds to this observable gap.
Multiple-file aggregation, malformed-file fallback, and the additional outcome mappings have no assertions either.
The existing cleanup assertion covers started commands, but not startup failure.

### [MEDIUM] Crash evidence is supplied by the fixture

**Location:** internal/gatecli/tierrun_test.go:512

**Issue:** The fake command only finds `--results-directory` and copies a prewritten file.
It ignores both `--logger` and `--blame`, so removing either still satisfies its crash assertion.
The real .NET fixture at `internal/gatecli/dotnetfixture_integration_test.go:168` lacks `{args}`.
It therefore never exercises the new controlled-results path.
No boundary test establishes that the requested options produce the crash evidence the gate promises.

### [MEDIUM] Environment capture is untested at the CLI boundary

**Location:** internal/gatecli/run.go:104

**Issue:** The new environment test injects `sessionEnv` directly into `testRequest`.
Deleting the production capture leaves that test green while ordinary calls stop reporting differences.
The integration harness removes inherited `GOFLAGS`, so it does not verify the capture either.

## Rejected

- The code-tests static-wording finding is rejected: `PATTERN-test-speed` permits a Go-filled marker's position to be pinned, which the adjacent phrase checks here.
- The code-tests tier0 finding duplicates F2 and is consolidated there.
- The cleanup warning mutation is not a missing removal check: the test already asserts the directory is gone.
- The handler's buffered-output and argument-support findings duplicate F6 and F4 respectively.

## Review checks

The board entry, decision record, supplied diff pieces, commit trailers, and relevant callers were reviewed.
The new marker is supplied by both production table builders, and the stencil retains its other rules.
Unknown-key names are sorted, and normal/start-failure cleanup reaches directory removal.
The Root-Cause trailers identify the defects their commits address.
The parent's amendment explicitly waives the red-first gate for this run; its stated reason is the non-Go stencil.
No additional waiver or finding against that decision is inferred.

The installed `dotnet test --help` was inspected, and Microsoft's logger implementation was checked.
No test suite was rerun: the available shell tool exposes no timeout parameter for the required 3600000 ms gate call.
The supplied reach, redundancy, and mutation reports were treated as advisory evidence, not fresh executions.

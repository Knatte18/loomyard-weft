---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/websterengine/triage_test.go#TestWriteTriageFrictionNote
    summary: No test pins that a note holding only verify command failures leaves out the flaky-test header, which is the exact defect issue 160 reports.
    fix: "In internal/websterengine/triage_test.go, TestWriteTriageFrictionNote, add a subtest \"command failures only\" that writes []flakyFailure{testFlaky(opaqueFailureID, FailureKindOpaque, \"/v/3.log\", \"gate test: build failed\")} and compares the note by exact equality to \"Integration-suite triage: non-regression verify failures\\n\\n\" followed by the command-failure paragraph, the bullet \"- first run's log /v/3.log\" and its fenced tail, as the content subtest spells them, so a note that writes the \"(flaky):\" header unconditionally fails."
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/websterengine/triage.go#writeFailureOutput
    summary: A tail line that holds three backticks closes the note's fixed three-backtick fence early, so the rest of the output renders as markdown.
    fix: In writeFailureOutput, find the longest run of backticks in tail and open and close the block with a fence one backtick longer, never shorter than three; add a row to TestWriteTriageFrictionNote's content subtest whose tail holds a three-backtick line and assert the four-backtick fence by exact equality.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/websterengine/verifygate_test.go#TestVerifyGate_FlakyFailurePassesOnRerunWithNote
    summary: The gate fake returns the same log content for the first run and the rerun, so the test cannot tell the first run's output tail from the rerun's.
    fix: "In TestVerifyGate_FlakyFailurePassesOnRerunWithNote, set the fake's logs to []string{tt.firstLog, \"rerun output only\\n\"} and assert the friction note does not contain \"rerun output only\"."
    origin: assist:assist
  - id: F4
    severity: LOW
    class: consistency
    location: internal/websterengine/verifygate.go:492
    summary: The rerun-pass log line still lists the opaque verify-command identity under the key flaky, the one place left that calls a verify command failure flaky.
    fix: "In newVerifyGate's rerun-pass branch, compute ids, commandLogs := splitFlaky over the failures just recorded (or over first with firstLog) and log ids under the key \"flaky\" and commandLogs under a key \"verify_command_failed\", each only when non-empty."
    origin: lens:code-affected-set
  - id: F5
    severity: LOW
    class: consistency
    location: contracts/specs/webster-spec.md:72
    summary: The spec, the package doc and triageWarnings' doc say a first run that named no failing test is a verify command failure, but a run naming only a failing package is still listed as flaky.
    fix: "Change \"named no failing test\" to \"named no failing test or package\" in contracts/specs/webster-spec.md:72, internal/websterengine/doc.go:993 and the second doc line of triageWarnings in internal/websterengine/triage.go."
    origin: handler
  - id: F6
    severity: LOW
    class: consistency
    location: contracts/stencils/landing/landing-template-describe.md:35
    summary: The landing describer is told to name the flaky and pre-existing identities the Integration suite triage section lists, and has no wording for its new list of verify command failures named by log.
    fix: "Extend the sentence at contracts/stencils/landing/landing-template-describe.md:35 so it also says to name each verify command failure the section lists by its first run's log, as a failure of the verify step rather than a flaky test."
    origin: lens:code-affected-set
  - id: F7
    severity: NIT
    class: consistency
    location: contracts/specs/loom-final-summary-spec.md:92
    summary: The pointer to webster-spec still describes webster's section as listing only identities that passed on rerun.
    fix: "In contracts/specs/loom-final-summary-spec.md:92, change \"the flaky-failure section the verify gate appends for identities that passed on rerun\" to \"the flaky-failure section the verify gate appends for identities, and verify command failures, that passed on rerun\"."
    origin: lens:code-affected-set
---

### [MEDIUM] The command-failure-only note is not pinned against the flaky header

**Location:** internal/websterengine/triage_test.go#TestWriteTriageFrictionNote
**Issue:** Issue #160 is a note listing `verify-command` under "These identities … (flaky):".
`TestWriteTriageFrictionNote/content` always mixes a test failure with the command failure, so the header is always expected.
The all-opaque row of `TestVerifyGate_FlakyFailurePassesOnRerunWithNote` (`verifygate_test.go:413-416`) checks only that expected lines are present.
A writer that emitted the flaky header unconditionally (`triage.go:80`) would pass every test.

### [LOW] A backtick fence in the tail breaks the note's block

**Location:** internal/websterengine/triage.go#writeFailureOutput
**Issue:** The block opens and closes with a fixed three-backtick fence (`triage.go:182`, `:186`), and tail lines are copied unchanged.
An opaque tail passes the whole output through `outputTail`, so any three-backtick line in a tool's output ends the block early.

### [LOW] The flaky-gate fixture cannot tell the first run's output from the rerun's

**Location:** internal/websterengine/verifygate_test.go#TestVerifyGate_FlakyFailurePassesOnRerunWithNote
**Issue:** The fake repeats its last log for later calls (`verifygate_test.go:86`), and the test supplies one log, so both runs read the same bytes.
The log path is pinned to `verify-1.log`, but a regression that took the tail from the rerun's output would pass.

### [LOW] The rerun-pass log line calls a command failure flaky

**Location:** internal/websterengine/verifygate.go:492
**Issue:** `logger.Warn` logs `failureIDs(first)` under the key `flaky`, so a `verify-command` failure lands in the log as flaky.
The warning, the note and `summary.md` all report it apart, and `doc.go:993` says it is "never" reported as a flaky test.

### [LOW] The classification rule is worded wider than the code

**Location:** contracts/specs/webster-spec.md:72
**Issue:** `ParseVerifyRun` emits a package identity for a build failure or a package with no failing test (`verifyparse.go:106-107`).
Such a first run named no failing test, yet `commandFailure` keeps it among the flaky identities, which the "a failing package" test row asserts.
The same wording sits in `doc.go:993` and `triage.go:41`.

### [LOW] The landing describer does not know the new sub-list

**Location:** contracts/stencils/landing/landing-template-describe.md:35
**Issue:** `AppendIntegrationTriage` now writes a "Verify command failed naming no failing test" list of log paths into the section (`summary.go:57`).
The stencil asks only for the flaky and pre-existing identities, so a PR description can drop the command failure or name it as flaky.

### [NIT] The final-summary spec's pointer is stale

**Location:** contracts/specs/loom-final-summary-spec.md:92
**Issue:** The pointer to `webster-spec.md` still says the section lists only identities that passed on rerun.

## Rejected

- Assistant F1 (BLOCKING, repeated `webster-verify-triage.md` filename in tests, under `PATTERN-code-conventions`): the writer's literal is unchanged from the base. A test spelling the expected file name is the test's independent oracle, so a rename fails it loudly instead of going stale. Deriving it from the writer's constant would make the check tautological.
- Code-correctness F2 (`addFlaky` drops a second opaque failure's log across gate evaluations): a flaky pass returns `pass()` and ends the gate's evaluations. `ParseVerifyRun` already dedups within one first run. No path that records a second flaky first run into the same notes was established.
- Code-tests F2 (`addFlaky`'s dedup branch untested): the dedup is base behaviour whose key changed only with the element type, and per the item above no path reaches the `found` branch. It is no new behaviour without coverage.
- Code-correctness F3 duplicates F4 and is merged into it. Assistant F2 duplicates F2, assistant F4 duplicates F5.
- Weaver's own `## Integration suite triage` writer (`internal/weaverengine/verify.go:355`) still lists every identity as flaky. The board entry concerns webster's verify triage, so this is no finding against this diff.

## Focus checks

- Cwd-reset bullet: the diff adds no filter, and none is needed. The tail comes from the verify log, which `verifytree` captures from lym's own subprocess, and no lym code writes the "Shell cwd was reset" line. The harness notice cannot enter an identity or the note body. `_lym/landing/summary.md` states this reasoning.
- Command-failure classification: `ParseVerifyRun` yields `FailureKindOpaque` only as the single identity when nothing parses, so a first run naming no test or package always lands there. `splitFlaky`, `triageWarnings`, `writeTriageFrictionNote` and `Apply`'s call into `AppendIntegrationTriage` keep it out of the flaky list on every path. The one exception is the log line (F4).
- `addFlaky` dedup: not a defect; see the Rejected item.
- `writeFailureOutput`: trailing newlines are trimmed by `outputTail` for package and opaque tails, and the test tails' blank lines stay inside the fence. Empty output reads "Its output was empty.". A fence in the tail is F2.
- Revert and reapply: the net spec diff is commit 87f6be00, as the parent's entry keeps it. The spec and `doc.go` state what the code does, bar F5. No `Flaky()` caller or doc reference remains.
- Test economy and loosened checks: the base's exact `notes.Flaky()` equality is replaced by an exact `DeepEqual` on `Apply`'s warnings, which pins the same identity list. No assertion is loosened, and every new expected value is a literal.

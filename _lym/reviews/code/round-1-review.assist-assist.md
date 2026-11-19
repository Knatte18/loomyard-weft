---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/testkit/tmuxkit/gitentry.go:76
    summary: The diff adds diagnostics but does not fix the test that writes the stray repository marker.
    fix: Identify the writing test or the code it drives and include the change that confines its writes to its fixture directory, with a regression assertion in the test of that behavior. The current goal still requires this repair; instrumentation alone does not complete it.
    origin: lens:code-scope
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/testkit/tmuxkit/gitentry.go:206
    summary: A delayed or incomplete process scan can still fail an innocent package.
    fix: Treat a proc-only snapshot as candidate information rather than proof of authorship. In snapshotProcesses and checkNewGitEntries, preserve the package exit code unless independent evidence identifies this package as the writer; report the observation as a machine finding otherwise. Update the corresponding documentation and existing tests, including a foreign writer that exits before observation and unavailable process information.
    origin: lens:code-correctness
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/testkit/tmuxkit/tmuxkit_test.go:347
    summary: The retained contained classification test lacks the required keep annotation.
    fix: Add a testtiming keep directive immediately above TestClassifyProcesses naming its exact other-binary list, descendant exclusion and diagnostic ordering assertions, which the live snapshot test does not check.
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/testkit/tmuxkit/gitentry.go:80
    summary: Watch shutdown can discard a creation and leave no finding when the entry has already disappeared.
    fix: Give watchGitEntries a shutdown protocol that preserves creation events queued before run completion before cancelling delivery. Extend TestRunCheckingGitEntries with a controlled create-and-remove case whose run returns without waiting for its snapshot, and assert that a machine finding survives shutdown. Cover watch-open failure through the same behavior test.
    origin: lens:code-correctness
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/testkit/tmuxkit/gitentry.go:108
    summary: Repeated creations overwrite earlier evidence and can change the run verdict.
    fix: Preserve each creation sighting per path, or accumulate all evidence relevant to reporting and the exit decision, instead of replacing it with the latest sighting. Add a row covering two creations with differing candidate sets and verify that the first observation remains reportable. If F2 removes snapshot-based failures, retain the earlier diagnostic evidence without restoring that unsafe failure rule.
    origin: lens:code-correctness
  - id: F6
    severity: MEDIUM
    class: consistency
    location: internal/testkit/tmuxkit/gitentry.go:161
    summary: The diagnostic and pattern text falsely promise that a common named binary identifies the writer.
    fix: Describe the listed binaries as concurrent candidates and remove the guaranteed-writer claim from describeGitEntry, pattern/PATTERN-test-isolation.md and the landing summary. Any intersection-based aid must include the reporting package in its candidate set and explicitly state that overlap does not establish authorship.
    origin: lens:code-affected-set
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/testkit/tmuxkit/gitentry.go:192
    summary: The off-Linux attribution fallback has no test of its production path.
    fix: Exercise the actual off-Linux snapshot-to-finding path in a platform-appropriate test and assert the preserved timestamp, unattributed result, diagnostic and unchanged exit code. Do not substitute a hand-built unattributed sighting for the production snapshot; retain the existing Linux boundary test.
    origin: lens:code-tests
  - id: F8
    severity: LOW
    class: design
    location: internal/testkit/tmuxkit/tmuxkit_test.go:351
    summary: The new diagnostic cap and incomplete-ancestry outcomes are untested.
    fix: Extend the existing classification behavior test with an argv exceeding maxArgvChars and test-binary parent chains that are missing or cyclic. Assert the bounded diagnostic and the resulting candidate classification without adding separate per-helper tests.
    origin: lens:code-tests
---

### [BLOCKING] The writing test remains unfixed

**Location:** internal/testkit/tmuxkit/gitentry.go:76
**Issue:** The Goal requires identifying the writer and fixing its test.
The range adds observation and changes attribution; it changes no test's writing behavior.
The fswatch package declaration avoids an import cycle, as its new header states.
The landing summary explicitly acknowledges the omitted repair under Design decisions.
There is no scope amendment or claimed parent correction that permits deferring it.

### [BLOCKING] A later process scan cannot establish authorship

**Location:** internal/testkit/tmuxkit/gitentry.go:206
**Issue:** Let B create the entry while A runs, then exit before A consumes the event and reads B's argv.
The scan omits B but still returns an attributed sighting with no other test binary.
The check at line 140 then fails innocent package A.
Pids also converts an unreadable process directory to an empty slice in proc.go:14–18.
ProcArgv failures and empty command lines are discarded, and ancestry reads are not identity-stable across PID reuse.
These are observations after creation, not proof of who was alive at creation or who wrote the entry.
The board explicitly permits a machine finding without failing the package, which avoids this unsupported inference.

### [BLOCKING] Preserve the classification test's distinct assertions explicitly

**Location:** internal/testkit/tmuxkit/tmuxkit_test.go:347
**Issue:** The current redundancy report marks this test contained by the live snapshot test.
Its exact other-binary list, descendant exclusion and ordering assertions are absent from that covering test.
PATTERN-test-speed's Test economy section requires a keep directive for this retained candidate.
The directive makes every listed constraint violation BLOCKING, including this annotation requirement.

### [MEDIUM] Shutdown can erase the only evidence of a transient entry

**Location:** internal/testkit/tmuxkit/gitentry.go:80
**Issue:** A run can create and remove the entry, then return before event delivery.
The stop closure calls fswatch.Close, which closes done before waiting for forwarding to finish.
Both selects in fswatch.forward may abandon delivery on done.
With no sighting and no surviving entry, checkNewGitEntries emits nothing.
Waiting for the snapshot inside every creating test run excludes this schedule from existing coverage.
This is distinct from the declared no-sighting fallback for a surviving entry, which still emits a finding.

### [MEDIUM] A later creation replaces an earlier observation

**Location:** internal/testkit/tmuxkit/gitentry.go:108
**Issue:** Each path stores only its latest sighting.
If an alone sighting is followed by removal and a second creation beside another binary, the second overwrites the first.
The current exit decision changes from failure to success, and the earlier time and candidates disappear.
Even after correcting attribution, discarding earlier observations loses evidence needed to investigate the writer.

### [MEDIUM] Candidate overlap is not writer identification

**Location:** internal/testkit/tmuxkit/gitentry.go:161
**Issue:** When A writes while A and B run, A's others list names B and B's list names A.
Their intersection is empty because each observer excludes itself at line 221.
Across occurrences, a recurring innocent neighbour can also remain in the intersection.
The diagnostic, pattern background and landing summary nevertheless promise that the common binary is the writer.

### [MEDIUM] The platform fallback is asserted only as a fabricated result

**Location:** internal/testkit/tmuxkit/gitentry.go:192
**Issue:** The only direct snapshot test is Linux-only.
The unreadable-platform row in TestCheckNewGitEntries supplies a hand-built sighting.
It would pass if the production off-Linux branch returned an attributed result or lost its timestamp.
Failed Linux reads are covered by F2's required regression; this finding covers the separate platform branch.

### [LOW] Diagnostic boundary behavior lacks assertions

**Location:** internal/testkit/tmuxkit/tmuxkit_test.go:351
**Issue:** Every fixture argv is short, so removing the truncation at gitentry.go:249 would leave the tests green.
The fixture's missing-parent process is bash and never enters the test-binary ancestry check.
No row exercises a missing or cyclic test-binary parent chain, although process reads can produce incomplete snapshots.
These cases belong in the existing diagnostic/classification behavior test.

## Consolidation

The correctness lens's diagnostic finding and the affected-set finding are combined in F6.
The tests lens's shutdown finding is combined in F4; its unavailable-process coverage requirement is combined in F2.
No lens finding was rejected outright.

## Focus outcomes and validation

The fswatch test-package move is a necessary import-cycle repair, not a fix to the writing test.
The pattern update is a consequential documentation change, not unrelated scope.
The moved cleanup retains its directory, gitfile and symlink behavior.
No changed caller or external parser was left expecting the old helper signatures or diagnostic.

The Loosened-Check trailers identify the changed assertions and name a historical measurement.
The gate-log search found the stated 22 matching runs through 2026-11-18, plus later occurrences.
Those logs establish failures and cleanup, not the claimed identity or innocence of every affected writer.
The rewritten tests still assert failure for the supplied alone sighting; F2 concerns whether that premise is justified.

Creation before watch installation and a surviving entry missed by the watch take the documented machine-finding path.
Watcher errors are logged by fswatch; no sighting still falls back to the final filesystem check.
No-sighting handling itself is not a separate defect.
The off-Linux implementation returns an unattributed sighting as described; its missing production-path test is F7.
The live snapshot test kills and waits for its child; its sleep argument does not make the test wait that duration.
Comment wrapping and mechanical build/test results are not review findings.

The focused foreground gate could not acquire a slot because this session cannot write its holder record:
`/home/knatte/Code/loomyard-LYMHUB/_bolt/.lym/gate` is read-only here.
No test ran in this review attempt, and no target file was changed.
The findings above come from code paths and the supplied current reach and redundancy reports.

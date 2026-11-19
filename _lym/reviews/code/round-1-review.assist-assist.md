---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/gatecli/mutation.go:47
    summary: The new report dispatcher adds a forbidden language-literal branch.
    fix: Route mutation-report eligibility through a language capability table keyed by the configured scoper, and replace the new comparison against the go literal with that lookup.
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/gatecli/mutation.go:115
    summary: A mutation report can match HEAD after test inputs change and invalidate its verdicts.
    fix: Stamp and compare all inputs used by the mutation run, including package-owned fixtures and linked-package inputs, or conservatively record and compare the complete judged tree. Reject incomplete stamps. Add a regression that changes a testdata file after writing a report and requires MutationReportMatches to return false.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/testcover/mutant.go:803
    summary: The log-only filter drops mutations of side effects in conditions and logger arguments.
    fix: Restrict noise spans to expressions proven to affect only logging. Retain mutation sites inside side-effecting conditions and arguments, and extend TestPlanLineMutants with a multiline condition or argument whose changed integer alters observable state.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/mutate.go:298
    summary: A later execution failure discards mutation findings collected before the failure.
    fix: Return the accumulated outcome from diffMutation.run on failure, assign it in mutationCall.run before handling the error, and classify unfinished work with its failure reason. Extend the report-step test with a completed package followed by an execution failure and assert that the earlier findings remain in mutation.json.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/mutation.go:128
    summary: No report-step test asserts persistence of nonempty findings lists.
    fix: Extend the existing TestGateMutate report-step behavior with nonempty survivor, unjudged-mutant, uncovered-block and not-judged-package results. Read mutation.json and compare each list to the expected entries, using the existing fixtures where possible.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/mutation.go:39
    summary: Mutation-step skip and advisory failure behavior lacks coverage.
    fix: Add rows to the existing report-step behavior test for red gates, calls outside a hub, unsupported scopers and a mutation-check failure. Assert the saved reason or error, absence of mutation work for ineligible calls, and preservation of the gate result.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/config_test.go:301
    summary: The Loosened-Check trailer supplies a feature rationale rather than a measured cause.
    fix: Record observed before-and-after config-loading results in a Loosened-Check trailer within the review range for the removed mutation-rejection assertion. Keep the requested acceptance behavior and its replacement test.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: _lym/landing/summary.md:19
    summary: The summary incorrectly promises representation of every package and file.
    fix: Replace the claim that every package and file gets a share with a description of even sampling over functions ordered by package, file and name, without promising representation of every group.
---

### [BLOCKING] Language choice bypasses the required table

**Location:** internal/gatecli/mutation.go:47

**Issue:** The new dispatcher compares the configured scoper directly with the Go language literal.
`PATTERN-code-conventions` forbids language-literal branches outside a keyed language table.
It explicitly says existing instances are debt and must not justify another instance.
No review-range departure explains this addition.

### [BLOCKING] Freshness ignores inputs that determine mutation outcomes

**Location:** internal/gatecli/mutation.go:115

**Issue:** The report hashes only Go files in packages containing changed functions.
`MutationReportMatches` delegates to `changedFunctionsMatch`, which checks those hashes and changed-function identities.
Suppose a covering test reads `testdata/expected.json` and that file changes after the report is written.
The changed-function set and recorded Go hashes remain equal, so the old survivor or kill verdict still matches HEAD.
`goFileHashes` excludes the fixture, whereas `testreach.Fresh` checks package-owned files and linked inputs.
The new consumer therefore violates `PATTERN-coverage-map-reader`'s requirement to use reports only for their judged tree.

### [MEDIUM] Logging classification hides observable work

**Location:** internal/testcover/mutant.go:803

**Issue:** `logsOnly` checks an if statement's initializer and bodies but ignores effects in its condition.
For a condition calling `counter.Add`, the statement changes counter state even when its body only logs.
A changed integer argument on its own line is a legitimate mutation site with an observable result.
The whole-statement noise span at `fileSites` suppresses that site.
The same broad classification suppresses side-effecting expressions passed as logger arguments.
The goal authorizes skipping statements that only log, not suppressing unrelated effects evaluated within them.

### [MEDIUM] Execution errors erase completed results

**Location:** internal/gatecli/mutate.go:298

**Issue:** Let package A finish with a survivor, then let package B fail to start a test or build its required lym binary.
`diffMutation.run` returns an empty outcome on that error.
`mutationCall.run` also assigns the outcome only on success.
The advisory step writes its error alongside zero counts and empty lists, losing A's completed finding.
Planning results already collected, including uncovered blocks and packages not judged, are lost too.
The new report needs the full findings list even when later work fails.

### [MEDIUM] Persisted findings are unasserted

**Location:** internal/gatecli/mutation.go:128

**Issue:** The report-step integration row produces one killed mutant and checks counts and function identity.
Its findings lists are empty.
Removing the list assignments would leave that row green.
Existing hand-verb rows exercise `fillMutateAnswer`, and rendering tests construct reports directly.
Neither checks this persistence path.

### [MEDIUM] Advisory failure and skip paths are unasserted

**Location:** internal/gatecli/mutation.go:39

**Issue:** The report-step integration row covers a successful eligible call only.
It does not pin skipping after a red gate, outside a hub or with an unsupported scoper.
It also does not pin error recording while preserving the enclosing gate's result.
The summary tests fabricate notes and errors, and the reach-step tests exercise a different implementation.

### [MEDIUM] Removed assertion lacks the required measurement

**Location:** internal/gateslot/config_test.go:301

**Issue:** Commit `e49d389cf5347591449a235d756ce41e173e5a5a` names the removed rejection assertion in its trailer.
Its reason says mutation is now accepted, but supplies no measured before-and-after loading result.
The rubric explicitly requires a measured cause for a removed assertion.
The new acceptance behavior itself is requested and should remain.

### [LOW] Sampling does not guarantee every group a share

**Location:** _lym/landing/summary.md:19

**Issue:** The summary says every package and file now gets a share.
For 100 ordered functions with the last alone in package B, a cap of 40 samples indices through 97.
Package B receives no mutant.
The implementation spreads over functions as requested; the categorical description exceeds that guarantee.

## Review notes

The scope lens found each change traceable to the board entry or a supporting update.
The cap stays deterministic and within its bound; current-platform comparisons retain mutation eligibility.
The summary prints the requested counts, and the marker passes through Bouncer and Burler wiring.
The historical trial figures are attributed to the trial rather than claimed as a new measurement of this branch.

The focused gate command was attempted in the foreground with a one-hour shell timeout.
It could not acquire a slot because the sandbox denied writes beneath the hub's `_bolt/.lym` directory.
No test executed in that attempt; this infrastructure refusal is not a code finding.

## Focus departures

- “Drive the real substrate: build, run, test what you review” could not be completed after the slot-write refusal.
  Source traces and the supplied gate reports support this review, but do not replace an executed regression check.

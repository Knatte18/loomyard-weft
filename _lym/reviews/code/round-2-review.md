---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/gatecli/mutatecheck.go#diffMutation.run
    summary: A mutation check that fails after planning returns an empty outcome, so the report of a failed call loses the uncovered blocks, not-judged packages and verdicts already found, though recordFindings' doc comment says it keeps the partial findings.
    fix: >-
      In internal/gatecli/mutatecheck.go, make diffMutation.run return d.outcome, with d.outcome.runs set from d.runs, on the planPackage context error and on a runPackage error, instead of mutateOutcome{}.
      Leave the holdSlot error returning an empty newMutateOutcome().
      In internal/gatecli/mutate.go, in mutationCall.run, assign found.outcome = outcome before the err check after check.run(), so the error return carries it.
      Keep recordFindings recording no Functions or FileHashes on error.
      The verb's error path prints only the error, so its envelope is unchanged.
    origin: assist:assist
    carries: F2
  - id: F2
    severity: LOW
    class: design
    location: internal/gatecli/mutation_test.go#TestMutationStep_RecordFindings
    summary: No test asserts that the step copies the unjudged mutants and the not-judged packages, both the pre-check ones and the check's own, into the report; dropping either assignment leaves every test green.
    fix: >-
      In TestMutationStep_RecordFindings, give found a notJudged entry, such as MutationNotJudged{Package: "calc", Reason: "pre-check"}.
      Give found.outcome an unjudged entry and a notJudged entry of its own.
      Add the expected Unjudged list and the NotJudged list, with the pre-check entry first, to each row's want.
    origin: assist:assist
    carries: F5
prior:
  - id: F1
    status: resolved
    answer: mutationStep.run, reach.go, redundancy.go, reachverb.go and mutate.go now ask gateslot's Config.HasGoCoverageMaps, which reads the scoper-keyed table scoperHasGoCoverageMaps.
  - id: F2
    status: open
    answer: >-
      The stale match is fixed: recordFindings records no Functions or FileHashes on error, and TestMutationStep_RecordFindings pins it.
      The partial-findings half stays open, since diffMutation.run returns mutateOutcome{} on a run failure and mutationCall.run never assigns it, so the disposition's claim that partial counts are kept holds only for refresh errors and pre-check not-judged packages.
  - id: F3
    status: resolved
    answer: gateslot/doc.go names the mutation report step as the fourth non-tier slot holder, beside each shared bound, and states the even spread of mutation_max_mutants.
  - id: F4
    status: resolved
    answer: TestGateMutate rows cover the red tier, no hub, a none scoper and no parent branch, each with its single summary line, and a red tier call keeps exit 1.
  - id: F5
    status: open
    answer: Survivors and uncovered blocks are now asserted from mutation.json; the unjudged mutants and not-judged packages the summary also named are still asserted only on the hand verb's envelope.
  - id: F6
    status: resolved
    answer: The report-step row refuses a HEAD that changes Clamp again, and TestMutationReportMatches_RefusesAReportThatJudgedNothing covers note, red-gate and prime.
  - id: F7
    status: resolved
    answer: logsOnly refuses an if whose condition holds a call, through holdsCall, and the Counted row keeps its condition's mutant.
  - id: F8
    status: resolved
    answer: TestPlanLineMutants has rows for else, else-if, init statement, literal-first and shadowed runtime.
  - id: F9
    status: resolved
    answer: TestRenderMutationReportNote has the non-red judged-nothing row and the HEAD match error row.
  - id: F10
    status: resolved
    answer: The summary and the selectWithinCap doc comment now say the cap is sampled evenly across the functions.
  - id: F11
    status: resolved
    answer: MutationReport, MutationReportMatches and the gatecli doc's Mutation report section state the non-Go file bound.
---

### [LOW] A failed check's report loses the findings it had already made

**Location:** `internal/gatecli/mutatecheck.go#diffMutation.run`

**Issue:** `diffMutation.run` returns `mutateOutcome{}` when a package's run fails or the context is cancelled after planning.
`mutationCall.run` then returns the error without assigning the outcome, so `recordFindings` gets only the refresh errors and the pre-check not-judged packages.
The uncovered blocks and verdicts the check had already reached never reach `mutation.json`.
`recordFindings`' doc comment says a failed call records "its partial findings", and the round-1 disposition says partial counts are kept.
Both claims hold for those two lists only.
The report never matches HEAD, so no review reads it, and severity drops to LOW.
This carries the unfixed half of round-1 F2.

### [LOW] The written unjudged and not-judged lists have no assertion

**Location:** `internal/gatecli/mutation_test.go#TestMutationStep_RecordFindings`

**Issue:** `recordFindings` copies `found.outcome.unjudged` into `report.Unjudged` and appends the check's not-judged packages to the pre-check ones.
The new report-step rows of `TestGateMutate` assert the survivors and uncovered blocks only.
`TestMutationStep_RecordFindings` gives neither list.
The tier0 recorder test is the cheapest place to pin both.
This carries the rest of round-1 F5.

The review covered `git diff 5823c67e3bb8bc2e9aab7aed301ffcb08a3f424c..HEAD` and the range's commit messages, against the board entry's three changes and the step.
The diff makes all four: the even spread in `selectWithinCap` and `spreadEvenly`, the log-only and other-platform noise sites in `testcover`, `mutation.json` with its one summary line, and the `mutation_report` marker.
The template lists `mutation` on every key of scope `diff`.
The one `Loosened-Check:` trailer covers the removed `TestLoadConfig` refusal row, which the new behavior replaces with a row that loads `mutation`.
No new BLOCKING finding sits on a changed hunk.

On the focus questions:
- `mutationStep.run` branches on no scoper literal; it asks `Config.HasGoCoverageMaps`, a table keyed by scoper name.
- A failed `mutationCall.run` records no Functions or FileHashes, so `MutationReportMatches` refuses it for any HEAD that changes a function.
- The red-gate, no-hub, non-Go scoper and no-parent branches each have a row; survivors and uncovered blocks are asserted from `mutation.json`, unjudged and not-judged lists are not (F2).
- `gateslot/doc.go` names the step in the holder list and in each bound paragraph, with the even spread.

## Rejected

- The assistant's ask for a new integration row that fails a later package's run is not carried: the round-1 fixer showed no deterministic failure point after a package has produced findings, and the finding is LOW.

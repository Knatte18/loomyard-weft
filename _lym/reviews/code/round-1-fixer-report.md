---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposed change dropped every marked pid with no running tier command among its ancestors. That also drops a tmux server that left the tier's tree, which the cap exists to count. The cap now drops only a pid whose sampled ancestry reaches the lym process without passing a running tier command, and counts a pid whose parentage the sample lost."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

## Changes

- F1: `pattern/PATTERN-test-speed.md` says the redundancy and reach steps judge beside the last tier and write their reports after the lints.
- F2: `newProcessCap` and `request.processCapFor` take the call's running commands, and `processCap.countable` leaves out marked processes that sit below the lym process but below no running tier command. `internal/gatecli/doc.go` describes it. `TestProcessCap` gained a row for a marked process outside every tier command and one for processes below a command or lost from the sample.
- F3: `TestRunTierMemoryGuard` records that the tier command is in the request's running commands while it runs and removed after.
  A self-check follow-up extracted `judgedReports.guard` and added `TestJudgedReportsGuard`, which pins `guard.foreign` for the report steps.
- F4: `TestLastTierStart` pins the start index over literal plans.
- F5: `internal/gateslot/doc.go` lines on the redundancy slot and the gate key's reports.
- F6: `TestReportChain` keeps one scenario, and its doc drops the stopped-chain sentence.
- F7: a `TestGateTest_Scenario` subtest runs a call whose key lists the redundancy report and whose last tier halts it after the chain started.
  It asserts exit 1 and that no report was written.
  The self-check follow-up lets tier0 pass, so the chain starts before the halt.

Gate repair: two comments added for F2, in `internal/gatecli/doc.go` and `internal/gatecli/processcap.go`, wrapped mid-sentence.
Each is now split at a sentence boundary, one sentence per line.

Gate repair, reach report: the report named `reportStepsOf` and `reportSteps.any` as reached by no mapped test.
`TestReportStepsOf` now covers both over literal rows.
`newProcessCap` and `reportChain.stop` stay unreached by a mapped test.
`newProcessCap` is the production constructor of fields the cap's tests stand in for, and `reportChain.stop` only cancels and waits, and the halt scenario of F7 runs it out of process.

Gate: `lym gate test` over `./internal/gatecli` and `./internal/gateslot` at tier0, and `--tier 1` over `./internal/gatecli`, pass.

## Self-check

- fix-code-correctness, MEDIUM: the F7 subtest halted before the chain started. Fixed as proposed: tier0 now passes, so the chain starts beside tier3 before its prebuild fails.
- fix-code-correctness, LOW: `guard.foreign` unpinned. Fixed: extracted `judgedReports.guard` and added `TestJudgedReportsGuard`.
- fix-code-correctness, LOW: a `TestProcessCap` row passes with `countable` removed. Disputed nowhere; no change, the row guards against over-dropping.
- fix-proposal, LOW: `guard.foreign` unpinned. Merged with the finding above and fixed with it.
- fix-proposal, LOW: F7 cannot see a report written after the process exits. No change, since a stopped chain is cancelled before the call returns and the subtest asserts the file is absent after exit.
- fix-proposal, LOW: `reportChain.stop` has no direct test. No change; the F7 subtest now reaches it on the halt path.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.

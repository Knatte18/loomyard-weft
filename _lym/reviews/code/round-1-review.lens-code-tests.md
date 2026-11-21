```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, name the test row that would fail if it broke.

**Per behaviour: the test that pins it, or none**

1. `AppendIntegrationTriage` lists the command-failure logs under their own title, after the flaky list.
   Pinned by `TestAppendSummarySections`, row "integration triage: a verify command failure is listed apart by its first run's log" (exact equality).
2. `AppendIntegrationTriage` is no longer a no-op when only `commandLogs` is non-empty (`&&` replaces the single check).
   Pinned by `TestVerifyGate_FlakyFailurePassesOnRerunWithNote`, row "a verify command that named no failing test", through the `wantSummary` line `` - `/verify/verify-1.log` ``.
3. `AppendIntegrationTriage(dir, nil, nil)` is a no-op.
   Pinned by `TestAppendSummarySections`, row "integration triage: empty is a no-op".
4. `commandFailure` and `splitFlaky` separate opaque failures from test and package failures, and package failures stay with the ids.
   Pinned by `TestTriageWarnings` rows "flaky tests" (a package id `p` stays in the id list) and "a verify command failure is no flaky test". Exact `DeepEqual`.
5. `triageWarnings` gives one warning for the ids and one warning per command log, and nil when there are none.
   Pinned by `TestTriageWarnings` rows "flaky tests", "a verify command failure is no flaky test" and "none".
   No row mixes both kinds, so the order of ids first, then commands, is unpinned. Not a finding: the doc comment states no order.
6. `writeTriageFrictionNote` writes each test identity with its log and fenced tail, a package failure with "Its output was empty.", and a command failure under its own header with its log and tail.
   Pinned by `TestWriteTriageFrictionNote/content` (exact equality over all three kinds together).
7. `writeTriageFrictionNote` leaves out the flaky-test header when every failure is a command failure (`if tests.Len() > 0`).
   **None.** The only all-opaque case, in `TestVerifyGate_FlakyFailurePassesOnRerunWithNote`, checks only `strings.Contains` for three expected lines and never checks that "(flaky):" is absent. `TestWriteTriageFrictionNote/content` always has a test failure. See finding 1.
8. `writeTriageFrictionNote` is a no-op for an empty dir or a nil list.
   Pinned by `TestWriteTriageFrictionNote/empty dir` and the nil subtest at `triage_test.go:87`.
9. `writeFailureOutput` writes a fenced, two-space-indented block, multi-line included, or the empty-output line.
   Pinned by `TestWriteTriageFrictionNote/content` (`"a_test.go:3: boom\nmore"`, and `""`).
   A whitespace-only tail, which the `TrimSpace` branch exists for, has no row. Not a finding: `outputTail` already trims trailing blank lines.
10. The gate records the first run's log (`firstLog := res.Log`, taken before the rerun reassigns `res`).
    Pinned by `TestVerifyGate_FlakyFailurePassesOnRerunWithNote` in both rows: the note and the warning name `/verify/verify-1.log`, and the rerun's log would be `verify-2.log`.
11. `addFlaky` stores each parsed `VerifyFailure` with its tail.
    Pinned by `TestVerifyGate_FlakyFailurePassesOnRerunWithNote`: `wantNote` holds the tail line `      tmuxkit: a stray entry` and `  gate test: failed runs: build (exit 1)`.
12. `addFlaky` dedups by `failure.ID` across gate evaluations (the comparator changed from `have == id` to `have.failure.ID == failure.ID`).
    **None.** Each test drives one gate evaluation, and `ParseVerifyRun` already dedups within one run, so the `found` branch never runs. See finding 2.
13. `flakyFailures` returns a copy and is empty after a hang.
    Pinned by `TestVerifyGate_HungVerifyFailsAtOnceWithoutRerun` (`notes.flakyFailures()` is empty), and by the `Apply` path in finding-free row 11.
14. `Apply` passes `splitFlaky`'s two lists to the summary and the full list to the note and the warnings.
    Pinned by `TestVerifyGate_FlakyFailurePassesOnRerunWithNote` (`DeepEqual` on the warnings, the summary line, the note lines) in both rows.

**Findings**

1. MEDIUM, design. `internal/websterengine/verifygate_test.go:413-416`, the row "a verify command that named no failing test".
   Nothing pins the command-failure-only friction note against listing the failure as flaky.
   This is the defect the board entry reports: `verify-command` in the "(flaky):" list.
   If `writeTriageFrictionNote` wrote the "These identities failed the first verify run … (flaky):" header unconditionally, every test would still pass. `TestWriteTriageFrictionNote/content` always includes a test failure, and this row checks only `strings.Contains` for expected lines.
   Fix: assert that the note does not contain "(flaky):", or add an all-opaque row with exact equality to `TestWriteTriageFrictionNote`.
   Evidence: `internal/websterengine/triage.go:80-83`, `triage_test.go:56-71`, `verifygate_test.go:466-470`.
2. LOW, design. `internal/websterengine/verifygate.go:97-107` (`addFlaky`).
   The changed dedup, which skips a failure whose `failure.ID` is already recorded, is reached by no test that records the same identity twice. `TestVerifyGate_FlakyFailurePassesOnRerunWithNote` drives a single evaluation.
   This fits the mutation report's one uncovered changed block.
   Fix: add an assertion, in an existing verify-gate test with two gate evaluations or as a second evaluation in `TestVerifyGate_FlakyFailurePassesOnRerunWithNote`, that a repeated identity appears once and keeps the first run's log.
   Evidence: `verifygate.go:97-107`; reach report entry for `VerifyGateNotes.addFlaky`.

Tool calls: none. The inherited context held every test and definition this lens needed.

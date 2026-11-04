```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test row that pins it, or none.

**Per behaviour: pinning test**

- Name match with or without extension, and a nested `group/t1`: `TestAcceptWithdraw`, subtest "Accept names a test by its profile file…", rows "a name without/with the file's extension" and "a nested file named by its relative path" (`verbs_test.go`, new).
- Unknown name refused, naming the name and the baseline directory: same table, rows "a typo…" and "an empty baseline refuses every name". These also cover a missing baseline directory.
- A nested file's bare name is refused: row "a nested file's bare name is refused".
- A refusal leaves the scope record unchanged: `requireRefusal`.
- A refusal leaves no kept output: only the main `TestAcceptWithdraw` table asserts it, with `keptOutput == ""`. The new name-check table does not, so the `requireBaselineTest` refusal is never checked for leaving kept output.
- Earlier refusals (wrong row, running, withdrawn by agent, caller without ruling, no scope), same messages: the existing `TestAcceptWithdraw` rows, which the diff only extends with `stageAcceptance`. The shared `acceptingCaller` and `admitAcceptance` run for every row.
- Accept keeps the captured output, clears the accepting copy, and clears `AcceptedProfileHead`: the main `TestAcceptWithdraw` table asserts `keptOutput`, the accepting file gone and the cleared head.
- A second acceptance replaces the first's kept files, including a different extension: subtest "a second acceptance replaces the output the first kept".
- No captured output for the test, or output only of another test, is refused: rows "an acceptance with no captured output…" and "a captured output of another test…".
- `CheckAccept` refuses as `Accept` does and writes nothing: subtest "CheckAccept refuses as Accept does…", with rows for admitted, typo and withdrawn. It asserts the scope record is unchanged but not that the accepting and kept directories are untouched. It has no row for a wrong row or running status; those paths are shared with `Accept` through `acceptingCaller`.
- `Withdraw` drops only that test's kept output: `TestAcceptWithdraw` Withdraw subtest, checking t1 dropped and t2 kept.
- `buildAcceptedProfile`: baseline copied, accepted test's files replaced by kept output, stale profile files removed: `TestDriftVerify`, subtest "the profile is the baseline with the accepted test's kept output in its place". It uses flat files only. A nested path copy and a kept file whose extension differs from the baseline's are not exercised.
- Missing kept output halts Drift-Verify naming the test, with no compare: subtest "an acceptance with no kept output is Stuck…".
- A moved accepted output halts Drift-Verify with ways forward and records no head: subtest "an accepted output that changed after its acceptance is Stuck…".
- Compare count, `AcceptedProfileHead` recorded, and the skip when the head already equals HEAD: the four-row table with `withKept`.
- An agent's acceptance is compared at the later commit against the output it saw: subtest "an agent's acceptance keeps the output it saw…" (profile reads `accepted at aaa`, compare at `bbb`).
- `CompareAccepted` returning an error propagates: none. A minor gap.
- `RunAcceptanceCapture` (timeout, non-zero exit, only another test's file, stale file emptied, passes with the test's file, no capture record, slot released, missing-dir way forward naming `lym drift accept`): `TestRunCapture`'s `acceptanceRows` plus the new missing-dir subtest (`capture_test.go`). A nested test name is not exercised here.
- `siteAcceptanceCapture` and the `missingDirRefusal` case: the missing-dir subtest.
- The `Drift.CompareAccepted` nil seam refused at the shed entry: `TestDriftEntries` row "VerifyNilCompareAccepted". The `DriftVerify.Call` seam check itself is not visibly pinned in the diff.
- Removed `CaptureAccepted` seam: the removed assertions in `driftwiring_integration_test.go` need no replacement.
- Verify agent judging text (`acceptanceKeepsOutput`): static prose, exempt.

**Findings**

1. MEDIUM, `design`, `internal/driftcli/cli.go:149` (`acceptTest`): no test covers the verb's own orchestration.
   - The diff touches no `internal/driftcli` test, and my earlier grep of the driftcli tests for `accept` found nothing.
   - Uncovered behaviours:
     - the order CheckAccept, then capture, then Accept, so a refusal spends no capture;
     - the capture refusal text, "capture of %s's output did not pass … way forward";
     - `scope.Skip` handed to the capture;
     - the new `log` key in the JSON result;
     - no `Accept` call after a failed capture.
   - Evidence: `cli.go:149-184` against `verbs_test.go` and `capture_test.go`, which test the engine only.
2. LOW, `design`, `internal/driftengine/accepted.go:117-136` (`testFiles`): no row for degenerate names.
   - An empty test name matching a top-level dotfile such as `.gitkeep`, and `t1.txt` matching `t1.txt.bak`, have no row.
   - This matters because the focus directive asks about exactly this match rule.
   - Evidence: the name-form rows in `verbs_test.go` use only `t1.txt`, `group/t1.txt` and `t11`.
3. LOW, `design`, `internal/driftengine/verbs_test.go` "CheckAccept refuses… and writes nothing": the writes-nothing claim is only checked on the scope record.
   - It does not check that the accepting and kept directories are unchanged.
   - A `CheckAccept` that moved kept output would pass.

No new test duplicates an existing one that I could name. The renamed `acceptanceRows` replace the old `acceptedRows` rather than adding to them.

Tool calls: none; my context already held every file and diff the lens needed.

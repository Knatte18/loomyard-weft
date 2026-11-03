```yaml
lens: code-tests
outcome: reported
```

Task: per behaviour the diff adds or changes, the test that pins it, or none.

**Behaviour → pinning test**

- `guard` verdict mapping:
  - Pinned by `TestPrePushCheckGuard` (`internal/loomcli/prepush_test.go:316`). Its rows cover skip, skip-unreadable, reconcile error, unreachable, active with nothing moved, active with an own branch moved, owned elsewhere (with and without a subject), stopped by an own branch, stopped by a shared branch, ended after own landing, ended elsewhere, and pause failure.
- `stopReason` filtering of a stopped entry's own branches:
  - Only `diverged` is exercised (row `StoppedOwnBranch`).
  - No row has several branches, or a mix of stopping and non-stopping states (`level`, `ahead`, `fast_forward`) among the own branches. The exclusion condition at `prepush.go:254` is unpinned.
  - No row has a report entry with another slug, which the `entry.Slug != c.slug` filter at `prepush.go:232` ignores.
- `landed` returning an error, read as "not holding":
  - The stub at `prepush_test.go:421` always returns a nil error, so the warn-and-stop path at `prepush.go:243-246` has no row. **None.**
- `movedBranches` joining several moved own branches:
  - The `ActiveOwnBranchMoved` row has one own moved branch and one foreign moved branch. It pins the own-slug filter, but not the `", "` join of several.
- `post` swallowing a failed notify:
  - The stub notify always returns nil, so a failed post is unpinned. **None.**
- The `skip` closure in `newPrePushCheck` (`prepush.go:148-164`):
  - It covers pushes off, the prime, `pauseCovered`, and the Finalize producer reading the status file. The test replaces `skip` wholesale. **None.** This is the only gate that keeps the check off Finalize, a paused run and the prime.
- The other `newPrePushCheck` seams (`reconcile`, `landed`, `pauseOwned`, `pause`, `notify`):
  - They are thin delegations to existing helpers. **None.**
- `loomCommitStatusDeps` `Spawn` stopped by the check:
  - It starts no push and returns nil, with commits left local (`wiring.go:214-217`).
  - The integration tests that use `loomCommitStatusDeps` (`wiring_commitstatus_integration_test.go:47`, `283`, `349`) do not call the stopped path. The file has no `guard`, `errPushStopped` or `push_stopped` reference. **None.**
- `loomCommitStatusDeps` `Push` returning the stop error (`wiring.go:225-227`): **None.**
- `commitRecordsVerb` `push_stopped` envelope (`commitrecords.go:51-52`):
  - The new ok envelope with `committed: true` and `push_stopped` has no assertion. The grep found no test mentioning `push_stopped`. **None.**
- `pullRequestDeps` `PushTask` passing the check first (`landingdeps.go:339-342`):
  - `watchfilechanges_test.go:135` only reads `RecordChanges`. **None.**
  - `landingshed`'s `PushTask` tests use stubs, so they do not reach this wiring.
- `PausePair` (`tendercli/deps.go:215`):
  - `cli_test.go:210` covers `PauseOwnedElsewhere` only. **None.**
  - No test checks that `PausePair` writes a plain entry with the default caller, or that it adds none when an entry already stands.
- Rename `ownerPauseGrace` to `pauseGrace`:
  - `TestPauseOwnedElsewhere_PausesTheSlugOnceUnderTheOwnerCaller` (`cli_test.go:210`) covers it. No behaviour change.

**Findings**

1. MEDIUM, design, `internal/loomcli/wiring.go:213-230` and `internal/loomcli/commitrecords.go:51`.
   - No test pins that a stopped check leaves `Spawn` returning nil with no push started, that `Push` returns the stop error, or that `commit-records` answers ok with `push_stopped`.
   - Evidence: `prepush_test.go` drives `guard()` only, and the integration file never drives a stop. A test of `commitRecordsVerb` over stub `commitStatusDeps` returning a wrapped `errPushStopped` would pin the envelope. A test of `newCommitStatusSeam` over a stubbed `Spawn` would pin the nil-return.
   - These dispositions are what makes the stop visible to callers.

2. MEDIUM, design, `internal/loomcli/prepush.go:148-164`.
   - The `skip` closure has no test: pushes off, prime, pause-covered and Finalize.
   - Evidence: `prepush_test.go:416` stubs `skip` with a bool. The Finalize and pause exemptions are the safety valve against pausing a run mid-landing, so a regression would go unnoticed.

3. LOW, design, `internal/loomcli/landingdeps.go:339-342`.
   - PR-Review's `PushTask` wiring of the guard has no test.
   - Evidence: no test references `PushTask` of `pullRequestDeps`. Every `landingshed` test sets its own stub.

4. LOW, design, `internal/tendercli/deps.go:215`.
   - `PausePair` has no test.
   - Evidence: `cli_test.go:210-246` covers only `PauseOwnedElsewhere`. It is a copy of that pause call minus the caller and the end step.

5. LOW, design, `internal/loomcli/prepush.go:243-246` and `:253-257`.
   - Three `guard` paths are missing as table rows, which would be cheap additions:
     - the `landed` error read as not holding;
     - a stopped entry with mixed own-branch states, which checks that `level`, `ahead` and `fast_forward` are left out of the naming;
     - an entry for another slug in the report.
   - Evidence: `prepush_test.go:343-408` has none of these.

**Tool calls**
1. `grep -rln "push_stopped|PausePair|loomCommitStatusDeps|pullRequestDeps|newPrePushCheck" internal --include=*_test.go`: to find whether any other test touches the new seams.
2. `grep -n ... wiring_commitstatus_integration_test.go watchfilechanges_test.go`: to confirm those two files never drive the stop path.

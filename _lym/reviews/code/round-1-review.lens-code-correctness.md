```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the changed functions and branches of the diff.

**Cases run (no tool calls were needed):**

| Function or branch | Case run | Result |
|---|---|---|
| `noFileReason` (`capture.go:141`) | `skip` nil or empty | Correct. It returns the old `exit 0 but wrote no file to <dir>` reason. |
| `noFileReason` | `skip` of 1 or 2 entries | Correct. It names "scope partial skipped 1 test" or "2 tests" and "every test the capture command ran may have been skipped". |
| `noFileReason` | `skip` containing duplicates, e.g. `["a","a"]` | Reports 2, though `SkipList` and the harness see 1 distinct test. This is the NIT below. |
| `RunCapture` default branch (`capture.go:130-134`) | exit 0 with no file | Correct. It sets `State`, `Reason` and `WroteNoFile` together. |
| `RunCapture` other failure paths | timeout, non-zero exit, `holdsFile` error, `failCapture` | Correct. `WroteNoFile` stays false. `rec` is rebuilt per call, so the flag never carries over from an earlier record. |
| `RunCapture` skip list | `req.Skip` against what the callers pass | Same list. `driftcli/cli.go:408` and `loomcli/driftwiring.go:155` both set `scope.Skip`, the list `rec.Skip` and the skip file hold. `RecordScope` refuses `--skip` outside scope partial, so "scope partial" is accurate whenever `len(skip) > 0`. |
| `emptyUnderSkips` (`capture.go:154`) | pre-change record with no `wrote_no_file` key | False, so it gets the plain capture way forward. No wrong rescope Stuck. |
| `emptyUnderSkips` | `WroteNoFile` true with `Skip` nil | False (a no-skip capture), so it gets the plain way forward. Correct. |
| `emptyUnderSkips` | read-back through `state.ReadJSON` and rewrite | The new field is read and written like the others. No field is lost. |
| `DriftAwait.await` (`await.go:78`) | no capture record | `rescope` is false (guarded by `captureFound`). Correct. |
| `DriftAwait.await` | failed record, empty under skips, HEAD on the branch point | Stuck lists scope full or a shorter `--skip` list, scope off, and defer, with no plain capture. Correct. |
| `DriftAwait.await` | same record, HEAD moved | `onBranchPoint` is false, so only scope off and defer are offered. Correct, since no capture may run on changed code. |
| `DriftAwait.await` | running record, passed record, or passed record at another HEAD | `WroteNoFile` is never set on these, so `rescope` is false. Correct. |
| `awaitWaysForward` and `rescopeWaysForward` (`await.go:113-132`) | each flag combination | Correct. `slices.Insert(ways, 0, …)` puts the scope-and-capture way first, and `, or ` joins the ways. |
| Scope-off way forward at Await | `RecordScope` as parent at a blocked Await, then `lym loom resume` | Admitted (`verbs.go:67`), and the drop is skipped for `ScopeOff` (`verbs.go:91`). Await then returns Done on `scope.Scope == ScopeOff`. Correct. |
| Scope-full way forward | `--scope full` (empty skip) at a blocked Await or Baseline | `dropCaptureSkipping` removes the capture record, because the old `capture.Skip` holds tests the new `[]` lacks. The following `lym drift capture` is admitted at both rows. Correct. |
| Shorter `--skip` way forward | new list a strict subset of `capture.Skip` | The record is dropped. Correct. |
| `captureAgain` (`baseline.go:221-226`) | `RunCapture` returns the empty-under-skips record | Hard-coded `rescopeWaysForward(true)` is sound. `RunCapture` returns that record only after `req.Check` passed under the lock, so HEAD was on the branch point. |
| `captureAgain` | `RunCapture` returns an error (lock held, `Check` refusal) | Returned as an error before the new branch. Unchanged. |
| `captureAgain` | failed record from `failCapture`, or a timeout or non-zero exit | `WroteNoFile` is false, so the plain way forward is kept. The `Reason` format is unchanged. |
| `captureAgain` Stuck | `CaptureHaltPrefix` reason at a blocked Drift-Baseline | `RecordScope` admits the parent there (`verbs.go:67`), and `Defer` admits the prefixed halt (`verbs.go:190`). |

**Findings**

1. NIT, class `design`, `internal/driftengine/capture.go:150`. The skipped-test count is `len(skip)` of the raw `scope.Skip`. `RecordScope` (`verbs.go:74-96`) does not deduplicate `--skip`, so a repeated test overcounts, while `SkipList` (`compare.go:114-126`) does deduplicate. The reason can then say "skipped 3 tests" for 2 distinct ones. Evidence: `capture.go:133,150`, `verbs.go:96`, `compare.go:116-122`.

No other correctness defect found in the changed code.

**Tool calls:** none. Every read this lens needed was already in the inherited context.

```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour, the pinning test**

1. Code side, a rejection outlasts the backoff and `HasUnpulled` is false, so the record is `other`, there is one mail, the mail names no divergence, and the error satisfies `IsPushRejected`.
   - Pinned by `TestPushFailureClasses/ARejectionThatOutlastsTheBackoff…/Code` (`pushclasses_integration_test.go`).
   - It is also asserted in `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` (`pushanchored_integration_test.go:367`).
2. Records side, same behaviour: pinned by the `Records` row.
3. Bolt side, same behaviour: pinned by the `Bolt` row.
   - `TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict` (`boltpullfirst_integration_test.go:619`) also expects `other`.
4. D3, the record error quotes git's refusal line: pinned by `strings.Contains(record.Error, "(pre-receive hook declined)")` in all three rows.
   - This goes through `PushRebaseFree`'s new wrapping and `lastLineHolding` with `rebaseRetryTriggers`.
5. A real divergence on the code side (origin holds a commit the branch lacks) records `not_fast_forward`.
   - This path now runs through `failRejection`'s `diverged` branch.
   - The unchanged code-divergence rows in `TestPushFailureClasses` pin it.
   - `TestPushPairAnchored_…DivergedCodeSide` (`pushanchored_integration_test.go:403`) covers the same case.
6. A replay conflict or destructive refusal stays `not_fast_forward` with paths: unchanged and pinned by the existing records and bolt replay-conflict rows. They never reach `failRejection`.
7. `failRejection` where `Fetch` fails, recording `other` with the fetch error joined: **no test.**
8. `failRejection` where `HasUnpulled` errors, recording `other`: **no test.**
9. The `failRejection` `diverged` branch on the records and bolt sides: **no test.**
   - A bare rejection that outlasts the backoff with origin ahead is not exercised on those sides.
   - They are covered only through the shared code path, which the code-side divergence test reaches. I rate this LOW.
10. The `other` way forward is a plain retry of the side's push step: **no test.**
    - The rows only assert that the mail lacks "holds commits this branch lacks" and "git merge".
    - Nothing asserts the retry advice is present.
    - `FailedPushWayForward` for `other` is not changed by the diff.
11. D3, the sentinel text no longer holds `remote diverged`: **no direct test.**
    - The only text-level change is `commitrecords_test.go:97`.
    - It now matches the substring `push rejected`, which the old text `gitrepo: push rejected (remote diverged)` also contained, so the row would pass if the text were reverted.
12. `PushRebaseFree` returns a wrapped sentinel: pinned by `push_test.go:255`, which checks `errors.Is`.
    - `push_test.go` does not check that the line is appended.
    - Behaviour 4 covers that through the fabricengine row.
13. The auth line now goes through `lastLineHolding` (refactored from `authRefusalLine`): pinned by the existing `push_test.go:313` auth case.
14. D2, exactly one mail: `len(fake.lines) != 1` is asserted in all three rows.
15. The old subtests "ARecordsRetryWhoseSecondPush…" and "ACodeRetryWhoseEveryPush…" were replaced by the table.
    - Their `not_fast_forward` assertions, and the `len(record.Commits)` check for the records row, are gone.
    - No `not_fast_forward` declining-hook case remains, which is correct under D1.
    - I found no duplicate of the new rows.
    - Item 1's secondary pins re-assert `other` for a declining hook on the code side and the bolt replay-drop. Neither adds a distinct behaviour. I consider this a NIT at most.

**Findings**

- **F-a — MEDIUM, consistency (coverage gap).**
  - Location: `internal/fabricengine/pushclassified.go:324-337` (`failRejection`); test file `pushclasses_integration_test.go`.
  - Summary: the fetch-failure and `HasUnpulled`-error branches of `failRejection` have no test row. Both are documented in the D1 Open risks and in the `doc.go` text ("a failed fetch … is an `other` failure"), and the summary claims them as behaviour.
  - Evidence: the diff adds no test that makes `Fetch` fail after the backoff.
  - The fix would be one row, for example removing or breaking the remote after the final push attempt, asserting `other` with the fetch error in `record.Error`.

- **F-b — LOW, consistency.**
  - Location: `internal/loomcli/commitrecords_test.go:97`.
  - Summary: the changed assertion `"push rejected"` also matches the old sentinel text, so the D3 criterion "text no longer holds `remote diverged`" is pinned nowhere.
  - Evidence: line 97 of the diff.
  - A `!strings.Contains(…, "remote diverged")` assertion on the `PushRebaseFree` rejection error in `push_test.go:255`, or in the new table, would pin it.

- **F-c — LOW, consistency.**
  - Location: `pushclasses_integration_test.go`, the new table.
  - Summary: no row asserts that the `other` way forward names a plain retry. It asserts only the absence of divergence words.
  - Evidence: the mail check in the new table.

- **F-d — LOW, scope.** Item 9 above: the `diverged` branch of `failRejection` has no records-side or bolt-side row.

**Tool calls**

1. `git diff … -- internal/fabricengine/pushclasses_integration_test.go`. The test diff for the main changed test file was not in my context.

```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test that would fail if it broke.

**Per behaviour, the test that pins it:**
1. `Topology.Remove` copies `rerun` into the teardown job and the pending record: `TestRemove_PendingRecordNamesTheRerunItsCallerGave` (`remoteteardown_rerun_integration_test.go:77`). It asserts the listed way forward ends in the caller's command.
2. `rerunWayForward` names `job.Rerun` for a removal that carries one: `TestPendingTeardowns`, row `cancelled` (`pendingteardown_test.go:40,58`).
3. `recordsArchiveWayForward` drops the "when origin still holds" condition when `Rerun` is set: `TestRemove_PendingRecordNamesTheRerunItsCallerGave` (`:94-95`), whose expected text carries no condition.
4. `recordsArchiveWayForward` keeps the condition, and `lym fabric remove`, when `Rerun` is empty: `TestRemove_RerunAfterItsTeardownChildEndedArchivesTheLocalRecordsTip` (`remoteteardown_rerun_integration_test.go:58`) and `TestPendingTeardowns` row `ended`.
5. The failed-step notices (weft delete, task delete, records-root lookup) name `job.Rerun` for a removal: no row. `TestNoticeRemoteTeardown` holds only `Rerun`-less remove jobs (`remoteteardown_test.go:45-48`). The branch itself is pinned by behaviour 2 through the same function, so no finding.
6. The held teardown lock's refusal from `Remove` names `re-run \`<rerun>\``: none.
7. `pairteardown.New`'s remove closure passes `req.Rerun` to `Topology.Remove` (`teardown.go:247`): none.
8. `removePairAndBranch` sets `Rerun: cancelCommand(slug, true)`: `TestCancelRun` (`cancel_test.go:492-493`).
9. The chokepoint guard counts five arguments: `TestEnforcement_PairTeardownScanIsNotVacuous` (`teardown_enforcement_test.go:91`).
10. The call-site changes (`""` added): signature-forced, so no behaviour to pin.

**Findings:**

- **MEDIUM** · design · `internal/fabricengine/remove.go:275-278`
  No test pins the held-lock refusal of `Remove` naming the caller's command.
  Evidence: `teardownlock_integration_test.go:355-358` covers only the discard's given rerun. Every `Remove` call in `teardownlock_integration_test.go` (`:105`, `:134`) passes `""`. `remoteteardown_childhold_integration_test.go:84` drives `spawnTeardownHold` directly, not `removePair`'s `lockRerun` choice.
  Mutating the `if rerun != ""` branch survives every test.
  Fix: add a row beside `DiscardWhoseWaitRunsOutNamesTheGivenRerun` that holds the lock and calls `Remove(..., "lym batten cancel <slug> --remove")`. It asserts the refusal names that command and not `lym fabric remove`.

- **LOW** · design · `internal/pairteardown/teardown.go:247`
  No test pins that `pairteardown` forwards `Request.Rerun` to `Topology.Remove`.
  Evidence: `TestCancelRun` checks only the request the cancel hands its fake `RemovePair` (`cancel_test.go:492-493`). The `pairteardown` tests build requests without `Rerun` (`teardown_test.go:338`). The reach report lists `pairteardown.New` tests, none of which passes a `Rerun`.
  Dropping `req.Rerun` at `teardown.go:247` survives.
  Fix: add an assertion in an existing `pairteardown` test of the remove path that a request with `Rerun` set reaches the removal. A tier-1 integration through `New` would do, or extend an existing one.

- **LOW** · design · `internal/fabricengine/pendingteardown_test.go:40`
  The new `TestPendingTeardowns` row carries no `WeftTip`, so it never reaches the condition-dropping branch of `recordsArchiveWayForward`. That branch is pinned only by the tier-1 integration test (behaviour 3).
  Evidence: `pendingteardown_test.go:40` has `{Kind: remoteTeardownRemove, Slug: "cancelled", Rerun: ...}` with no `WeftTip`. `remoteteardown.go:243-244` returns before the condition when `WeftTip` is empty.
  Fix: give the row, or a sibling row, a `WeftTip` and `Worktree`, so a tier-0 test pins that a `Rerun`-carrying removal gets no "when origin still holds" prefix.

Not a finding: the integration test and the `TestPendingTeardowns` row overlap only on the trailing re-run text. The integration test also pins that the job carries `Rerun` and that the condition is dropped, so neither duplicates the other.

**Tool calls:**
1. grep of the remove-teardown test files for `Rerun` and `batten cancel`: to learn whether a lock-refusal or notice row passes a caller's rerun, which my context lacked.
2. grep of `TestNoticeRemoteTeardown` rows and the `pairteardown` `RemovePair` tests: to learn whether any remove-job notice row or `pairteardown` request carries `Rerun`, which my context lacked.

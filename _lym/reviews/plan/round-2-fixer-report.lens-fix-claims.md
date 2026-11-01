```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every claim the fix added or rewrote against the code at HEAD.

No findings.

Claims checked:
- **Card 4:** `cancelRun` takes `cancelDeps`, defined at `internal/battencli/cancel.go:29`. `TestCancelRun` already drives `cancelRun` over `fake.deps()` (`internal/battencli/cancel_test.go:830`), so its fakes can drive `cancelJudgingEndOnly` with the same deps. `cancelRun` returns `report, nil` for an `EndOnly` report (`internal/battencli/cancel.go:200`-`203`), so the composition's nil-error branch is where the end-only error belongs.
- **Card 11:** `reconcileRecordsBranch` sets `BranchAhead` with `Moved = true` after a successful replay whose head is not origin's (`internal/fabricengine/originreconcilemoves.go:226`-`232`). The doc comment at line 168 says the replayed commits sit on origin's head, which matches the new sentence. A records branch that was already ahead returns `BranchAhead` with `Moved` unset (lines 207-212), so it still warns, as the reworded D2 says. Code-branch moves go through `ffWorktree` and set `Moved` only for `BranchLevel`, so the new row is a records-branch case, as written. The `BranchOutcome` doc comment (`internal/fabricengine/originreconcile.go:163`) says `CodeWorktree` is where every fabric verb of a records branch runs, and line 737 sets it from the target.
- **Card 13:** `internal/adoptcli/check.go:226` posts `way forward: fix it, then run %s in the prime` for the failed-tier case. The same line also serves the before-any-tier case, which the card's no-verdict sentence already covers. The new sentence is scoped to a failed tier, so the two do not conflict.
- **Card 12:** `docs/overview.md:673` holds the batten-paragraph sentence that `lym adopt` starts a run's first pass detached, beside `lym beam land`. The added sentence names it correctly.

Tool calls:
1. `internal/fabricengine/originreconcilemoves.go:160`-`240` and `:395`-`452`: the parent's context showed only the grep hits for `Moved` and `BranchAhead`, not the replay branch that sets them together.
2. Grep for `cancelDeps`, `newCancelFake`, `cancelRun(` call sites and the `CodeWorktree` assignments: needed to confirm the test fakes produce `cancelDeps` and that `CodeWorktree` is filled for records targets, which the parent's context did not show.

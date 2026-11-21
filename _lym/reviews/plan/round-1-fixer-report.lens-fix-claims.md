```yaml
lens: fix-claims
outcome: reported
```

**fix-claims lens:** checking that every claim the fix added or rewrote holds against the code at HEAD.

**MEDIUM** — card 2 (was card 3), the `pushJob.push` unseeded-board guard bullet: "refuses, before any push … The refusal names the way forward: run the same `lym fabric clone --single-repo` again with `--reset`." Card 4's CLI paragraph and card 5's "unseeded board" row repeat the claim.
- **What's wrong:** the code does not give that refusal a single, clean way forward. A refusal returned from `pushJob.push` falls through `pushClassified` into the generic failure path. That path records a failed board push, and the returned error carries a second, generic way forward after the guard's own.
- **Evidence:**
  - `internal/fabricengine/pushclassified.go:140-143`: a board with no upstream reads as having unpushed commits, so `job.push()` runs.
  - `:166`: a refusal that matches no class goes to `return job.fail(PushFailureOther, pushErr, …)`.
  - `:387-388`: `fail` writes a failed-push record.
  - `:58-59`: `PushFailedError.Error()` appends `"; way forward: " + FailedPushWayForward(e.Record)`.
- **Consequences:**
  - The leftover failed-push record makes the next push or reconcile retry the board push, which is refused again. That conflicts with "never pushes an unseeded board unleased" only in wording, not in safety.
  - The card 5 row that asserts the refusal "naming the same clone again with `--reset`" still passes, but against a message that holds two ways forward.
- **Fix:** the card should either classify this refusal, for example as a `*destructiveRefusal`, which `classifyFailure` treats specially at `:415-417`, or place the guard in `pushClassified` before `job.push()`. It should then say whether a failed-push record is written.

**Claims that hold:**
- A joined board adopts `origin/main` with `--track`, so its upstream is set (`boardweft.go`, the `worktree add --track -b` form).
- Card 3's "Fifteen" matches its Edit list.
- `remote_integration_test.go` is a tier1 file with no tier0 requirement.
- `recordPush` writes `branch.<b>.remote` and `branch.<b>.merge`, so "the two keys a two-repository clone's first board push writes" is correct.
- `Fabric.PushBranch` goes through `PushWarpRebaseFreeAt`. That fits the records-replay row, but I did not trace it all the way to `pushanchored.go:207`.

Tool calls:
1. `boardweft.go`, `pushClassified` and the `PushBranch` lookup: my context did not have the board worktree's upstream setting or how a refusal is classified.
2. `pushanchored.go` and `pushclassified.go:120-170`: I needed the full classification path the refusal takes.
3. `pushJob.fail` and `PushFailedError.Error`: I needed to see how a failed push is recorded and how its message is rendered.
4. `fabric.go` `PushBranch`, the test file's tier, and `FailedPushWayForward`: to confirm the records-replay row and the tier claim. The `FailedPushWayForward` grep returned nothing.

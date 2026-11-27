```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff to `TestBattenIntegration_Rows` and the new `rowStep` type.

**Functions and branches, the case I ran, and the verdict**

- **`rowStep` type** (`internal/battencli/lifecycle_integration_test.go:337`): a step with `run == nil`, as `posixOnly` gives on Windows. The loop skips it before `t.Run`. Correct.
- **The group loop's closure capture** (`:411-424`): `group` and `step` are used inside the parallel `t.Run` closures. `go.mod` declares `go 1.26`, so each iteration gets its own variables and every group runs its own steps. Correct.
- **The group subtest's `t.Parallel` and `CopyHub`** (`:413-414`): each group pauses, then copies its own hub. That hub is `t`-scoped, so its cleanup runs when the group ends, after its sequential steps. `CopyHub` already ran under a top-level `t.Parallel`, so a parallel caller is not new. Correct.
- **The failure branch** (`:419-421`): a step that fails returns `false` from `t.Run`, and the group closure returns. Later steps of that group are not generated, and the other groups go on. The failure reaches the parent through the subtest. Correct, and it matches the doc comment's "A failed step ends its group".
- **A group of nil steps only** (CommandRows on Windows): the group copies a hub and runs no step. That costs a hub copy and gives no wrong result. Correct.
- **Process-wide state across groups.**
  - `pairPushCounts` is mutex-guarded and keyed by worktree path, and each hub's paths differ. Correct.
  - `spawnPairPush` is set once in `TestMain`. Correct.
  - `logcapture.Capture` in `stepCommandRows_IgnoreACommandSetInTheTasksTrackedLayer` swaps the process-wide logger output. Other groups' log lines land in its buffer while it runs. The assertion is a `Contains` on a warning that only that step's tracked layer produces, so extra lines cannot falsify it. The parallel tests of the package hold no second concurrent `Capture`: `TestSettleCodeTip` and `TestLoadOverviewThresholds` are serial. No wrong result. Whether the rule on `t.Parallel` plus process-global state holds is outside this lens.
  - The rest is per hub: board `batten.yaml`, the pre-receive hook on `RecordsBare`, the records `origin` URL, orch state and the prime's README. Correct.
- **Each step on a fresh hub.**
  - SeedChild: the two steps that commit prime records run last. Correct.
  - FourRowRun: its Land runs over a fresh prime at its remote head. Correct.
  - TeardownRemote: the pre-receive hook is restored before UnreachableRemote, and UnreachableRemote restores the URL on success. On a failure the group ends anyway. Correct.
  - TeardownLocal: MidListResume and AwaitingChildDone each run Land and Teardown. On the old hub these ran after a SiblingDirt pair had been left in place, and they still do. Correct.
  - RunShed: MarkWatched leaves a strand in the prime's orch state ahead of StopReport, TaskWorktreeSeams and Status, as in the old order. Correct.
  - CreateRow: DirtyPrime runs last. Correct.
  - CommandRows: board commands are reset in cleanup. Correct.
  - No step relies on another group's leftovers for its setup or for an assertion to mean something.
- **Every old step kept once:** the old table has 36 entries. The new one has 6 + 1 + 3 + 6 + 7 + 7 + 6 = 36, and each name maps to a single entry. Correct.
- **The doc comment's ordering claim** (`:346-347`): "A step that leaves the hub changed for good runs last in its group" is false as written (finding below).

**Findings**

1. **LOW**, class `consistency`, location `internal/battencli/lifecycle_integration_test.go:347`.
   The doc comment says a step that leaves the hub changed for good runs last in its group, but several such steps run mid-group:
   - `stepSeedChild_IgnoresPrimesCommittedBattenRecords` commits batten records on prime (`recordleak_integration_test.go:53-60`) and is followed by `RealReadStatus_IgnoresPrimesCommittedStatusForTheSameSlug` (`:364-365`).
   - `stepMarkWatched_…` leaves the orch strand recorded on prime (`:1823`) and is followed by three RunShed steps (`:387-390`).
   - `stepCreateRow_LeftoverBranchIsRewordedForPrime` leaves a prime branch (`:1125`).
   - `stepTeardown_SiblingDirtOutsideRecordPathsRefusesShutdown` leaves a dirty pair in place (`:1708-1731`).

   No step's result is wrong. The rule as stated is broader than the table keeps, so it could mislead a later regrouping.
   - Fix: narrow the sentence to a step that leaves the hub changed in a way later steps would trip on, such as the dirty prime.

No `design` finding: every branch and failure path of the changed code gives the right result.

Tool calls:
1. `grep go.mod` for the Go version, because closure capture in the parallel loop depends on per-iteration loop variables, which my context did not show.

```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the proposals the fixer carried out: F15, the F16 ruling, F17 and F18. I found one finding.

**MEDIUM — card 12, "Support for the moved tests", the `pushkit` bullet (F18's "fabric's copies go with their callers retargeted")**
- **Problem:** Carrying out the proposal makes card 12 change three test files that belong to other concerns. That breaks the artifact's own D9 criterion, and the card never says why the change is allowed anyway.
- **The case:** fabric's `requireOneFailedPush` and `requireNoFailedPush` are deleted from `pushclasses_integration_test.go`. Their callers outside this concern must then be edited:
  - `landing_integration_test.go`, which is part 4 (landing);
  - `boltpullfirst_integration_test.go`, which is part 5 (pull);
  - `rejectedtakeover_integration_test.go`, which is part 4 (takeover).
- **Evidence:**
  - Acceptance criterion: "each part's diff changes in `internal/fabricengine` only its own concern's files besides the executor and shared types, in D9's order (D9)". It makes no exception for test files.
  - Card 12 itself moves `plantRunTransition` into a new helper file "so no other concern's file changes for it". So the card treats an edit to another concern's file as something to avoid.
  - The changed text only says "that retarget is a call-site rename and no row changes". That describes the edit; it does not reconcile it with the criterion.
  - Seen in the earlier grep: `requireOneFailedPush` is used in `boltpullfirst_integration_test.go` and `landing_integration_test.go`. `requireNoFailedPush` is used in `rejectedtakeover_integration_test.go` and `landing_integration_test.go`.
- **What would settle it:** Either the parent records that a helper retarget is exempt from D9's file rule, or those three files keep a local helper until their own part moves.

**Cases I built that turned out not to be defects:**
- **F17:** `writeRunStatus` has no state argument, so it writes `State` as `""` where `statusJSON` wrote `"running"`. The replay's rule only checks `status.State == "done"` (`replay.go:309`), so both values read the same.
- **F17:** The prime record is written at `shedrun.StatusFile(prime)`, in the code worktree. That worktree's `_lym` is a junction (`junction.go:28`), so committing the file in the records worktree still works.
- **F15:** The other in-package hits do not reach the new refusal. `teardownlock_test.go` calls `DetachedPush` on worktrees that do not exist, and `DetachedPush` returns nil before `CoalescePushRequest` (`teardownlock.go:122-130`). `coalesce_test.go` only drives the `coalescePush` loop.
- **F16:** Today an error from a first push or a replay push that names no class records `other` (`classifyFailure`'s default, then `pushClassified:184`). Mapping it to `OutcomeProtectedUnrejected`, which also records `other`, keeps that.

Tool calls, each for something my context lacked:
1. Grepped `replay.go` for `State` use, and `hubforge` and `junction.go` for the `_lym` junction: needed to test F17's state field and record path.
2. Grepped the package-`fabricengine` hits other than `pushdeferral` for calls to coalesce or push entry points: needed to test whether F15's set is complete.
3. Read `teardownlock_test.go:25-60` and grepped `DetachedPush`: needed to see whether the `ParentLocked` row reaches `coalescePushSides`.
4. Read `teardownlock.go:95-135`: needed to confirm the gone-worktree return comes before `CoalescePushRequest`.

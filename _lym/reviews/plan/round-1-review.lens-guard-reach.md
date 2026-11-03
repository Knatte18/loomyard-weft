```yaml
lens: guard-reach
outcome: reported
```

I checked every guard, refusal, branch and test row the four cards add. Each one can be reached, and none guards a site that recurs on an unguarded path. One test row describes a state that cannot occur (finding F1).

**Per guard, reachable or not**

Card 1:
- **`refuseWindowMove` order-change refusal: reachable.** `UpdateStrand` → `updateStrandLocked` passes `validateDisplay` (an overview with no group, then `render.ValidateDisplay`, which checks only the own-window size; `render/policy.go:10-15`). The anchor-move check passes, then the new order check fires. The tier0 row `OverviewOrderChangeRejected` reaches it.
  - `UpdateStrand` has no production caller; the grep found none. So the input arrives only through the engine API, but it is named and a test reaches it.
  - The guarded site does not recur unguarded:
    - An IfAbsent relaunch keeps the stored `Display` and never adopts the spec's (`strand.go:608-631`), so a relaunch cannot change the order.
    - `ReplaceStrand` changes the order through remove and add, which D1 licenses.
- **`OverviewFocusOnlyChangeAccepted`: reachable.** Same order, so `refuseWindowMove` passes. The strand is not hidden and needs no surface launch.
- **The removed second-overview refusal in `addStrandLocked`: removal is sound.** `ReplaceStrand`'s "third strand" refusal had no other path; it reached only through `addStrandLocked` (`replace.go:636`). The name uniqueness the stated bound rests on stays in `strandNameLocked` (`strand.go:216-235`).
- **`overviewWindowInsertArgs` branches: all reachable.**
  - `-a`: some listed window's key is not higher than the new one.
  - `-b`: no listed window qualifies, including a nil keys map.
  - Nothing: no window listed.
  - The Resume replay after a server restart goes through the same `openOwnWindowLocked`. The old pane ids are missing from the pane map, so the keys come out empty and each overview opens `-b` in strand-table order. Two strands with distinct keys still end in key order whichever replays first, so no unguarded placement path remains.
- **`overviewWindowKeys` exclusions: all reachable.**
  - Strand with no pane: the strand being added is appended with an empty `PaneID` before launch (`strand.go:390-404`).
  - Pane the map lacks: the pane-listing failure path, or a pane in another session.
  - Non-overview strand: every own-window strand.
- **Relaunch "old window carries the strand's own key": reachable.** `launchStrandLocked` runs with the strand's old `PaneID` still bound, which `previousWindow = windowByPane[s.PaneID]` already relies on (`spawn.go:261-270`).
- **Tier2 rows:** each reaches its claim, apart from the relaunch row's command (F1).

Card 2:
- **`relevantWindow` lowest-index overview fallback: reachable** from attach, `FollowRun` and `RemoveStrand` → `killStrandPanes(selectOnEnd=true)` → `selectRelevantWindowLocked`. On a `ReplaceStrand` that ends an overview, `selectOnEnd` is false and the new window is selected directly (`replace.go:625-654`), which D1 intends.
- **D2's "pane exit" half: the state cannot occur, and card 2 says so correctly.** `planReconcile` never kills a present own-window pane, alive or dead (`reconcile.go` own-window branch, "A present own-window pane keeps its binding, alive or dead, and is never killed here"). No test is owed.
- **`followTargetWindow` lowest-index marker fallback: reachable.** The `TestFollowSessionVia` row lists the lower index first, so it tells the new rule from the old one.
- **`relevantWindowFixture` and the renamed attach row: reachable.** Panes are listed `@4` then `@3`, so the old last-listed rule would answer `@3` and fail.
- **Tier2 `WithNoLiveAgentEndingTheViewedLowestOverviewSelectsTheNextLowest`: reachable.** The existing last subtest removes every own-window strand (`windoworder_integration_test.go`, `RemovingTheViewedAgent…`), and card 1's agent is removed, so no live agent is left. `RemoveStrand` takes the strand out of state before the select, so `third` (the lower index) wins over `pawl`.

Card 3:
- **The `ensureRunsStrand` add beside `loom-status`: reachable.** The row's "no replace and no removal" assertion is real: `fakeStrands` records `remove` calls (`internal/orchcli/cli_test.go:46-76`).
- **The removed `loomStatus` replace branch:** its guarded state, reed refusing a second overview, no longer occurs once card 1 lands, so dropping it leaves nothing unguarded.

Card 4:
- **`ensureStatusStrand` replace path in the new smoke subtest: reachable.** A sidecar naming another build fails `StrandCurrent` and sends the strand to replace (`bootstrap.go:226-235`). A refused replace only warns, but the row asserts a different guid, so a failure cannot pass silently.
- **`statusStrandAddSpec`:** after the change it has no branch, so there is no guard to reach.

**Findings**

- **F1** (NIT, consistency). Location: card 1 (`01-reed-overview-order-key.md`), tier2 subtest `RelaunchingOrReplacingEitherOverviewKeepsTheOrder`.
  - The row says it relaunches `pawl` "with `AddStrand` `IfAbsent` and cmd `sleep 300`". The IfAbsent relaunch branch ignores the spec's command and display: it launches the stored `ResumeCmd`, else `Cmd`, which is `exit` from the first subtest.
  - So the relaunched `pawl` pane dies again, and the live relaunched pawl the row implies never occurs. The order assertion it makes is still reached.
  - Evidence: `internal/reedengine/strand.go:624-631` (ifAbsentRelaunch: `launchCmd := strand.ResumeCmd`, else `strand.Cmd`) and the matched-branches-never-rewrite comment at `strand.go:615-620`.
  - Fix: drop the "cmd `sleep 300`" wording from the row, or add `pawl` with a long-lived command and end its pane by other means.

**Tool calls**
1. Read `internal/reedengine/strand.go:600-690` and `render.ValidateDisplay`: the manifest lacked the IfAbsent relaunch branch and the render check that the order refusal and the focus-only row pass through.
2. Read `internal/orchcli/cli_test.go` `fakeStrands` and `loomcli/bootstrap.go` `resolveStatusStrandAction`: the manifest lacked whether the fake records removals (card 3's assertion) and the keep/replace rule the card 4 smoke row reaches.

```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Result: no findings.**

I ran each function and branch with one concrete input. All of them behave correctly.

- **`anchorTemplate` build and `newAnchorHub`:**
  - Case: `Anchor: "."` with two commits on the prime warp.
  - Result: the build's steps and order match the old per-call steps.
  - The board is untouched, so `requireBoardAtUpstreamTip` passes.
  - Each test installs its seam after the copy: `killAt` at 636, 661 and 748, the failure hook at 501, 534, 593 and 717, and `SetWeftHeadSHAForTest` at 569.
  - `newIntegrateHub`'s copy at 828 comes before its hook.
  - Correct.
- **`publishPrimeRecords` and `originTemplate`:** pushing the prime records branch with its upstream gives the same start state as the old `newOriginHub`. Correct.
- **`originHubOf`:** it reads `primeBranch` and `primeRecords` from the copy, as the old constructor did, and `primeRecordsPath` is `PrimeRecords()` as before. Correct.
- **`buildOriginTask` and `originTaskTemplate`:**
  - Order is publish, then `originPair`, which calls `AddPair` and sets the upstream.
  - `AddPair` pushes the pair's branches. That is not a board commit, so the board HEAD still equals its tip.
  - Correct.
- **`newOriginTaskHub`:** the worktree comes from `PairCodeWorktree("task")` and the branch from `CurrentBranch` on the copy, the same values the old `originPair` returned. All eight replaced sites keep their hooks after the copy. Correct.
- **`movesTemplate` and `newMovesPair`:**
  - The build order matches the old one: pair, then the "Land" status, then the records-branch publish.
  - `StatusFile` resolves under the hub's own `Location`, so the file is copied with the hub.
  - The `boltreplayers` test at line 103 copies the pair before `WithoutBoltReplayersForTest`, and no build reaches a replay.
  - Correct.
- **`endedTemplate` and `arrangeEnded`:**
  - Case: `slugs = {"task"}`. The pair, status and live board entry are built, then `publishBoard` commits and pushes, so the board HEAD is the bare's tip and the build check passes.
  - The copy's relocated-binding amend still leaves HEAD as the tip.
  - Empty `slugs` would publish an empty board; no caller does that.
  - Correct.
- **`endedHub`:**
  - Case: copy, then `originBoard` on the branch from `CurrentBranch(BoardDir)` with "task" marked done.
  - The board ends ahead in the bare and the tracking ref stays at the old tip, the same state the old function left.
  - Its old second return value, the branch, is unused by every caller.
  - Correct.
- **Two-slug test, `arrangeEnded(t, hub, "task", "two")`:** the pairs are added, then the live board is published, then both are marked done on origin. The order and the board contents match the old `endedHub(t, "task", "two")`. Correct.
- **`testing.TB` signature changes (`originPair`, `writeStatus`, `publishBoard`, `publishBranch`, `boardFileOf`):** the failure paths now call `tb.Fatalf`, which a `templateBuildTB` records. A `t.Fatalf` is not needed on an error path in a build goroutine. Correct.
- **Process-global rules for builds:** no build calls `os.Setenv`, `os.MkdirTemp`, global git config, a seam setter or a once-only registration. `InstallReplay` runs at the top of the Judge, Moves and Ended tests before any copy. The anchor tests and the other `newMovesPair` or `newOriginHub` callers never reach a replay. Correct.

Tool calls:
1. `grep` of `StatusFile` in `internal/shedrun/paths.go`: to check that the status path lives under the hub's `Location`, which my context lacked.

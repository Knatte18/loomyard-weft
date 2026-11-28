---
round: 2
focus:
  - "`primeShuttle` in internal/battencli/drive.go: does a test fail if the warning `locate the task worktree to record its tackler and arbiter sessions failed` comes back for a prime with a non-empty slug and no task worktree, and is the usage recorder still wired when the pair is present?"
  - "Worktree-Create's remote lookups and fetches (`resolveWeftBranch` in internal/fabricengine/weftbranch.go, Add's pre-flight probes, adopted-warp fetch and weft fast-forward fetches in internal/fabricengine/add.go, the `TaskBranchOnOrigin` closure passed to `beamForkParent` in internal/battencli/wire.go): does each retry a transport failure as the board entry asks, return any other failure at once, and stay safe to run again, including under any lock held across the backoff?"
  - "The retry tests, among them internal/fabricengine/add_remoteleftover_integration_test.go: does each fail when its retry or its no-retry case breaks, are its recorded expectations read, and is every call that can fail the test made on the test goroutine?"
  - "_lym/landing/summary.md: do its claims about which lookups retry, and how long a retry extends the prime-lock hold (backoff plus further attempts, each bounded by git's remote deadline when it times out), match the code at HEAD?"
---

The entries cover the areas where the range may carry text no fresh reviewer has read.

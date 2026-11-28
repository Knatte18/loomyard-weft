```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every member of each set-premised finding.

**Findings left unchecked:** none. Within F2's set, I did not read the board claim's own network calls (`claimBoardEntry`).

1. **MEDIUM: the new `Add` doc in `internal/fabricengine/add.go` claims more than the code does.**
   - **Changed text:** "A read of origin that fails in transit runs again… That covers each probe, lookup and fetch of a branch the pair is made from." The `AddOptions.BackoffSleep` comment ("an origin read, a lookup or a fetch, waits it out…") is just as broad.
   - **Evidence:** two origin reads inside `Add` still run once:
     - `add.go:254`: under `AdoptOnly`, `t.PairTaskBranchOrOrigin` looks up the pair's branch name on both origins (`pairbranch.go:65-70`, then `originPairBranch`). That is a lookup of the very branch the pair is made from. `battencli/readopt.go:82` reaches it.
     - `add.go:462`: `warpDefaultBranch` calls `remoteDefaultBranch`, an `ls-remote --symref` (`addadopt.go:60-68`), under `AdoptLocal`.
   - **Summary:** `_lym/landing/summary.md` ("covers Add's later origin reads: …") also names neither as unretried. F2's fix text asked for that for any site left unwrapped.
   - Neither site is on the Worktree-Create row's own `Add` call (`Parent`, `Anchor` and `Hold` only), so narrowing the text or wrapping the two sites both resolve it.

2. **LOW: the new summary.md sentence understates one attempt's bound (F3).**
   - **Changed text:** "up to three more attempts, each bounded by git's remote deadline".
   - **Evidence:** a retried read can be several remote commands, each with its own deadline. The weft probe runs `ls-remote`, the tag `ls-remote` and a `fetch` (`remoteleftover.go:120-155`). The beam lookup through `TaskBranchOnOrigin` runs several `ls-remote` calls (`pairbranch.go:147-245`). The bound is per remote command, not per attempt.

3. **LOW: half of F1's stated gap is still open.**
   - **Changed text:** the new assertion in `TestPrimeShuttle_WiresTheReplyProbe`.
   - **Evidence:** F1's prose also says no test catches a regression that "stops wiring the usage recorder for a present pair". The new assertion covers only the absent pair, so deleting `runner.SetSessionEnd(...)` in `drive.go` would still pass it. The review's `fix` key did not ask for this, so the fixer-report should at least say so.

The other sets are covered:
- **`resolveWeftBranch` callers:** all three pass a reader. `Add` retries; checkout and reconcile read once.
- **F2's named sites:** all are wrapped.
- **F4 to F6:** each concerns a single site in the one test, and each is changed.

Tool calls:
1. Read the `CreateWorktree` closure and `reconcileTaskPair` in `wire.go`, to enumerate F2's remote reads; my context did not hold them.
2. Same call's second half read the head of `createRefusal`, to bound the closure; my context lacked it.
3. Grepped `PairTaskBranchOrOrigin`, the `AdoptOnly`/`AdoptLocal` setters and `warpDefaultBranch`, to find unwrapped reads reachable from `Add`; my context lacked the first function's body and its setters.

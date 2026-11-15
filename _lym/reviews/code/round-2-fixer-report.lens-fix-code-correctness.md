```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's changed code through the correctness lens (`withTaskWorktree` and the arbiter seam wiring, `TestArbiterSeam_Changes`, the `fakeWorktree` argument pins).

- **`withTaskWorktree`, `internal/battencli/drive.go`:** I found no defect.
  - **Task worktree present:** `taskWorktreePresent` is true, `taskWorktreeLocation` resolves the task location, `Worktree` becomes `ArbiterSeam(taskLocation)`, and `Prompt.Worktree` becomes the task path. That is the ruled behaviour.
  - **Task worktree absent:** `deps` comes back unchanged, so `Worktree` is nil, `MayEdit` is false and the prompt takes the no-fix fragment. `Prompt.Worktree` stays the prime path.
  - **Stat error:** a failure other than "not exist" propagates out of `Judge`.
  - **Per-call seam:** `shedarbiter.Seam` holds no state, so building it on each `Judge` call instead of caching it is safe. The `loaded` cache now holds the base `Deps`, and `withTaskWorktree` takes `Deps` by value, so nothing leaks between calls.
  - **Callers:** `loadedArbiterDeps` and `arbiterSeam` have no callers outside `drive.go`.
- **`TestArbiterSeam_Changes`:** it fails against each defect I tried.
  - If untracked files were dropped, `sub/new.go` goes missing.
  - If the path were joined wrong, the hashes read "deleted" rather than the content hashes.
  - If keys were absolute paths, the map no longer matches.
  - The gate log shows the test ran at tier 1.
- **`fakeWorktree` pins:** a swapped `IsAncestor` or `CommitLog` call would give `("bbb","aaa")` and trip the `("aaa","bbb")` assertion. Guarding the check on `tt.after.Head == moved.Head` skips the rows that never reach those reads.

One finding:

- **LOW**, `withTaskWorktree` in `internal/battencli/drive.go`: the F1 ruling's new branches have no test.
  - **Missing rows:** nothing pins that the task worktree gives a seam over the task path, or that an absent task worktree gives no seam and the no-fix prompt.
  - **What would still pass:** if someone reverted the wiring to the prime, every existing test would pass.
  - **Evidence:** the gate passed for `./internal/battencli` and the diff adds no test there.

Calls made:
1. `grep` of `func Seam` in `internal/shedarbiter` plus callers of `arbiterSeam` and `loadedArbiterDeps`: my context lacked `seam.go`, and I needed to confirm the seam is stateless and see who calls the changed functions.
